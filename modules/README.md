## How python interpreter load a module.

---

### What is a module

As everything in python, a module is an object. We can create our own module manually by invoking the Module constructor.

```python
# main.py
import os.path
import types
import sys

# let's "import" module1 manually

# first we need to load the code from file
module_name = 'module1'
module_file = 'module1_source.py'
module_path = '.'

module_rel_file_path = os.path.join(module_path, module_file)
module_abs_file_path = os.path.abspath(module_rel_file_path)

# next we create a module object
mod = types.ModuleType(module_name)
```

From there we can add the location of our module :
```python 
mod.__file__ = module_abs_file_path
```

When we import a module by using th `import` keyword, the python interpreter will internally read the code as a text, then will call two built in functions `compile` and `exec`.

`compile` : will convert the code to bytecode.
`exec`: will execute the bytecode and attach it to a module `__dict__` object.

```python
# read source code from file
with open(module_rel_file_path, 'r') as code_file:
    source_code = code_file.read()

# compile the module source code into a code object
# optionally we should tell the code object where the source came from
# the third parameter is used to indicate that our source consists of a sequence of statements
code = compile(source_code, filename=module_abs_file_path, mode='exec')

# execute the module
# we want the global variables to be stored in mod.__dict__
exec(code, mod.__dict__)
```

We can then use our module :

```python

# our module is now imported!
# We can use it directly via our mod variable

mod.hello()

# but we can also import it, using the module name we specified
import module1

module1.hello()
```

Learning that, we can create our own importer "from scratch"

```python 

import os.path
import types
import sys

def import_(module_name, module_file, module_path):
    if module_name in sys.modules:
        return sys.modules[module_name]

    module_rel_file_path = os.path.join(module_path, module_file)
    module_abs_file_path = os.path.abspath(module_rel_file_path)

    # read source code from file
    with open(module_rel_file_path, 'r') as code_file:
        source_code = code_file.read()

    # next we create a module object
    mod = types.ModuleType(module_name)
    mod.__file__ = module_abs_file_path

    # insert a reference to the module in sys.modules
    sys.modules[module_name] = mod

    # compile the module source code into a code object
    # optionally we should tell the code object where the source came from
    # the third parameter is used to indicate that our source consists of a sequence of statements
    code = compile(source_code, filename=module_abs_file_path, mode='exec')

    # execute the module
    # we want the global variables to be stored in mod.__dict__
    exec(code, mod.__dict__)

    # return the module
    return sys.modules[module_name]

```

### How python realy import under the hood


In order that python find and load a module, it requires a `finder` and a `loader` object.
if we have `finder` + `loader` then we have an `importer`

#### Loaders

Loaders objects could be found for each module on the special `__spec__` magic attribute. 

```python 
>>> import math
>>> math.__spec__
ModuleSpec(name='math', loader=<class '_frozen_importlib.BuiltinImporter'>, origin='built-in')
```

#### Finders

Finders are located on the `meta_path` attribute 