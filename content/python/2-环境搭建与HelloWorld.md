

```python
print("hello world")
```
hello world

与Java相比，没有包的引入。
没有显示的申明引入哪些包。

`print()` 函数是 Python 的一个内置函数，它是 Python 语言核心的一部分，因此不属于任何特定的包或模块。换句话说，`print()` 函数是直接可用的，不需要通过 `import` 语句导入。

### Python 内置函数

Python 提供了一系列内置函数，它们在任何 Python 环境中都是直接可用的，不需要导入任何模块或包。这些函数包括但不限于：

- `print()`
- `len()`
- `type()`
- `int()`
- `str()`
- `list()`
- `dict()`
- `set()`
- `tuple()`
- `input()`
- `sum()`
- `max()`
- `min()`

这些内置函数在 Python 解释器启动时自动加载，可以随时调用。

### `print()` 函数的使用示例

```python
# 输出字符串
print("Hello, world!")

# 输出多个值
print("The answer is", 42)

# 输出带有自定义分隔符
print("apple", "banana", "cherry", sep=", ")

# 输出带有自定义结束符
print("Hello", end=" ")
print("world!")

# 输出到文件
with open("output.txt", "w") as f:
    print("Hello, file!", file=f)
```

### 内置函数的来源

Python 的内置函数定义在 CPython 的 C 源代码中，具体是在 `bltinmodule.c` 文件中。在其他实现中（如 PyPy、Jython），这些内置函数的实现会有所不同，但它们在功能上保持一致。

这些内置函数也可以通过 `builtins` 模块进行访问：

```python
import builtins

# 使用 builtins 模块调用内置函数
builtins.print("Hello from builtins!")
```

### 总结

`print()` 函数是 Python 的一个内置函数，属于 Python 语言核心的一部分，不需要从任何包或模块导入。Python 提供了一系列这样的内置函数，使得许多常见操作可以直接使用而无需额外的导入。


```java
public class Hello {
  public static void main(String[] args) {
    String s="hello world!";
    System.out.println(s);
  }
}

```
`builtins.py` 是 Python 标准库的一部分，包含了 Python 的所有内置对象、函数和异常。实际上，在 Python 中，所有的内置函数和异常都在 `builtins` 模块中定义。虽然在日常编程中我们通常不需要直接导入 `builtins` 模块，但它是 Python 解释器自动导入的，确保我们可以直接使用这些内置功能。

### `builtins` 模块的作用

`builtins` 模块包含了 Python 内置的所有标准函数、异常和一些类型。它是 Python 解释器启动时自动加载的模块，因此在任何地方都可以直接使用这些内置功能，而无需显式导入。

### 常见内置函数和类型

以下是一些常见的内置函数和类型，它们都在 `builtins` 模块中定义：

- **函数**：
  - `print()`
  - `len()`
  - `type()`
  - `range()`
  - `int()`
  - `float()`
  - `str()`
  - `list()`
  - `dict()`
  - `set()`
  - `tuple()`
  - `input()`
  - `sum()`
  - `max()`
  - `min()`
  - `abs()`
  - `round()`
  - `sorted()`
  - `open()`
  - `help()`

- **异常**：
  - `Exception`
  - `ValueError`
  - `TypeError`
  - `IndexError`
  - `KeyError`
  - `FileNotFoundError`
  - `ZeroDivisionError`

- **类型**：
  - `int`
  - `float`
  - `str`
  - `list`
  - `dict`
  - `set`
  - `tuple`

### 使用 `builtins` 模块

虽然我们通常不需要显式导入 `builtins` 模块，但在某些情况下，了解其存在和使用方式是有帮助的。例如，如果我们在代码中定义了一个变量或函数名与内置函数冲突，我们仍然可以通过 `builtins` 模块访问内置函数。

#### 示例

```python
# 假设我们在代码中定义了一个名为 `print` 的变量
print = "This is a string, not a function."

# 直接调用 `print` 会引发错误
# print("Hello, world!")  # 这会引发 TypeError: 'str' object is not callable

# 我们可以使用 builtins 模块访问原始的 print 函数
import builtins
builtins.print("Hello from builtins!")
```

### 查看 `builtins` 模块中的内容

我们可以使用 `dir()` 函数列出 `builtins` 模块中的所有内容：

```python
import builtins
print(dir(builtins))
```

这将输出 `builtins` 模块中定义的所有函数、类型和异常。

### 总结

`builtins.py` 是 Python 标准库中的一个重要模块，定义了所有内置的函数、异常和一些类型。尽管我们通常不需要显式导入 `builtins` 模块，但它的存在确保了我们可以在任何地方直接使用这些内置功能。了解 `builtins` 模块可以帮助我们避免命名冲突，并在必要时访问原始的内置功能。


在java代码中，属于java.lang的包也没有显示的引入。 而且其实是类似的。

