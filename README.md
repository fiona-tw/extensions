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


Then moved onto the [other tutorial](https://realpython.com/build-python-c-extension-module/) I skipped at first, which 
went onto more of the technicalities of how the C code is written.

- Concept of Python API 🤯
- `PyArg_ParseTuple`