# Python 常见错误 Python Common Pitfalls

记录在使用Python过程中容易犯的错误和解决方案。

## 1. 可变默认参数 Mutable Default Arguments

### 错误示例
```python
def add_item(item, items=[]):
    items.append(item)
    return items

# 问题：默认参数在函数定义时只创建一次
print(add_item(1))  # [1]
print(add_item(2))  # [1, 2] - 意外！
print(add_item(3))  # [1, 2, 3] - 意外！
```

### 正确做法
```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [2] - 正确！
print(add_item(3))  # [3] - 正确！
```

### 说明
- 默认参数在函数定义时只创建一次，所有调用共享同一个对象
- 对于可变对象（list, dict, set等），使用 `None` 作为默认值

## 2. 闭包中的变量绑定 Variable Binding in Closures

### 错误示例
```python
# 期望创建不同的函数
functions = []
for i in range(5):
    functions.append(lambda: i)

# 问题：所有函数都返回4
print([f() for f in functions])  # [4, 4, 4, 4, 4]
```

### 正确做法
```python
# 方法1：使用默认参数
functions = []
for i in range(5):
    functions.append(lambda x=i: x)

print([f() for f in functions])  # [0, 1, 2, 3, 4]

# 方法2：使用functools.partial
from functools import partial

def get_value(x):
    return x

functions = [partial(get_value, i) for i in range(5)]
print([f() for f in functions])  # [0, 1, 2, 3, 4]
```

## 3. 浅拷贝 vs 深拷贝 Shallow Copy vs Deep Copy

### 问题示例
```python
import copy

# 嵌套列表
original = [[1, 2, 3], [4, 5, 6]]

# 浅拷贝
shallow = copy.copy(original)
shallow[0][0] = 999

print(original)  # [[999, 2, 3], [4, 5, 6]] - 被修改了！
print(shallow)   # [[999, 2, 3], [4, 5, 6]]
```

### 解决方案
```python
import copy

original = [[1, 2, 3], [4, 5, 6]]

# 深拷贝
deep = copy.deepcopy(original)
deep[0][0] = 999

print(original)  # [[1, 2, 3], [4, 5, 6]] - 未被修改
print(deep)      # [[999, 2, 3], [4, 5, 6]]
```

### 说明
- 浅拷贝只复制第一层对象，嵌套对象仍然是引用
- 深拷贝递归复制所有嵌套对象
- 对于简单列表，可以使用 `list.copy()` 或 `list[:]`

## 4. 字典迭代时修改 Modifying Dict During Iteration

### 错误示例
```python
d = {'a': 1, 'b': 2, 'c': 3}

# 运行时错误：RuntimeError: dictionary changed size during iteration
for key in d:
    if d[key] == 2:
        del d[key]
```

### 正确做法
```python
d = {'a': 1, 'b': 2, 'c': 3}

# 方法1：创建键的副本
for key in list(d.keys()):
    if d[key] == 2:
        del d[key]

# 方法2：字典推导式（更Pythonic）
d = {k: v for k, v in d.items() if v != 2}

print(d)  # {'a': 1, 'c': 3}
```

## 5. 整数缓存 Integer Caching

### 令人困惑的行为
```python
# 小整数（-5到256）
a = 256
b = 256
print(a is b)  # True

# 大整数
a = 257
b = 257
print(a is b)  # False（在某些情况下）
```

### 说明
- Python缓存小整数对象（-5到256）
- 比较值使用 `==`，比较身份使用 `is`
- 对于数值比较，总是使用 `==` 而不是 `is`

## 6. 列表推导式中的变量泄漏（Python 2问题）

### Python 2问题（Python 3已修复）
```python
# Python 2
x = 'original'
result = [x for x in range(5)]
print(x)  # 4 - 变量泄漏！

# Python 3
x = 'original'
result = [x for x in range(5)]
print(x)  # 'original' - 正确！
```

### 说明
- Python 3中列表推导式有自己的作用域
- Python 2中需要注意变量名冲突

## 7. 浮点数精度 Floating Point Precision

### 问题示例
```python
print(0.1 + 0.2)  # 0.30000000000000004
print(0.1 + 0.2 == 0.3)  # False
```

### 解决方案
```python
from decimal import Decimal
import math

# 方法1：使用Decimal
a = Decimal('0.1')
b = Decimal('0.2')
print(a + b)  # 0.3

# 方法2：使用math.isclose（推荐）
print(math.isclose(0.1 + 0.2, 0.3))  # True

# 方法3：round到特定精度
print(round(0.1 + 0.2, 1) == 0.3)  # True
```

## 8. 异常处理中的变量作用域

### 问题示例
```python
# Python 3中，异常变量在except块外不可访问
try:
    raise ValueError("error")
except ValueError as e:
    pass

print(e)  # NameError: name 'e' is not defined
```

### 解决方案
```python
# 在except块外定义变量
error = None
try:
    raise ValueError("error")
except ValueError as e:
    error = e

if error:
    print(f"捕获到错误: {error}")
```

## 9. 类变量 vs 实例变量 Class Variables vs Instance Variables

### 错误示例
```python
class MyClass:
    items = []  # 类变量，所有实例共享
    
    def add_item(self, item):
        self.items.append(item)

obj1 = MyClass()
obj2 = MyClass()

obj1.add_item(1)
obj2.add_item(2)

print(obj1.items)  # [1, 2] - 意外！
print(obj2.items)  # [1, 2] - 意外！
```

### 正确做法
```python
class MyClass:
    def __init__(self):
        self.items = []  # 实例变量
    
    def add_item(self, item):
        self.items.append(item)

obj1 = MyClass()
obj2 = MyClass()

obj1.add_item(1)
obj2.add_item(2)

print(obj1.items)  # [1]
print(obj2.items)  # [2]
```

## 10. 字符串拼接性能 String Concatenation Performance

### 低效做法
```python
# 在循环中使用 + 拼接字符串（O(n²)时间复杂度）
result = ""
for i in range(10000):
    result += str(i)
```

### 高效做法
```python
# 方法1：使用join（推荐）
result = "".join(str(i) for i in range(10000))

# 方法2：使用列表累积然后join
parts = []
for i in range(10000):
    parts.append(str(i))
result = "".join(parts)

# 方法3：使用f-string或format（适合少量拼接）
name = "Alice"
age = 30
result = f"{name} is {age} years old"
```

## 11. 生成器耗尽 Generator Exhaustion

### 问题示例
```python
gen = (x for x in range(5))

print(list(gen))  # [0, 1, 2, 3, 4]
print(list(gen))  # [] - 生成器已耗尽！
```

### 解决方案
```python
# 方法1：重新创建生成器
def create_gen():
    return (x for x in range(5))

gen1 = create_gen()
print(list(gen1))  # [0, 1, 2, 3, 4]

gen2 = create_gen()
print(list(gen2))  # [0, 1, 2, 3, 4]

# 方法2：使用列表（如果数据量不大）
data = list(range(5))
print(data)  # [0, 1, 2, 3, 4]
print(data)  # [0, 1, 2, 3, 4]
```

## 12. import陷阱 Import Pitfalls

### 循环导入问题
```python
# file_a.py
import file_b

def func_a():
    return file_b.func_b()

# file_b.py
import file_a  # 循环导入！

def func_b():
    return file_a.func_a()
```

### 解决方案
```python
# 方法1：重构代码，避免循环依赖
# 将共同依赖提取到第三个模块

# 方法2：延迟导入
# file_a.py
def func_a():
    import file_b  # 在函数内部导入
    return file_b.func_b()

# 方法3：使用from import
# file_b.py
def func_b():
    from file_a import func_a  # 导入特定函数
    return func_a()
```

## 最佳实践 Best Practices

1. **使用类型提示**: 帮助捕获类型相关的错误
```python
from typing import List, Optional

def process_items(items: List[int], default: Optional[int] = None) -> int:
    return sum(items) if items else (default or 0)
```

2. **使用with语句**: 确保资源正确释放
```python
# 好的做法
with open('file.txt', 'r') as f:
    content = f.read()

# 避免
f = open('file.txt', 'r')
content = f.read()
f.close()  # 可能忘记或异常时不执行
```

3. **使用enumerate而不是range(len())**
```python
# 好的做法
for i, item in enumerate(items):
    print(f"{i}: {item}")

# 避免
for i in range(len(items)):
    print(f"{i}: {items[i]}")
```

4. **使用字典的get方法**
```python
# 好的做法
value = d.get('key', default_value)

# 避免
if 'key' in d:
    value = d['key']
else:
    value = default_value
```

## 标签 Tags
`python` `pitfalls` `debugging` `best-practices` `常见错误`
