[styleguide](https://google.github.io/styleguide/)
==================================================

Google Python Style Guide
=========================


### 2.1 Lint

`pylint` is a tool for finding bugs and style problems in Python source
code. 

Run `pylint` over your code using this
[pylintrc](https://google.github.io/styleguide/pylintrc).


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

    from sound.effects import echo
    ...
    echo.EchoFilter(input, output, delay=0.7, atten=4)


Make the following section of a markdown document shorter 
The 

### 2.4 Exceptions

Exceptions must follow certain conditions:

-   Make use of built-in exception classes when it makes sense. For
    example, raise a `ValueError` to indicate a programming mistake like
    a violated precondition (such as if you were passed a negative
    number but required a positive one). Do not use `assert` statements
    for validating argument values of a public API. `assert` is used to
    ensure internal correctness, not to enforce correct usage nor to
    indicate that some unexpected event occurred. If an exception is
    desired in the latter cases, use a raise statement. For example:

        Example:
    ```
      def connect_to_next_port(self, minimum: int) -> int:
        """Connects to the next available port.

        Args:
          minimum: A port value greater or equal to 1024.

        Returns:
          The new minimum port.

        Raises:
          ConnectionError: If no available port is found.
        """
        if minimum < 1024:
          # Note that this raising of ValueError is not mentioned in the doc
          # string's "Raises:" section because it is not appropriate to
          # guarantee this specific behavioral reaction to API misuse.
          raise ValueError(f'Min. port must be at least 1024, not {minimum}.')
        port = self._find_next_open_port(minimum)
        if port is None:
          raise ConnectionError(
              f'Could not connect to service on port {minimum} or higher.')
        assert port >= minimum, (
            f'Unexpected port {port} when minimum was {minimum}.')
        return port
    ```

-   Libraries or packages may define their own exceptions. When doing so
    they must inherit from an existing exception class. Exception names
    should end in `Error` and should not introduce repetition
    (`foo.FooError`).

-   Never use catch-all `except:` statements, or catch `Exception` or
    `StandardError`, unless you are

    -   re-raising the exception, or
    -   creating an isolation point in the program where exceptions are
        not propagated but are recorded and suppressed instead, such as
        protecting a thread from crashing by guarding its outermost
        block.

-   Use the `finally` clause to execute code whether or not an exception
    is raised in the `try` block. This is often useful for cleanup,
    i.e., closing a file.

### 2.5 Mutable Global State

Avoid mutable global state.

In those rare cases where using global state is warranted, mutable
global entities should be declared at the module level or as a class
attribute and made internal by prepending an `_` to the name. If
necessary, external access to mutable global state must be done through
public functions or class methods. 
Please explain the design reasons why mutable global state is being used
in a comment or a doc linked to from a comment.

Module-level constants are permitted and encouraged. For example:
`_MAX_HOLY_HANDGRENADE_COUNT = 3` for an internal use constant or
`SIR_LANCELOTS_FAVORITE_COLOR = "blue"` for a public API constant.
Constants must be named using all caps with underscores. 

### 2.6 Nested/Local/Inner Classes and Functions

Nested local functions or classes are fine when used to close over a
local variable. Inner classes are fine.

A class can be defined inside of a method, function, or class. A
function can be defined inside a method or function. Nested functions
have read-only access to variables defined in enclosing scopes.

Allows definition of utility classes and functions that are only used
inside of a very limited scope. Very
[ADT](https://en.wikipedia.org/wiki/Abstract_data_type)-y. Commonly used
for implementing decorators.

They are fine with some caveats. Avoid nested functions or classes
except when closing over a local value other than `self` or `cls`. Do
not nest a function just to hide it from users of a module. Instead,
prefix its name with an \_ at the module level so that it can still be
accessed by tests.

### 2.7 Comprehensions & Generator Expressions

Okay to use for simple cases.

List, Dict, and Set comprehensions as well as generator expressions
provide a concise and efficient way to create container types and
iterators without resorting to the use of traditional loops, `map()`,
`filter()`, or `lambda`.

Simple comprehensions can be clearer and simpler than other dict, list,
or set creation techniques. Generator expressions can be very efficient,
since they avoid the creation of a list entirely.

Okay to use for simple cases. Each portion must fit on one line: mapping
expression, `for` clause, filter expression. Multiple `for` clauses or
filter expressions are not permitted. Use loops instead when things get
more complicated.

    Example:
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

### 2.8 Default Iterators and Operators

Use default iterators and operators for types that support them, like
lists, dictionaries, and files.

The default iterators and operators are simple and efficient. They
express the operation directly, without extra method calls. A function
that uses default operators is generic. It can be used with any type
that supports the operation.

Use default iterators and operators for types that support them, like
lists, dictionaries, and files. The built-in types define iterator
methods, too. Prefer these methods to methods that return lists, except
that you should not mutate a container while iterating over it.

    Example:  for key in adict: ...
          if obj in alist: ...
          for line in afile: ...
          for k, v in adict.items(): ...


### 2.9 Generators

Use generators as needed.

A generator function returns an iterator that yields a value each time
it executes a yield statement. After it yields a value, the runtime
state of the generator function is suspended until the next value is
needed.

Use “Yields:” rather than “Returns:” in the docstring for
generator functions.


### 2.10 Lambda Functions

Okay for one-liners. Prefer generator expressions over `map()` or
`filter()` with a `lambda`.

Lambdas define anonymous functions in an expression, as opposed to a
statement.

Okay to use them for one-liners. If the code inside the lambda function
is longer than 60-80 chars, it’s probably better to define it as a
regular nested function.


### 2.11 Conditional Expressions

Okay to use for simple cases. Each portion must fit on one line:
true-expression, if-expression, else-expression. Use a complete if
statement when things get more complicated.

    Example:
        one_line = 'yes' if predicate(value) else 'no'
        slightly_split = ('yes' if predicate(value)
                          else 'no, nein, nyet')



### 2.17 Function and Method Decorators

Use decorators judiciously when there is a clear advantage. Avoid
`staticmethod` and limit use of `classmethod`.

[Decorators for Functions and
Methods](https://docs.python.org/3/glossary.html#term-decorator) (a.k.a
“the `@` notation”). One common decorator is `@property`, used for
converting ordinary methods into dynamically computed attributes.
However, the decorator syntax allows for user-defined decorators as
well. Specifically, for some function `my_decorator`, this:

    class C:
        @my_decorator
        def method(self):
            # method body ...

is equivalent to:

    class C:
        def method(self):
            # method body ...
        method = my_decorator(method)

Use decorators judiciously when there is a clear advantage. Decorators
should follow the same import and naming guidelines as functions.
Decorator pydoc should clearly state that the function is a decorator.
Write unit tests for decorators.

Avoid external dependencies in the decorator itself (e.g. don’t rely on
files, sockets, database connections, etc.), since they might not be
available when the decorator runs (at import time, perhaps from `pydoc`
or other tools). A decorator that is called with valid parameters should
(as much as possible) be guaranteed to succeed in all cases.

Decorators are a special case of “top-level code” - see
[main](#s3.17-main) for more discussion.

Never use `staticmethod` unless forced to in order to integrate with an
API defined in an existing library. Write a module-level function
instead.

Use `classmethod` only when writing a named constructor, or a
class-specific routine that modifies necessary global state such as a
process-wide cache.


#### 2.21.1 Definition

Type annotations (or “type hints”) are for function or method arguments
and return values:

    def func(a: int) -> list[int]:

You can also declare the type of a variable using similar
[PEP-526](https://peps.python.org/pep-0526/) syntax:

    a: SomeType = some_func()


Type annotations improve the readability and maintainability of your
code. The type checker will convert many runtime errors to build-time
errors, and reduce your ability to use [Power
Features](#power-features).

You are strongly encouraged to enable Python type analysis when updating
code. When adding or modifying public APIs, include type annotations and
enable checking via pytype in the build system. As static analysis is
relatively new to Python, we acknowledge that undesired side-effects
(such as wrongly inferred types) may prevent adoption by some projects.
In those situations, authors are encouraged to add a comment with a TODO
or link to a bug describing the issue(s) currently preventing type
annotation adoption in the BUILD file or in the code itself as
appropriate.

3 Python Style Rules
--------------------

#### 3.8.1 Docstrings

Python uses *docstrings* to document code. A docstring is a string that
is the first statement in a package, module, class or function. These
strings can be extracted automatically through the `__doc__` member of
the object and are used by `pydoc`. (Try running `pydoc` on your module
to see how it looks.) Always use the three-double-quote `"""` format for
docstrings (per [PEP 257](https://peps.python.org/pep-0257/)). A
docstring should be organized as a summary line (one physical line not
exceeding 80 characters) terminated by a period, question mark, or
exclamation point. When writing more (encouraged), this must be followed
by a blank line, followed by the rest of the docstring starting at the
same cursor position as the first quote of the first line. There are
more formatting guidelines for docstrings below.

#### 3.8.2 Modules

Every file should contain license boilerplate. Choose the appropriate
boilerplate for the license used by the project (for example, Apache
2.0, BSD, LGPL, GPL).

Files should start with a docstring describing the contents and usage of
the module.

    """A one-line summary of the module or program, terminated by a period.

    Leave one blank line.  The rest of this docstring should contain an
    overall description of the module or program.  Optionally, it may also
    contain a brief description of exported classes and functions and/or usage
    examples.

    Typical usage example:

      foo = ClassFoo()
      bar = foo.FunctionBar()
    """


##### 3.8.2.1 Test modules

Module-level docstrings for test files are not required. They should be
included only when there is additional information that can be provided.

Examples include some specifics on how the test should be run, an
explanation of an unusual setup pattern, dependency on the external
environment, and so on.

    """This blaze test uses golden files.

    You can update those files by running
    `blaze run //foo/bar:foo_test -- --update_golden_files` from the `google3`
    directory.
    """

Docstrings that do not provide any new information should not be used.

    """Tests for foo.bar."""

#### 3.8.3 Functions and Methods

In this section, “function” means a method, function, generator, or
property.

A docstring is mandatory for every function that has one or more of the
following properties:

-   being part of the public API
-   nontrivial size
-   non-obvious logic

A docstring should give enough information to write a call to the
function without reading the function’s code. The docstring should
describe the function’s calling syntax and its semantics, but generally
not its implementation details, unless those details are relevant to how
the function is to be used. For example, a function that mutates one of
its arguments as a side effect should note that in its docstring.
Otherwise, subtle but important details of a function’s implementation
that are not relevant to the caller are better expressed as comments
alongside the code than within the function’s docstring.

The docstring may be descriptive-style
(`"""Fetches rows from a Bigtable."""`) or imperative-style
(`"""Fetch rows from a Bigtable."""`), but the style should be
consistent within a file. The docstring for a `@property` data
descriptor should use the same style as the docstring for an attribute
or a [function argument](#doc-function-args)
(`"""The Bigtable path."""`, rather than
`"""Returns the Bigtable path."""`).

A method that overrides a method from a base class may have a simple
docstring sending the reader to its overridden method’s docstring, such
as `"""See base class."""`. The rationale is that there is no need to
repeat in many places documentation that is already present in the base
method’s docstring. However, if the overriding method’s behavior is
substantially different from the overridden method, or details need to
be provided (e.g., documenting additional side effects), a docstring
with at least those differences is required on the overriding method.

Certain aspects of a function should be documented in special sections,
listed below. Each section begins with a heading line, which ends with a
colon. All sections other than the heading should maintain a hanging
indent of two or four spaces (be consistent within a file). These
sections can be omitted in cases where the function’s name and signature
are informative enough that it can be aptly described using a one-line
docstring.

[*Args:*](#doc-function-args)  
List each parameter by name. A description should follow the name, and
be separated by a colon followed by either a space or newline. If the
description is too long to fit on a single 80-character line, use a
hanging indent of 2 or 4 spaces more than the parameter name (be
consistent with the rest of the docstrings in the file). The description
should include required type(s) if the code does not contain a
corresponding type annotation. If a function accepts `*foo` (variable
length argument lists) and/or `**bar` (arbitrary keyword arguments),
they should be listed as `*foo` and `**bar`.

[*Returns:* (or *Yields:* for generators)](#doc-function-returns)  
Describe the semantics of the return value, including any type
information that the type annotation does not provide. If the function
only returns None, this section is not required. It may also be omitted
if the docstring starts with Returns or Yields (e.g.
`"""Returns row from Bigtable as a tuple of strings."""`) and the
opening sentence is sufficient to describe the return value. Do not
imitate older ‘NumPy style’
([example](https://numpy.org/doc/1.24/reference/generated/numpy.linalg.qr.html)),
which frequently documented a tuple return value as if it were multiple
return values with individual names (never mentioning the tuple).
Instead, describe such a return value as: “Returns: A tuple (mat\_a,
mat\_b), where mat\_a is …, and …”. The auxiliary names in the docstring
need not necessarily correspond to any internal names used in the
function body (as those are not part of the API).

[*Raises:*](#doc-function-raises)  
List all exceptions that are relevant to the interface followed by a
description. Use a similar exception name + colon + space or newline and
hanging indent style as described in *Args:*. You should not document
exceptions that get raised if the API specified in the docstring is
violated (because this would paradoxically make behavior under violation
of the API part of the API).

    def fetch_smalltable_rows(
        table_handle: smalltable.Table,
        keys: Sequence[bytes | str],
        require_all_keys: bool = False,
    ) -> Mapping[bytes, tuple[str, ...]]:
        """Fetches rows from a Smalltable.

        Retrieves rows pertaining to the given keys from the Table instance
        represented by table_handle.  String keys will be UTF-8 encoded.

        Args:
            table_handle: An open smalltable.Table instance.
            keys: A sequence of strings representing the key of each table
              row to fetch.  String keys will be UTF-8 encoded.
            require_all_keys: If True only rows with values set for all keys will be
              returned.

        Returns:
            A dict mapping keys to the corresponding table row data
            fetched. Each row is represented as a tuple of strings. For
            example:

            {b'Serak': ('Rigel VII', 'Preparer'),
             b'Zim': ('Irk', 'Invader'),
             b'Lrrr': ('Omicron Persei 8', 'Emperor')}

            Returned keys are always bytes.  If a key from the keys argument is
            missing from the dictionary, then that row was not found in the
            table (and require_all_keys must have been False).

        Raises:
            IOError: An error occurred accessing the smalltable.
        """

#### 3.8.4 Classes

Classes should have a docstring below the class definition describing
the class. If your class has public attributes, they should be
documented here in an `Attributes` section and follow the same
formatting as a [function’s `Args`](#doc-function-args) section.

    class SampleClass:
        """Summary of class here.

        Longer class information...
        Longer class information...

        Attributes:
            likes_spam: A boolean indicating if we like SPAM or not.
            eggs: An integer count of the eggs we have laid.
        """

        def __init__(self, likes_spam: bool = False):
            """Initializes the instance based on spam preference.

            Args:
              likes_spam: Defines if instance exhibits this preference.
            """
            self.likes_spam = likes_spam
            self.eggs = 0

        def public_method(self):
            """Performs operation blah."""

All class docstrings should start with a one-line summary that describes
what the class instance represents. This implies that subclasses of
`Exception` should also describe what the exception represents, and not
the context in which it might occur. The class docstring should not
repeat unnecessary information, such as that the class is a class.

    # Example:
    class CheeseShopAddress:
      """The address of a cheese shop.

      ...
      """

    class OutOfCheeseError(Exception):
      """No more cheese is available."""

#### 3.8.5 Block and Inline Comments

The final place to have comments is in tricky parts of the code. If
you’re going to have to explain it at the next [code
review](http://en.wikipedia.org/wiki/Code_review), you should comment it
now. Complicated operations get a few lines of comments before the
operations commence. Non-obvious ones get comments at the end of the
line.

    # We use a weighted dictionary search to find out where i is in
    # the array.  We extrapolate position based on the largest num
    # in the array and the array size and then do binary search to
    # get the exact number.

    if i & (i-1) == 0:  # True if i is 0 or a power of 2.

To improve legibility, these comments should start at least 2 spaces
away from the code with the comment character `#`, followed by at least
one space before the text of the comment itself.

On the other hand, never describe the code. Assume the person reading
the code knows Python (though not what you’re trying to do) better than
you do.

    # BAD COMMENT: Now go through the b array and make sure whenever i occurs
    # the next element is i+1

#### 3.8.6 Punctuation, Spelling, and Grammar

Pay attention to punctuation, spelling, and grammar; it is easier to
read well-written comments than badly written ones.

Comments should be as readable as narrative text, with proper
capitalization and punctuation. In many cases, complete sentences are
more readable than sentence fragments. Shorter comments, such as
comments at the end of a line of code, can sometimes be less formal, but
you should be consistent with your style.

Although it can be frustrating to have a code reviewer point out that
you are using a comma when you should be using a semicolon, it is very
important that source code maintain a high level of clarity and
readability. Proper punctuation, spelling, and grammar help with that
goal.

### 3.10 Strings

Use an
[f-string](https://docs.python.org/3/reference/lexical_analysis.html#f-strings),
the `%` operator, or the `format` method for formatting strings, even
when the parameters are all strings. Use your best judgment to decide
between string formatting options. A single join with `+` is okay but do
not format with `+`.

    Example: x = f'name: {name}; score: {n}'
         x = '%s, %s!' % (imperative, expletive)
         x = '{}, {}'.format(first, second)
         x = 'name: %s; score: %d' % (name, n)
         x = 'name: %(name)s; score: %(score)d' % {'name':name, 'score':n}
         x = 'name: {}; score: {}'.format(name, n)
         x = a + b

    No: x = first + ', ' + second
        x = 'name: ' + name + '; score: ' + str(n)

Avoid using the `+` and `+=` operators to accumulate a string within a
loop. In some conditions, accumulating a string with addition can lead
to quadratic rather than linear running time. Although common
accumulations of this sort may be optimized on CPython, that is an
implementation detail. The conditions under which an optimization
applies are not easy to predict and may change. Instead, add each
substring to a list and `''.join` the list after the loop terminates, or
write each substring to an `io.StringIO` buffer. These techniques
consistently have amortized-linear run-time complexity.

    Example: items = ['<table>']
         for last_name, first_name in employee_list:
             items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
         items.append('</table>')
         employee_table = ''.join(items)

    No: employee_table = '<table>'
        for last_name, first_name in employee_list:
            employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
        employee_table += '</table>'

Be consistent with your choice of string quote character within a file.
Pick `'` or `"` and stick with it. It is okay to use the other quote
character on a string to avoid the need to backslash-escape quote
characters within the string.

    Example:
      Python('Why are you hiding your eyes?')
      Gollum("I'm scared of lint errors.")
      Narrator('"Good!" thought a happy Python reviewer.')

    No:
      Python("Why are you hiding your eyes?")
      Gollum('The lint. It burns. It burns us.')
      Gollum("Always the great lint. Watching. Watching.")

Prefer `"""` for multi-line strings rather than `'''`. Projects may
choose to use `'''` for all non-docstring multi-line strings if and only
if they also use `'` for regular strings. Docstrings must use `"""`
regardless.

Multi-line strings do not flow with the indentation of the rest of the
program. If you need to avoid embedding extra space in the string, use
either concatenated single-line strings or a multi-line string with
[`textwrap.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent)
to remove the initial space on each line:

      No:
      long_string = """This is pretty ugly.
    Don't do this.
    """

      Example:
      long_string = """This is fine if your use case can accept
          extraneous leading spaces."""

      Example:
      long_string = ("And this is fine if you cannot accept\n" +
                     "extraneous leading spaces.")

      Example:
      long_string = ("And this too is fine if you cannot accept\n"
                     "extraneous leading spaces.")

      Example:
      import textwrap

      long_string = textwrap.dedent("""\
          This is also fine, because textwrap.dedent()
          will collapse common leading spaces in each line.""")

Note that using a backslash here does not violate the prohibition
against [explicit line continuation](#line-length); in this case, the
backslash is [escaping a
newline](https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals)
in a string literal.

#### 3.10.1 Logging

For logging functions that expect a pattern-string (with %-placeholders)
as their first argument: Always call them with a string literal (not an
f-string!) as their first argument with pattern-parameters as subsequent
arguments. Some logging implementations collect the unexpanded
pattern-string as a queryable field. It also prevents spending time
rendering a message that no logger is configured to output.

      Example:
      import tensorflow as tf
      logger = tf.get_logger()
      logger.info('TensorFlow Version is: %s', tf.__version__)

      Example:
      import os
      from absl import logging

      logging.info('Current $PAGER is: %s', os.getenv('PAGER', default=''))

      homedir = os.getenv('HOME')
      if homedir is None or not os.access(homedir, os.W_OK):
        logging.error('Cannot write to home directory, $HOME=%r', homedir)

      No:
      import os
      from absl import logging

      logging.info('Current $PAGER is:')
      logging.info(os.getenv('PAGER', default=''))

      homedir = os.getenv('HOME')
      if homedir is None or not os.access(homedir, os.W_OK):
        logging.error(f'Cannot write to home directory, $HOME={homedir!r}')


#### 3.10.2 Error Messages

Error messages (such as: message strings on exceptions like
`ValueError`, or messages shown to the user) should follow three
guidelines:

1.  The message needs to precisely match the actual error condition.

2.  Interpolated pieces need to always be clearly identifiable as such.

3.  They should allow simple automated processing (e.g. grepping).

      Example:
      if not 0 <= p <= 1:
        raise ValueError(f'Not a probability: {p!r}')

      try:
        os.rmdir(workdir)
      except OSError as error:
        logging.warning('Could not remove directory (reason: %r): %r',
                        error, workdir)

      No:
      if p < 0 or p > 1:  # PROBLEM: also false for float('nan')!
        raise ValueError(f'Not a probability: {p!r}')

      try:
        os.rmdir(workdir)
      except OSError:
        # PROBLEM: Message makes an assumption that might not be true:
        # Deletion might have failed for some other reason, misleading
        # whoever has to debug this.
        logging.warning('Directory already was deleted: %s', workdir)

      try:
        os.rmdir(workdir)
      except OSError:
        # PROBLEM: The message is harder to grep for than necessary, and
        # not universally non-confusing for all possible values of `workdir`.
        # Imagine someone calling a library function with such code
        # using a name such as workdir = 'deleted'. The warning would read:
        # "The deleted directory could not be deleted."
        logging.warning('The %s directory could not be deleted.', workdir)


### 3.12 TODO Comments

Use `TODO` comments for code that is temporary, a short-term solution,
or good-enough but not perfect.

A `TODO` comment begins with the word `TODO` in all caps, a following
colon, and a link to a resource that contains the context, ideally a bug
reference. A bug reference is preferable because bugs are tracked and
have follow-up comments. Follow this piece of context with an
explanatory string introduced with a hyphen `-`. The purpose is to have
a consistent `TODO` format that can be searched to find out how to get
more details.

    # TODO: crbug.com/192795 - Investigate cpufreq optimizations.

Old style, formerly recommended, but discouraged for use in new code:

    # TODO(crbug.com/192795): Investigate cpufreq optimizations.
    # TODO(yourusername): Use a "\*" here for concatenation operator.

Avoid adding TODOs that refer to an individual or team as the context:

    # TODO: @yourusername - File an issue and use a '*' for repetition.

If your `TODO` is of the form “At a future date do something” make sure
that you either include a very specific date (“Fix by November 2009”) or
a very specific event (“Remove this code when all clients can handle XML
responses.”) that future code maintainers will comprehend. Issues are
ideal for tracking this.


### 3.14 Statements

Generally only one statement per line.

However, you may put the result of a test on the same line as the test
only if the entire statement fits on one line. In particular, you can
never do so with `try`/`except` since the `try` and `except` can’t both
fit on the same line, and you can only do so with an `if` if there is no
`else`.

    Example:

      if foo: bar(foo)

    No:

      if foo: bar(foo)
      else:   baz(foo)

      try:               bar(foo)
      except ValueError: baz(foo)

      try:
          bar(foo)
      except ValueError: baz(foo)


### 3.15 Getters and Setters

Getter and setter functions (also called accessors and mutators) should
be used when they provide a meaningful role or behavior for getting or
setting a variable’s value.

In particular, they should be used when getting or setting the variable
is complex or the cost is significant, either currently or in a
reasonable future.

If, for example, a pair of getters/setters simply read and write an
internal attribute, the internal attribute should be made public
instead. By comparison, if setting a variable means some state is
invalidated or rebuilt, it should be a setter function. The function
invocation hints that a potentially non-trivial operation is occurring.
Alternatively, [properties](#properties) may be an option when simple
logic is needed, or refactoring to no longer need getters and setters.

Getters and setters should follow the [Naming](#s3.16-naming)
guidelines, such as `get_foo()` and `set_foo()`.

If the past behavior allowed access through a property, do not bind the
new getter/setter functions to the property. Any code still attempting
to access the variable by the old method should break visibly so they
are made aware of the change in complexity.

### 3.16 Naming

`module_name`, `package_name`, `ClassName`, `method_name`,
`ExceptionName`, `function_name`, `GLOBAL_CONSTANT_NAME`,
`global_var_name`, `instance_var_name`, `function_parameter_name`,
`local_var_name`, `query_proper_noun_for_thing`,
`send_acronym_via_https`.

Function names, variable names, and filenames should be descriptive;
avoid abbreviation. In particular, do not use abbreviations that are
ambiguous or unfamiliar to readers outside your project, and do not
abbreviate by deleting letters within a word.

Always use a `.py` filename extension. Never use dashes.


#### 3.16.1 Names to Avoid

-   single character names, except for specifically allowed cases:

    -   counters or iterators (e.g. `i`, `j`, `k`, `v`, et al.)
    -   `e` as an exception identifier in `try/except` statements.
    -   `f` as a file handle in `with` statements
    -   private [type variables](#typing-type-var) with no constraints
        (e.g. `_T = TypeVar("_T")`, `_P = ParamSpec("_P")`)

    Please be mindful not to abuse single-character naming. Generally
    speaking, descriptiveness should be proportional to the name’s scope
    of visibility. For example, `i` might be a fine name for 5-line code
    block but within multiple nested scopes, it is likely too vague.

-   dashes (`-`) in any package/module name

-   `__double_leading_and_trailing_underscore__` names (reserved by
    Python)

-   offensive terms

-   names that needlessly include the type of the variable (for example:
    `id_to_name_dict`)

#### 3.16.2 Naming Conventions

-   “Internal” means internal to a module, or protected or private
    within a class.

-   Prepending a single underscore (`_`) has some support for protecting
    module variables and functions (linters will flag protected member
    access). Note that it is okay for unit tests to access protected
    constants from the modules under test.

-   Prepending a double underscore (`__` aka “dunder”) to an instance
    variable or method effectively makes the variable or method private
    to its class (using name mangling); we discourage its use as it
    impacts readability and testability, and isn’t *really* private.
    Prefer a single underscore.

-   Place related classes and top-level functions together in a module.
    Unlike Java, there is no need to limit yourself to one class per
    module.

-   Use CapWords for class names, but lower\_with\_under.py for module
    names. Although there are some old modules named CapWords.py, this
    is now discouraged because it’s confusing when the module happens to
    be named after a class. (“wait – did I write `import StringIO` or
    `from StringIO import StringIO`?”)

-   New *unit test* files follow PEP 8 compliant lower\_with\_under
    method names, for example, `test_<method_under_test>_<state>`. For
    consistency(\*) with legacy modules that follow CapWords function
    names, underscores may appear in method names starting with `test`
    to separate logical components of the name. One possible pattern is
    `test<MethodUnderTest>_<state>`.

#### 3.16.3 File Naming

Python filenames must have a `.py` extension and must not contain dashes
(`-`). This allows them to be imported and unittested. If you want an
executable to be accessible without the extension, use a symbolic link
or a simple bash wrapper containing `exec "$0.py" "$@"`.


#### 3.16.4 Guidelines derived from [Guido](https://en.wikipedia.org/wiki/Guido_van_Rossum)’s Recommendations

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


#### 3.16.5 Mathematical Notation

For mathematically heavy code, short variable names that would otherwise
violate the style guide are preferred when they match established
notation in a reference paper or algorithm. When doing so, reference the
source of all naming conventions in a comment or docstring or, if the
source is not accessible, clearly document the naming conventions.
Prefer PEP8-compliant `descriptive_names` for public APIs, which are
much more likely to be encountered out of context.

### 3.17 Main

In Python, `pydoc` as well as unit tests require modules to be
importable. If a file is meant to be used as an executable, its main
functionality should be in a `main()` function, and your code should
always check `if __name__ == '__main__'` before executing your main
program, so that it is not executed when the module is imported.

When using [absl](https://github.com/abseil/abseil-py), use `app.run`:

    from absl import app
    ...

    def main(argv: Sequence[str]):
        # process non-flag arguments
        ...

    if __name__ == '__main__':
        app.run(main)

Otherwise, use:

    def main():
        ...

    if __name__ == '__main__':
        main()

All code at the top level will be executed when the module is imported.
Be careful not to call functions, create objects, or perform other
operations that should not be executed when the file is being `pydoc`ed.

### 3.18 Function length

Prefer small and focused functions.

We recognize that long functions are sometimes appropriate, so no hard
limit is placed on function length. If a function exceeds about 40
lines, think about whether it can be broken up without harming the
structure of the program.

Even if your long function works perfectly now, someone modifying it in
a few months may add new behavior. This could result in bugs that are
hard to find. Keeping your functions short and simple makes it easier
for other people to read and modify your code.

You could find long and complicated functions when working with some
code. Do not be intimidated by modifying existing code: if working with
such a function proves to be difficult, you find that errors are hard to
debug, or you want to use a piece of it in several different contexts,
consider breaking up the function into smaller and more manageable
pieces.

### 3.19 Type Annotations

#### 3.19.1 General Rules

-   Familiarize yourself with
    [PEP-484](https://peps.python.org/pep-0484/).

-   In methods, only annotate `self`, or `cls` if it is necessary for
    proper type information. e.g.,

        @classmethod
        def create(cls: Type[_T]) -> _T:
          return cls()

-   Similarly, don’t feel compelled to annotate the return value of
    `__init__` (where `None` is the only valid option).

-   If any other variable or a returned type should not be expressed,
    use `Any`.

-   You are not required to annotate all the functions in a module.

    -   At least annotate your public APIs.
    -   Use judgment to get to a good balance between safety and clarity
        on the one hand, and flexibility on the other.
    -   Annotate code that is prone to type-related errors (previous
        bugs or complexity).
    -   Annotate code that is hard to understand.
    -   Annotate code as it becomes stable from a types perspective. In
        many cases, you can annotate all the functions in mature code
        without losing too much flexibility.



#### 3.19.5 NoneType

In the Python type system, `NoneType` is a “first class” type, and for
typing purposes, `None` is an alias for `NoneType`. If an argument can
be `None`, it has to be declared! You can use `|` union type expressions
(recommended in new Python 3.10+ code), or the older `Optional` and
`Union` syntaxes.

Use explicit `X | None` instead of implicit. Earlier versions of PEP 484
allowed `a: str = None` to be interpreted as `a: str | None = None`, but
that is no longer the preferred behavior.

    Example:
    def modern_or_union(a: str | int | None, b: str | None = None) -> str:
      ...
    def union_optional(a: Union[str, int, None], b: Optional[str] = None) -> str:
      ...

#### 3.19.9 Tuples vs Lists

Typed lists can only contain objects of a single type. Typed tuples can
either have a single repeated type or a set number of elements with
different types. The latter is commonly used as the return type from a
function.

    a: list[int] = [1, 2, 3]
    b: tuple[int, ...] = (1, 2, 3)
    c: tuple[int, str, float] = (1, "2", 3.5)



#### 3.19.12 Imports For Typing

For symbols (including types, functions, and constants) from the
`typing` or `collections.abc` modules used to support static analysis
and type checking, always import the symbol itself. This keeps common
annotations more concise and matches typing practices used around the
world. You are explicitly allowed to import multiple specific symbols on
one line from the `typing` and `collections.abc` modules. For example:

    from collections.abc import Mapping, Sequence
    from typing import Any, Generic, cast, TYPE_CHECKING

Given that this way of importing adds items to the local namespace,
names in `typing` or `collections.abc` should be treated similarly to
keywords, and not be defined in your Python code, typed or not. If there
is a collision between a type and an existing name in a module, import
it using `import x as y`.

    from typing import Any as AnyType

Prefer to use built-in types as annotations where available. Python
supports type annotations using parametric container types via
[PEP-585](https://peps.python.org/pep-0585/), introduced in Python 3.9.

    def generate_foo_scores(foo: set[str]) -> list[float]:
      ...
