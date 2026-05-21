# 示例

这些真实风格的代码示例展示四项原则。每个示例都会说明 LLM 常见的错误做法，以及应该如何修正。

---

## 1. 编码前先思考

### 示例 1：隐藏假设

**用户请求：** “添加一个导出用户数据的功能”

**错误做法：LLM 擅自假设**

```python
def export_users(format='json'):
    """Export all users to a file."""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"Exported {len(users)} users"
```

**问题：**
- 假设要导出所有用户（分页和隐私怎么办？）
- 未询问就假设了文件位置。
- 假设了要包含哪些字段。
- 未检查真实数据结构就假设了 CSV 字段名。

**正确做法：暴露假设**

```
实现前我需要澄清几点：

1. 范围：导出所有用户，还是筛选后的子集？这涉及隐私影响。
2. 格式：你说的“导出”是指：
   - 在浏览器中下载文件？
   - 后台任务完成后邮件通知？
   - API endpoint 直接返回数据？
3. 字段：需要导出哪些用户字段？有些字段可能敏感。
4. 数据量：通常会有多少用户？这会影响实现方式。

最简单的方案：添加一个返回分页 JSON 的 API endpoint。
如果要做文件导出，还需要更多信息。

你倾向哪种方式？
```

### 示例 2：多种理解

**用户请求：** “让搜索更快”

**错误做法：LLM 默默选择一种理解**

```python
# 同时添加缓存、数据库索引和异步处理，
# 但没有询问用户到底想要哪种“更快”
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 lines of optimization code
```

**正确做法**

```
“让搜索更快”可能有几种含义：

1. 响应时间更短（从 500ms 降到 100ms 内）
   - 添加数据库索引
   - 缓存高频查询
   - 预估工作量：2 小时

2. 支持更多并发搜索（吞吐量）
   - 添加 async/await
   - 使用连接池
   - 预估工作量：4 小时

3. 感知速度更快（用户体验）
   - 先显示部分结果
   - 渐进式加载
   - 预估工作量：3 小时

当前典型查询耗时约 500ms。
你的场景最看重哪一项？
```

---

## 2. 简洁优先

### 示例 1：过度抽象

**用户请求：** “添加一个计算折扣的函数”

**错误做法：LLM 过度工程化**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 一个简单计算却需要 30 多行配置才能使用
```

**正确做法：保持简单**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """Calculate discount amount. percent should be 0-100."""
    return amount * (percent / 100)

# Usage
discount = calculate_discount(100.0, 10.0)  # $10 off
```

**什么时候增加复杂度：** 只有当确实需要多种折扣类型时再加。如果未来出现这个需求，到时候再重构。

### 示例 2：推测性功能

**用户请求：** “把用户偏好保存到数据库”

**错误做法：LLM 添加不必要功能**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """Save preferences with optional merging, validation, and notifications."""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # Another 30 lines for a feature nobody asked for
        pass
```

**正确做法：只做被请求的事**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """Save user preferences to database."""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**之后按需添加：** 性能真的需要时再加缓存，出现坏数据时再加校验，有明确需求时再做合并。

---

## 3. 精准改动

### 示例 1：顺手重构

**用户请求：** “修复空邮箱导致校验器崩溃的 bug”

**错误做法：LLM 改太多**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+     
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-     
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")
      
-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")
      
      return True
```

**问题：**
- 在修 bug 的同时“改进”了邮箱校验。
- 添加了用户没要求的用户名校验。
- 改了注释。
- 添加了 docstring。

**正确做法：精准修复**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")
      
      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")
      
      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")
      
      return True
```

**只改了：** 修复空邮箱处理所必需的几行。

### 示例 2：风格漂移

**用户请求：** “给上传函数添加日志”

**错误做法：LLM 重新格式化一切**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
+     
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         
-         response = requests.post(destination, files={'file': data})
-         
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+         
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+         
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**问题：**
- 把单引号改成了双引号。
- 添加了用户没要求的类型标注。
- 添加了 docstring。
- 重新整理了空白。
- 改了布尔返回逻辑。

**正确做法：匹配现有风格**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

**保持一致：** 单引号、无类型标注、原有布尔分支模式、原有间距风格。

---

## 4. 目标驱动执行

### 示例 1：模糊目标 vs 可验证目标

**用户请求：** “修复认证系统”

**错误做法：LLM 给出模糊方案**

```
我会这样修复认证系统：
1. 查看代码
2. 识别问题
3. 做一些改进
4. 测试修改

[随后在没有清晰成功标准的情况下开始改动]
```

**正确做法：定义可验证目标**

```
要“修复认证”，我需要先定义成功标准。
我们具体要解决哪个问题？

如果问题是“用户改密码后旧 session 仍然有效”：

计划：
1. 写测试：修改密码 → 验证旧 session 失效
   验证：测试失败，能复现 bug

2. 实现：改密码时使 session 失效
   验证：测试通过

3. 检查边界情况：多个活跃 session、并发修改
   验证：补充测试通过

4. 验证无回归：现有认证测试仍通过
   验证：完整测试套件通过

当前认证测试覆盖情况：[展示覆盖情况]
你遇到的具体认证问题是什么？
```

### 示例 2：多步骤任务与验证

**用户请求：** “给 API 添加 rate limiting”

**错误做法：LLM 一次性做完所有东西**

```python
# 一次提交里实现 Redis、多种策略、配置系统和监控，
# 代码 300 行，没有清晰验证步骤
```

**正确做法：增量推进并验证**

```
Rate limiting 计划：

1. 添加基础内存限流（先覆盖单个 endpoint）
   验证：
   - 测试：100 个请求 → 前 10 个成功，其余返回 429
   - 手动：curl 该 endpoint 11 次，看到限流错误

2. 抽取为 middleware（应用到所有 endpoint）
   验证：
   - 测试：限流同时作用于 /users 和 /posts
   - 现有 endpoint 测试仍通过

3. 添加 Redis backend（用于多服务实例）
   验证：
   - 测试：应用重启后限流状态仍存在
   - 测试：两个应用实例共享限流计数

4. 添加配置（不同 endpoint 不同速率）
   验证：
   - 测试：/search 允许 10/min，/users 允许 100/min
   - 配置文件解析正确

每一步都可以独立验证和部署。
是否先从第 1 步开始？
```

### 示例 3：先测试再验证

**用户请求：** “排序遇到重复分数时会出问题”

**错误做法：LLM 不复现就修**

```python
# 立刻修改排序逻辑，但没有确认 bug
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**正确做法：先复现**

```python
# 1. 先写一个能复现问题的测试
def test_sort_with_duplicate_scores():
    """Test sorting when multiple items have same score."""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # bug：重复分数的排序不稳定
    # 多运行几次，该结果应该保持一致
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：运行测试 10 次 → 出现不一致失败

# 2. 现在用稳定的排序规则修复
def sort_scores(scores):
    """Sort by score descending, then name ascending for ties."""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试稳定通过
```

---

## 反模式总结

| 原则 | 反模式 | 修正 |
|------|--------|------|
| 编码前先思考 | 默默假设文件格式、字段、范围 | 明确列出假设，并请求澄清 |
| 简洁优先 | 为一次折扣计算引入策略模式 | 先用一个函数，等真的需要复杂度再扩展 |
| 精准改动 | 修 bug 时重排引号、添加类型标注 | 只改能修复问题的行 |
| 目标驱动 | “我会查看并改进代码” | “为 bug X 写测试 → 让它通过 → 验证无回归” |

## 核心洞察

“过度复杂”的示例并不是一眼看上去就错，它们使用了设计模式和最佳实践。真正的问题是 **时机**：它们在需要之前就添加复杂度，这会：

- 让代码更难理解。
- 引入更多 bug。
- 花更长时间实现。
- 更难测试。

“简单”的版本则：
- 更容易理解。
- 更快实现。
- 更容易测试。
- 将来真的需要复杂度时仍可以重构。

**好代码是用简单方式解决今天问题的代码，而不是提前解决明天问题的代码。**
