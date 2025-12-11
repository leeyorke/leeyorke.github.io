---
title: "一日一技|python中dataclass的使用"
date: 2025-11-23T20:45:42+08:00
lastmod:
draft: false
tags:
categories:
description: ""
---

## 默认值的定义
```python
@dataclass
class Script:
    description: str
    call: str
    kwargs: Dict[str, Any] = field(default_factory=dict)
    export: Optional[Export] = None
```

### `description: str`
这种定义意味着:
- 告诉用户该类在初始化时description参数类型应该为str。
- 告诉用户在初始化时这是一个不可为空的参数，必须传入一个值进来。

**注意**：
如果你传了一个整型，那么python解释器是不会报错的。如果你需要检查参数类型，必须自行实现验证方法。
如下：
```python
@dataclass
class Script:
    description: str
    call: str
    kwargs: Dict[str, Any] = field(default_factory=dict)
    export: Optional[Export] = None

    def __post_init__(self):
        if not isinstance(self.description, str):
            raise ValueError("description必须是字符串")
```

只有这样解释器才会在初始化后，去验证参数类型。`__post_init__` 方法会在`__init__` 方法执行之后自动调用。
所以可以在该方法下调用一些自定义函数。

### `kwargs: Dict[str, Any] = field(default_factory=dict)`
这种定义意味着:
- 告诉用户该类在初始化时 `kwargs` 参数类型应该为 `Dict[str, Any]`。
- 告诉用户在初始化时这是一个可为空的参数。如果不传参，它将是一个{}。但是不会为None。

**注意**：
如果要为可变类型，如list, dict等设置默认值，不应这样:
```python
kwargs: Dict[str, Any] = {}
```
而应该
```python
kwargs: Dict[str, Any] = field(default_factory=dict)
```
### `export: Optional[Export] = None`
这种定义意味着:
- 告诉用户该类在初始化时 `export` 参数类型应该为 `Export` 或 `None`。
- 告诉用户在初始化时这是一个可为空的参数。如果不传参，它将为None。
