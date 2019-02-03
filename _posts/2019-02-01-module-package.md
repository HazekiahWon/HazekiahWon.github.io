---
title: "Program Organization in Python3: Modules and Packages"
categories:
  - Tutotials

tags:
  - python3
author_profile: true
---
Module and Package are ways to organize complex programs in Python3.
A Module is recognized as one '.py' file, and a Package is recognized as one directory 
that includes multiple modules or sub-packages. Package is higher level organization 
in the sense that Package consists of multiple modules and Module 
consists of multiple classes or functions, which are the basic units
of programs.

## Module
As said, Module is a '.py' file wherein classes and functions are written towards
identical functionality .

### Importing modules
Global variables is defined in the context of `namespace`. This is in essence a symbol table. 
Every module has its own global symbol table, in which all the names are recorded -- classes, 
functions and variables names.

The importing behavior has something to do with this symbol table, because the user 
should be equipped with the ability to reference whatever he might have imported, 
which is realized by adding imported names to the importing module's local symbol table, 
or `namespace`.

Basically there are ways to import things:

```python
import math
import math as mathematics
from math import sqrt
```
- The first way import the namespace of 'math', and access any member of it via `math.*`.
- The second way import a namespace of 'mathematics' as an alias for 'math'.  
we can access members of 'math' via 'mathematics', as 'mathematics' is now bound for
'math' in the local symbol table.  
This kind of alias is often intended
for : 1) abbreviations, 2) avoid the conflict with the same names in current namespace, 
e.g., another variable 'math'
- the third way only imports the function (or class, variable) into the current namespace.  
So this function is visited via the current namespace, and thus no need for `*.`.  
Also note that, the outer-scope variables in the imported function is not explicitly 
imported into the current namespace, which means you cannot access it. But the interpreter 
seems to have some mechanism to reference it.

```python
# assume a global variable foo used in math.sqrt()
print(foo) # raise error because there is no such name (variable) in current namespace
print(math.foo) # raise error because there is no such name as 'math' (module) imported
```

As an emphasis, the difference between the first, second example and the third example, 
is the imported name. The former examples import the module namespace and reference needed 
names with it, while the latter example directly imports the needed names without importing 
their parent module's namespace.

```python
from math import *
```
This is another way of importing all the names in a module (except a few), but is not recommeded due to its ambiguity. 
In other words, this easily breaks when imported names shadow the same names defined 
in the local symbol table, or causes trouble in readability because such imports do 
not show traces explicitly. 

Note that names in a module that precedes with `__` is not imported, for fear of shadowing
ones defined in the local namespace.

### `__pycache__`
`__pycache__` is a cache directory for the interpreter to speed up compiling. All it 
does is to store the compiled bytecode of any module upon its first call. During the 
next and after call, the interpreter determines whether to compile a module and stores
it by comparing the modified time of the bytecode '.pyc' file with that of the '.py' file
-- if python file is modified after the bytecode file then re-compile otherwise do not.

### `__name__`
`__name__` is often (explicitly) used for testing. When a module is imported, `__name__` is set as 
the module's name. When it is run by the interpreter, `__name__` is set `__main__`.

```python
if __name__=='__main__':
    # testing code
```

## Package
A package is identified with a directory containing a `__init__.py`.

Initialization code will be executed in `__init__.py` when the package is imported, 
including the import statements. Setting `__all__` tells the interpreter to load 
some sub-modules also.

## The logic of importing names
Before I scrutinize into these, I often encounter `ImportError` due to complex project 
hierarchy. What is the logic of the interpreter to locate imported names? To explain this, 
we have two questions to answer:  
1. how does the interpreter know where to search a module?
2. how does the interpreter recognize the location of intended imports from the written names?

### [The module search path](https://docs.python.org/3/tutorial/modules.html?highlight=package#the-module-search-path)
When a name is imported, the interpreter first search in built-in modules, 
then search in `sys.path`.

Before any script runs, `sys.path` consists of :
- PYTHONPATH (a list of directory names, with the same syntax as the shell variable PATH)
- The installation-dependent default. (On UNIX, this default path is normally '/usr/local/lib/python3/'.)

However, during initialization (shortly after running the script), the interpreter 
automatically inserts the current working directory -- The directory containing the input script 
(or the current directory when no file is specified) -- ahead of `PYTHONPATH`. 
And after initialization, executable statement as `sys.path.append()` can modify `sys.path`.

As a result, the real search paths are organized in the following order:
1. built-in modules. (check it with `dir(__builtins__)`)
2. current working directory. (check it with `os.getcwd()`)
3. PYTHONPATH
4. the installation-dependent default.
5. the user specified paths.

Note that the second is temporary, dependent on the running script.

Note that the third is also refered to as 'the standard library', which I confuse 
with 'the built-in modules'. An explanation is that, 'the standard library' refers 
to the peripheral modules around the core 'built-ins', both of which add up to the 
distributed library with Python.

Note that, on file systems which support symlinks, the directory containing 
the input script is calculated after the symlink is followed. In other words 
the directory containing the symlink is not added to the module search path.

### [absolute and relative imports](https://www.python.org/dev/peps/pep-0328/)
The largest distinction between the two is leading dots. Relative imports always have 
leading dots while absolute imports never.

There are many rules that eventually cause all that mysteries.

**Rule 1** Relative imports are relative to the packages.  
A single leading dot means the current _package_ in which the importing module lies, and 
additional leading dots refer to the parents of the current package. The names right 
following the dots should be sub-packages or sub-modules. 

I have to emphasize 'package' here is not substitutable with 'directory'. A directory 
containing a `__init__.py` file is recognized as package then. This is important because 
the interpreter does not respond to a leading dot that is meant for a directory. 
The interpreter will report that the parent package not known because the directory containing 
the importing module is not a valid package.

**Rule 2** Absolute imports are relative to directories in the search paths.
Here I use directory because absolute imports do not require packages. The interpreter 
simply enumerates the search paths and checks if the given names are present.

That 'relative' word may be confusing, but I think it's the correct description. After 
all we are not typing the absolute path in importing statements -- it is still relative 
to the current working directory.

**Rule 3** The running script should only use absolute imports.
Relative imports require `__package__` and `__name__` information to determine the 
position in the hierarchy. As in the main module, `__package__` is set `None` and 
`__name__` is set `__main__`, the interpreter cannot resolve the relativity for 
the main module.

**Rule 4** Syntax
```python
# absolute imports
import package # 1
import item.subitem.subsubitem # 2: items can only be packages except the last
from package import item # 3: item cannot be a package
from package.subpackage import * # 4
# relative imports
from . import item # 5
from ..item import item # 6
```

In the `from A import B` syntax, 'B' cannot be a package.

The fourth will not import sub-modules except that `__all__` list is defined in the 
`__init__.py` of `subpackage`.

