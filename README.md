# Python outside Jupyter
This repository contains a few tips for when working with Python outside the Jupyter environment. Jupyter is an excellent environment to work in when your computations are of smaller scale and execute quickly. But the moment you want to build larger more complex programs it is preferable to be able to write functions in separate files to be imported etc.

When working with Python outside Jupyter you will commonly use REPL in a terminal. REPL abbreviates "read-eval-print loop". Jupyter is essentially a REPL environment too, but flexible. It saves your history of commands and allows easy editting. Still, working outside Jupyter also has its benefits, in particular when debugging and optimizing the performance of functions.

## Debugging 
The simplest python debugging will often involve adding a number of `print` statements to your code to output and inspect the values of particular variables. This is a useful trick, but can involve a lot of code editing and is in general not particularly flexible. What if your code takes 20 minutes to run and you suddenly midway realize it was the wrong variable you decided to output?

Python has a built in debugging tool in the package `pdb` and it is fairly easy to use; simply `import pdb` and you are ready to go. You can place a trace-point at a particular point in your code by adding `pdb.set_trace()`, which will stop execution of the code when it reaches this point. From here it provides a REPL like environment where you can inspect variables in scope and crawl around the code.

Various development environments, e.g. vscode, have built-in integration for python debugging. With these debugging is often even easier and you can specify breakpoints in your code where execution halts without having to modify your files directly.

## Profiling
You have already seen iPython's (which Jupyter runs on top) built-in magic method `%timeit` which provides execution timing for the provided expression. This is an excellent tool for simple tasks, but sometimes we need something more powerful: `%timeit` might tell you that your code takes on average say 3 seconds to run. This might be good. Maybe you double the input size and now it takes 30 seconds on average. Something is scaling the execution time by a factor of 10, this seems extreme since we only doubled the input. We want to know more about what is going on inside the function! The python package `cProfile` can help us with this.

The `cProfile` package provides a flexible tool for profiling your python code. Here we will just provide your with a small piece of code used to generate a profiling report, which provides insight into the inner working of a function. For more information on profiling in python we suggest you visit https://docs.python.org/3/library/profile.html

The following function provides what in python terminology is called a *decorator* method.
```python
import cProfile, pstats, io

def profile(fnc):
  """ Decorator running cProfile on function """
  
  def inner(*args, **kwargs):
    pr = cProfile.Profile()
    pr.enable()
    retval = fnc(*args, **kwargs)
    pr.disable()
    s = io.StringIO()
    sortby = 'cumulative'
    ps = pstats.Stats(pr, stream=s).sort_stats(sortby)
    ps.print_stats()
    with open('profile.txt','w') as f:
      f.write('{s:s}'.format(s=s.getvalue()))
    return retval
    
  return inner
```

A decorator is a function which modifies another function. In this case, the *decorated* function will behave as before, but calling it will generate a profile for the function call which is saved into the text file `profile.txt`. 

Let's give a small example:
```python
def foo(a,b):
  return a + b

foo(5,3) 
# outputs 8

bar = profile(foo)
bar(5,3) 
# outputs 8 and generates 'profile.txt' with content
#
#         2 function calls in 0.000 seconds
#
#   Ordered by: cumulative time
#
#   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
#        1    0.000    0.000    0.000    0.000 <ipython-input-2-749cc0ecfad3>:1(foo)
#        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
#
```

Alternatively we could *decorate* the `foo` function. The following version of `foo` behaves exactly like `bar` from above. The `@profile` in front essentially provides a function call `foo = profile(foo)` before `foo` is used. Decorators can be very useful for writing flexible code like this.
```python
@profile
def foo(a,b):
  return a + b
```

### Reading the profile report
*to be written...*
