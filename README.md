# Goals for this project
 - Build a simple C extension to be imported to python
 - Import and run this extension in a python script


For example I'd expect the (python) script to look something like the following:
```python
import my_c_extension

print(my_c_extension.simple())
```
```bash
$ python run_c_extension.py
Hello World
```
---

# Notes of implementation
Following [this](https://realpython.com/build-python-c-extension-module/) tutorial.
Soon switched to [this](https://tutorialedge.net/python/python-c-extensions-tutorial/) much simpler tutorial not 
focusing on the C code itself! The other link started with reading/writing to files in C!


Without knowing C (copying and pasting the helloworld code), building and using C extensions is pretty straightforward. 
It is just like defining your own python package, with a `setup.py` file with an extra arg in the `setup` method: 
`ext_modules = [Extension('moduleName', ['filename.c']]`.


I encountered some unexpected behaviour when playing around with the C module in my python script:
  - I called it wrong one time and ipython unexpectedly quit on me 😔. I cannot remember what exactly but I guess you 
  have to be careful with error handling for runtime errors with the C code (I managed to compile the extension 
  correctly so I'm only guessing it was a runtime C error if that is possible?)
  - I called `myModule.fib(1000)` to test performance, and I could not stop it using the normal CTRL-C/D, I had to 
  terminate it completely!


This was interesting (same result when ran twice --> probably due to exceeding largest rep of int):
```python
import myModule
print([myModule.fib(i) for i in range(50)])
 
[0, 1, 1, 2, 3, 5, ..., 1836311903,
-1323752223,  # Instead of `2971215073`
512559680,  # Instead of `4807526976`
-811192543  # Instead of `7778742049`
]
```
---

Then moved onto the [other tutorial](https://realpython.com/build-python-c-extension-module/) I skipped at first, which 
went onto more of the technicalities of how the C code is written.
 
- This is all build using a Python API 🤯 --> [api ref manual](https://docs.python.org/3/c-api/index.html)

## Structures
The following is notes from [structure docs](https://docs.python.org/3/c-api/structures.html)

All objects in python share a small number of fields at the begining of it's representation in memory, represented by one of the two types:
 - `PyObject`
 - `PyVarObject`

All object types are extensions of the `PyObject` type, which tells the Python Interpreter to treat a pointer to an object as an object.
`PyVarObject` is an extension of `PyObject` that adds the `ob_size` field which is only used for objects that have a notion of `length`.

Macros have the syntax: `Py_MACRO_NAME`

`METH_VARARGS` and `METH_KEYWORDS` are flags (a.k.a calling conventions of which there are 6) to add to methods which expect args/kwargs, or `METH_NOARGS` to be used for methods which don't need args.

`PyArg_ParseTuple` parses arguments recieved from the python programe into local (C) variables. See full docs [here](https://docs.python.org/3/c-api/arg.html#c.PyArg_ParseTuple).
This can only be used for methods which takes positional arguments, for kwargs use `PyArg_ParseTupleAndKeywords`
Example usage:
```c
PyArg_ParseTuple(args, "ss", &str, &filename)
```

- `args` are a type of `PyObject`
- `"ss"` is the format specifier which specifies the data type of the arguments to parse
- `&str` and `&filename` are pointers to local variables which the parsed values will be assigned to.

If anything fails, the built in function will return `false`, `NULL` will be returned and evaulation will stop 
`PyArg_ParseTuple` will return `false` if it fails to parse the args provided

##  Defining a method
`PyMethodDef` - structure used to describe a method of an extension type
Members / Fields which represent a single method in the module:
- `ml_name`: name of function to be called in Python
- `ml_meth`: name of C function to invoke
- `ml_flags`: flags which indicate how the function should be constructed - calling convention
- `ml_doc`: docstring contents

### Calling conventions

- `METH_VARARGS`: `self` and `args`
- `METH_VARARGS | METH_KEYWORDS`: `self` and `args`
- `METH_VARARGS`: `self`, `args` and `kwargs`
- `METH_FASTCALL`: `self` and C array of `PyObject` ? (aparently not part of limited API whatever that means)
- `METH_FASTCALL | METH_KEYWORDS`: extension of above supporting kwargs
- `METH_NOARGS`: `self` only
- `METH_O`: single object argument == using `PyArg_ParseTuple` with `"o"` arg
