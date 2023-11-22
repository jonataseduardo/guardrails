Python Style Guide
=========================
## 1 The Zen of Python

Before coding, remember the Zen of Python [PEP20](https://peps.python.org/pep-0020/)

```
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

This style guide heavily relies on the [Google Style Guide](https://google.github.io/styleguide/pyguide.html). Therefore, topics not covered in this document can refer to the Google Style Guide for guidance.

## 2 General Rules

### 2.1 Lint and format

Use [ruff](https://github.com/astral-sh/ruff) to lint and format you code. 

### 2.2 Imports

Use `import` statements for packages and modules only, not for
individual classes or functions.

-   Use `import x` for importing packages and modules.
-   Use `from x import y` where `x` is the package prefix and `y` is the
    module name with no prefix.
-   Use `from x import y as z` in any of the following circumstances:
    -   Two modules named `y` are to be imported.
    -   `y` conflicts with a top-level name defined in the current
        module.
    -   `y` conflicts with a common parameter name that is part of the
        public API (e.g., `features`).
    -   `y` is an inconveniently long name.
    -   `y` is too generic in the context of your code (e.g.,
        `from storage.file_system import options as fs_options`).
-   Use `import y as z` only when `z` is a standard abbreviation (e.g.,
    `import numpy as np`).

For example the module `sound.effects.echo` may be imported as follows:

```python
    from sound.effects import echo
    ...
    echo.EchoFilter(input, output, delay=0.7, atten=4)
```


Make the following section of a markdown document shorter 
The 

### 2.3 Exceptions

Exceptions must follow certain conditions:

To create an example of a simple function that reads data and uses try, except, and assert, consider the following rules:

- Avoid using catch-all `except:` statements or catching `Exception` unless you are re-raising the exception or creating an isolation point where exceptions are recorded and suppressed.
- Use built-in exception classes when appropriate. For example, raise a `ValueError` for a programming error such as a violated precondition. 
- Use `assert` for internal correctness and use `raise` to indicate unexpected events or enforce correct usage.
- Libraries/packages can define their own exceptions, but they should inherit from an existing exception class. Exception names should end in `Error` and avoid repetition (e.g., `foo.FooError`).
- Use `finally` to execute code regardless of whether an exception is raised in the `try` block. This is often used for cleanup tasks, such as closing a file.

Example:

```python
def read_data_from_file(filename):
    assert isinstance(filename, str), "filename must be a string"
    data = None
    try:
        with open(filename, 'r') as file:
            data = file.read()
    except FileNotFoundError:
        raise FileReadError(f"File {filename} not found")
    except IOError:
        raise FileReadError(f"Error occurred while reading the file {filename}")
    finally:
        if data is None:
            print("No data read from the file")
        else:
            print(f"Read {len(data)} characters from the file")
    return data
```

### 2.4 Comprehensions & Generator Expressions

It is recommended to use comprehension for simple cases. Use loops instead when things become more complicated.

```python
      result = [mapping_expr for value in iterable if filter_expr]

      result = [{'key': value} for value in iterable
                if a_long_filter_expression(value)]

      result = [complicated_transform(x)
                for x in iterable if predicate(x)]

      descriptive_name = [
          transform({'key': key, 'value': value}, color='black')
          for key, value in generate_iterable(some_input)
          if complicated_condition_is_met(key, value)
      ]

      result = []
      for x in range(10):
          for y in range(5):
              if x * y > 10:
                  result.append((x, y))

      return {x: complicated_transform(x)
              for x in long_generator_function(parameter)
              if x is not None}

      squares_generator = (x**2 for x in range(10))

      unique_names = {user.name for user in users if user is not None}

      eat(jelly_bean for jelly_bean in jelly_beans
          if jelly_bean.color == 'black')
```
### 2.5 Default Iterators and Operators

Please utilize default iterators and operators for types that have support for them, such as lists, dictionaries, and files.

The default iterators and operators are simple and effective. They directly represent the operation without requiring any extra method calls. A function that makes use of default operators is versatile and can be used with any type that supports the operation.

Example: 
```python
for key in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in adict.items(): ...
```

### 2.6 Generators

Please use generators as necessary.

A generator function produces an iterator that generates a value each time it executes a yield statement. Once it yields a value, the generator function's runtime state is paused until the next value is required.

When documenting generator functions, please use "Yields:" instead of "Returns:" in the docstring.


### 2.7 Lambda Functions

Lambdas are used to define anonymous functions within an expression, rather than a statement. They are suitable for one-liners. However, if the code inside the lambda function exceeds 60-80 characters, it is generally preferable to define it as a regular nested function.


### 2.8 Conditional Expressions


One-line conditional expressions are acceptable to use for simple cases. Each portion must fit on one line: true-expression, if-expression, else-expression. Use a complete if statement when things become more complicated.

```python
one_line = 'yes' if predicate(value) else 'no'
```

### 2.9 Function and Method Decorators

Use decorators wisely when there is a clear advantage. Decorators should follow the same import and naming conventions as functions. The pydoc for decorators should explicitly state that the function is a decorator. It is important to write unit tests for decorators. Try to avoid using `staticmethod` and limit the usage of `classmethod`.

```python
    class C:
        @my_decorator
        def method(self):
            # method body ...
```
is equivalent to:

```python
    class C:
        def method(self):
            # method body ...
        method = my_decorator(method)
```



3 Python Style Rules
--------------------

### 3.1 Naming

Function names, variable names, and file names should be descriptive; avoid abbreviations. In particular, do not use abbreviations that are ambiguous or unfamiliar to readers outside your project.

Always use a `.py` filename extension. Never use dashes.

Use descriptive names such as `module_name`, `package_name`, `ClassName`, `method_name`, `ExceptionName`, `function_name`, `GLOBAL_CONSTANT_NAME`, `global_var_name`, `instance_var_name`, `function_parameter_name`, `local_var_name`, `query_proper_noun_for_thing`, and `send_acronym_via_https`.


#### 3.1.1 Guidelines derived from [Guido](https://en.wikipedia.org/wiki/Guido_van_Rossum)’s Recommendations

<table>
<thead>
<tr class="header">
<th>Type</th>
<th>Public</th>
<th>Internal</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Packages</td>
<td><code>lower_with_under</code></td>
<td></td>
</tr>
<tr class="even">
<td>Modules</td>
<td><code>lower_with_under</code></td>
<td><code>_lower_with_under</code></td>
</tr>
<tr class="odd">
<td>Classes</td>
<td><code>CapWords</code></td>
<td><code>_CapWords</code></td>
</tr>
<tr class="even">
<td>Exceptions</td>
<td><code>CapWords</code></td>
<td></td>
</tr>
<tr class="odd">
<td>Functions</td>
<td><code>lower_with_under()</code></td>
<td><code>_lower_with_under()</code></td>
</tr>
<tr class="even">
<td>Global/Class Constants</td>
<td><code>CAPS_WITH_UNDER</code></td>
<td><code>_CAPS_WITH_UNDER</code></td>
</tr>
<tr class="odd">
<td>Global/Class Variables</td>
<td><code>lower_with_under</code></td>
<td><code>_lower_with_under</code></td>
</tr>
<tr class="even">
<td>Instance Variables</td>
<td><code>lower_with_under</code></td>
<td><code>_lower_with_under</code> (protected)</td>
</tr>
<tr class="odd">
<td>Method Names</td>
<td><code>lower_with_under()</code></td>
<td><code>_lower_with_under()</code> (protected)</td>
</tr>
<tr class="even">
<td>Function/Method Parameters</td>
<td><code>lower_with_under</code></td>
<td></td>
</tr>
<tr class="odd">
<td>Local Variables</td>
<td><code>lower_with_under</code></td>
<td></td>
</tr>
</tbody>
</table>

### 3.2 Type Annotations

Type annotations enhance the readability and maintainability of your code. They enable the use of a type checker that can convert numerous runtime errors into compile-time errors.

Type annotations, also known as "type hints," are used for specifying the types of function or method arguments and return values:

```python
def func(a: int) -> list[int]:
```

You can also declare the type of a variable using a similar syntax as described in [PEP-526](https://peps.python.org/pep-0526/):

```python
a: SomeType = some_func()
```

#### 3.2.1 General Rules


- It is encouraged, but not required, to annotate all the functions in a module.

    - At the very least, annotate your public APIs.
    - Use your judgment to strike a good balance between safety and clarity on one hand, and flexibility on the other.
    - Annotate code that is prone to type-related errors (such as previous bugs or complexity).
    - Annotate code that is difficult to understand.
    - Annotate code as it becomes stable from a types perspective. In many cases, you can annotate all the functions in mature code without sacrificing too much flexibility.

- Make sure to familiarize yourself with [PEP-484](https://peps.python.org/pep-0484/).

#### 3.2.2 NoneType

It is recommended to use the explicit form "X | None" instead of the implicit form.

Here are the corrected versions of the functions:

```python
def modern_or_union(a: str | int | None, b: str | None = None) -> str:
  ...

def union_optional(a: Union[str, int, None], b: Optional[str] = None) -> str:
  ...
```

#### 3.2.3 Tuples vs Lists

Typed lists can only contain objects of the same type. Typed tuples can have either a single repeated type or a set number of elements with different types. The latter is commonly used as the return type from a function.

```python
a: list[int] = [1, 2, 3]
b: tuple[int, ...] = (1, 2, 3)
c: tuple[int, str, float] = (1, "2", 3.5)
```
#### 3.2 Docstrings

Python uses docstrings to document code. A docstring is a string that is the first statement in a package, module, class, or function. These strings can be extracted automatically through the __doc__ member of the object and are used by pydoc. (Try running pydoc on your module to see how it looks.) Always use the three-double-quote """ format for docstrings (per PEP 257).

A docstring is required for any function that meets one or more of the following criteria:

- It is part of the public API.
- It has a nontrivial size.
- It has non-obvious logic.

*Args*
Each parameter should be listed by name, followed by a description separated by a colon and either a space or a newline. 
If the description is too long to fit on one line (80 characters), use a hanging indent of 2 or 4 spaces after the parameter name.
The description should include the required type(s) if the code does not have a corresponding type annotation. 
If a function accepts `*foo` (variable length argument lists) and/or `**bar` (arbitrary keyword arguments), they should be listed as `*foo` and `**bar`.

*Returns* (or [Yields] for generators)
Describe the meaning of the return value, including any type information that is not provided by the type annotation. If the function only returns None, this section is not necessary.

*Raises*
List all relevant exceptions for the interface, followed by a description. Use a similar format as described in *Args*.

*Example* (Optional) 
Write a prompt exemple of the function usage

*References*  (Optional)
Write a list of links of usefull references


```python
def calculate_sum(a: int, b: int, *args, **kwargs) -> int:
    """
    Calculates the sum of two numbers, with optional additional numbers.
    Args:
        a (int): The first number to add.
        b (int): The second number to add.
        *args (int): Optional additional numbers to add.
        **kwargs: Optional keyword arguments. Currently unused.
    Returns:
        int: The sum of a, b and any additional numbers.
    Raises:
        TypeError: If any of the arguments are not integers.
    Example:
        >>> calculate_sum(1, 2)
        3
        >>> calculate_sum(1, 2, 3, 4)
        10
    References:
        https://docs.python.org/3/tutorial/controlflow.html#defining-functions
    """
    if not all(isinstance(x, int) for x in (a, b) + args):
        raise TypeError('All arguments must be integers')
    return a + b + sum(args)
```

### 3.3 Error Messages

Write a example python of error message that following the rules:

Error messages, such as message strings on exceptions like `ValueError`, or messages shown to the user, should adhere to three guidelines:

1. The message should accurately reflect the specific error condition.
2. Interpolated pieces should always be easily identifiable.
3. They should be designed to allow for simple automated processing, such as grepping.

```python
def divide_numbers(numerator: int | float, denominator: int | float) -> float:
    if not isinstance(numerator, (int, float)) or not isinstance(denominator, (int, float)):
        raise ValueError(f"Both numerator ({numerator}) and denominator ({denominator}) must be numbers.")
    if denominator == 0:
        raise ValueError("Denominator cannot be zero.")
    return numerator / denominator

try:
    divide_numbers(10, 'a')
except ValueError as e:
    print(f"Error: {e}")
```

### 3.4 Strings

Please utilize an [f-string](https://docs.python.org/3/reference/lexical_analysis.html#f-strings) and refrain from using the `+` and `+=` operators to accumulate a string within a loop.

Multi-line strings do not align with the indentation of the rest of the program. If you want to avoid adding extra space in the string, you can either use concatenated single-line strings or a multi-line string with [`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) to remove the initial space on each line:

```python
long_string = textwrap.dedent("""\
    This is also acceptable, as textwrap.dedent()
    will remove common leading spaces in each line.""")
```

#### 3.4.1 Mathematical Notation

You can use LaTeX for mathematical notation in docstrings and other suitable places, such as figure labels and captions.

### 3.5 Comments

Use comments in your code sparingly. Comments should be used to explain why the code was written, not what the code is doing. If the code is too difficult to understand, try breaking it down into simpler parts.

#### 3.5.1 TODO Comments

Please use `TODO` comments for code that is temporary, a short-term solution, or good enough but not perfect.

If your `TODO` is in the form of "At a future date do something," please ensure that you either provide a specific date ("Fix by November 2009") or a specific event ("Remove this code when all clients can handle XML responses") that future code maintainers will understand. Issues are perfect for tracking this.


### 3.5 Main

Please include the statement `if __name__ == "__main__"` in your code. This practice is advantageous for organizing code, promoting reusability, avoiding unintended consequences, and facilitating testing. By using this statement, specific sections of the code will only run when the file is executed directly, rather than when it is imported as a module in another script. This prevents undesired side effects when the module is imported and enables the file to be independently executed for testing purposes. Additionally, it enhances code reusability as functions and classes defined in the script can be utilized in other scripts without executing the entire script.

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

### 3.6 Function length

Prefer small and focused functions. Long functions may be necessary in certain cases, so there is no strict limit on function length. As a general guideline, if a function exceeds approximately 40 lines, consider whether it can be divided without compromising the program's structure.
