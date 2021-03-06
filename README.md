# CodeSnap

[![build](https://github.com/gaogaotiantian/codesnap/workflows/build/badge.svg)](https://github.com/gaogaotiantian/codesnap/actions?query=workflow%3Abuild)  [![pypi](https://img.shields.io/pypi/v/codesnap.svg)](https://pypi.org/project/codesnap/)

CodeSnap is a light-weighted deterministic profiling tool that can visualize python code running result in flame graph. The major data CodeSnap displays is FEE(function entry/exit), or equivalently, the call stack. 

Unlike traditional flame graph, which is normally generated by sampling profiler, CodeSnap can display every function executed the the corresponding entry/exit time from the beginning of the program to the end, which is helpful for programmers to catch sporatic performance issues. 

With CodeSnap, the programmer can intuitively understand what their code is doing and how long each function takes.  

You can take a look at the [demo](http://www.minkoder.com/codesnap/result.html) result of an example program running recursive merge and quick sort algorithm.

A modified version of [d3-flame-graph](https://github.com/spiermar/d3-flame-graph) is used to display the data.

## Requirements

CodeSnap requires python 3.5+. No other package is needed. For now, CodeSnap binary on pip only supports CPython + Linux. However, in theory the source code can build on Windows/MacOS.

## Install

The prefered way to install CodeSnap is via pip

```
pip install codesnap
```

You can also download the source code and build it yourself.

## Usage

There are a couple ways to use CodeSnap

### Command Line

The easiest way to use CodeSnap it through command line. Assume you have a python script to profile and the normal way to run it is:

```
python3 my_script.py
```

You can simply use CodeSnap as 

```
python3 -m codesnap my_script.py
```

which will generate a ```result.html``` file in the directory you run this command. Open it in browser and there's your result.

If your script needs arguments like 

```
python3 my_script.py arg1 arg2
```

Just feed it as it is to CodeSnap

```
python3 -m codesnap my_script.py arg1 arg2
```

### Inline

Sometimes the command line may not work as you expected, or you do not want to profile the whole script. You can manually start/stop the profiling in your script as well.

First of all, you need to import ```CodeSnap``` class from the package, and make an object of it.

```python
from codesnap import CodeSnap

snap = CodeSnap()
```

If your code is executable by ```exec``` function, you can simply call ```snap.run()```

```python
snap.run("import random;random.randrange(10)")
```

This will as well generate a ```result.html``` file in your current directory. You can pass other file path to the function if you do not like the name ```result.html```

```python
snap.run("import random; random.randrange(10)", output_file = "better_name.html")
```

When you need a more delicate profiler, you can manually enable/disable the profile using ```start()``` and ```stop()``` function.

```python
snap.start()
# Something happens here
snap.stop()
snap.save() # also takes output_file as an optional argument
```

With this method, you can only record the part that you are interested in

```python
# Some code that I don't care
snap.start()
# Some code I do care
snap.stop()
# Some code that I want to skip
snap.start()
# Important code again
snap.stop()
snap.save()
```

**It is higly recommended that ```start()``` and ```stop()``` function should be in the same frame(same level on call stack). Problem might happen if the condition is not met**

### Choose tracer

The default tracer for current version is a pure-python tracer, which works fine with any OS/Interpreter. However, the overhead it introduces is pretty large. In the worst case(a lot of FEEs), it might be 20-30x. 

To deal with this problem, the c tracer is under development and is currently available for CPython on Linux. It might work on Windows, but I've never tested it. 

To use c tracer, set the optional argument ```tracer``` to ```"c"``` when you initialize ```CodeSnap``` object.

```python
snap = CodeSnap(tracer="c")
```

#### Cleanup of c tracer

The interface for c trace is almost exactly the same as python tracer, except for the fact that c tracer does not support command line run now. However, to achieve lower overhead, some optimization is applied to c tracer so it will withhold the memory it allocates for future use to reduce the time it calls ```malloc()```. If you want the c trace to free all the memory it allocates while collecting trace, use

```python
snap.cleanup()
```

## Performance

Overhead is a big consideration when people choose profilers. CodeSnap now has a similar overhead as native cProfiler. It works slightly worse in the worst case(Pure FEE) and better in easier case because even though it collects some extra information than cProfiler, the structure is lighter. 

Admittedly, CodeSnap is only focusing on FEE now, so cProfiler also gets other information that CodeSnap does not acquire.

An example run for test_performance with Python 3.8 / Ubuntu 18.04.4 on Github VM

```
fib       (10336, 10336): 0.000852800 vs 0.013735200(16.11)[py] vs 0.001585900(1.86)[c] vs 0.001628400(1.91)[cProfile]
hanoi     (8192, 8192): 0.000621400 vs 0.012924899(20.80)[py] vs 0.001801800(2.90)[c] vs 0.001292900(2.08)[cProfile]
qsort     (10586, 10676): 0.003457500 vs 0.042572898(12.31)[py] vs 0.005594100(1.62)[c] vs 0.007573200(2.19)[cProfile]
slow_fib  (1508, 1508): 0.033606299 vs 0.038840998(1.16)[py] vs 0.033270399(0.99)[c] vs 0.032577599(0.97)[cProfile]
```

## Limitations

CodeSnap uses ```sys.setprofile()``` for its profiler capabilities, so it will conflict with other profiling tools which also use this function. Be aware of it when using CodeSnap.

## Bugs/Requirements

Please send bug reports and feature requirements through [github issue tracker](https://github.com/gaogaotiantian/codesnap/issues). CodeSnap is currently under development now and it's open to any constructive suggestions.

## License

Copyright Tian Gao, 2020.

Distributed under the terms of the  [Apache 2.0 license](https://github.com/gaogaotiantian/codesnap/blob/master/LICENSE).