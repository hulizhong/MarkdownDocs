[TOC]

## ReadMe
反编译pyc文件；

## pyc file
pyc文件是Python源码编译的结果就是PyCodeObject（下面将PyCodeObject的实例简称为“代码对象”）；

代码对象的结构
```python
pyc_magic  #4字节，简单校验.pyc的魔数，代表着Python的版本；
mtime     #4字节，.pyc对应的源文件的修改时间；
...
```

-----------------
从pyc中得知python的编译版本（从幻数中得知，那如果magic被更改了呢？）
```python
f = open（'test25.pyc'）
magic = f.read（4）
magic.encode（'hex'）  #'b3f20d0a'

f = open（'test26.pyc'）
magic = f.read（4）
magic.encode（'hex'）  #'d1f20d0a'

import imp
print imp.get_magic().encode('hex')  #'d1f20d0a'
```

各种Python版本的magic幻数如下：
```bash
# Python/import.c - merged by aix from Python 2.7.2 and Python 3.2.2
# EDIT: added little endian hex values for comparison first two bytes of Igor Popov's method -jimbob

Python 1.5:   20121  0x994e
Python 1.5.1: 20121  0x994e
Python 1.5.2: 20121  0x994e
Python 1.6:   50428  0x4cc4
Python 2.0:   50823  0x87c6
Python 2.0.1: 50823  0x87c6
Python 2.1:   60202  0x2aeb
Python 2.1.1: 60202  0x2aeb
Python 2.1.2: 60202  0x2aeb
Python 2.2:   60717  0x2ded
Python 2.3a0: 62011  0x3bf2
Python 2.3a0: 62021  0x45f2
Python 2.3a0: 62011  0x3bf2 (!)
Python 2.4a0: 62041  0x59f2
Python 2.4a3: 62051  0x63f2
Python 2.4b1: 62061  0x6df2
Python 2.5a0: 62071  0x77f2
Python 2.5a0: 62081  0x81f2 (ast-branch)
Python 2.5a0: 62091  0x8bf2 (with)
Python 2.5a0: 62092  0x8cf2 (changed WITH_CLEANUP opcode)
Python 2.5b3: 62101  0x95f2 (fix wrong code: for x, in ...)
Python 2.5b3: 62111  0x9ff2 (fix wrong code: x += yield)
Python 2.5c1: 62121  0xa9f2 (fix wrong lnotab with for loops and
					storing constants that should have been removed)
Python 2.5c2: 62131  0xb3f2 (fix wrong code: for x, in ... in listcomp/genexp)
Python 2.6a0: 62151  0xc7f2 (peephole optimizations and STORE_MAP opcode)
Python 2.6a1: 62161  0xd1f2 (WITH_CLEANUP optimization)
Python 2.7a0: 62171  0xdbf2 (optimize list comprehensions/change LIST_APPEND)
Python 2.7a0: 62181  0xe5f2 (optimize conditional branches:
		introduce POP_JUMP_IF_FALSE and POP_JUMP_IF_TRUE)
Python 2.7a0  62191  0xeff2 (introduce SETUP_WITH)
Python 2.7a0  62201  0xf9f2 (introduce BUILD_SET)
Python 2.7a0  62211  0x03f3 (introduce MAP_ADD and SET_ADD)

Python 3000:   3000  0xb80b
			  3010  0xc20b (removed UNARY_CONVERT)
			  3020  0xcc0b (added BUILD_SET)
			  3030  0xd60b (added keyword-only parameters)
			  3040  0xe00b (added signature annotations)
			  3050  0xea0b (print becomes a function)
			  3060  0xf40b (PEP 3115 metaclass syntax)
			  3061  0xf50b (string literals become unicode)
			  3071  0xff0b (PEP 3109 raise changes)
			  3081  0x090c (PEP 3137 make __file__ and __name__ unicode)
			  3091  0x130c (kill str8 interning)
			  3101  0x1d0c (merge from 2.6a0, see 62151)
			  3103  0x1f0c (__file__ points to source file)
Python 3.0a4:  3111  0x270c (WITH_CLEANUP optimization).
Python 3.0a5:  3131  0x3b0c (lexical exception stacking, including POP_EXCEPT)
Python 3.1a0:  3141  0x450c (optimize list, set and dict comprehensions:
	   change LIST_APPEND and SET_ADD, add MAP_ADD)
Python 3.1a0:  3151  0x4f0c (optimize conditional branches:
   introduce POP_JUMP_IF_FALSE and POP_JUMP_IF_TRUE)
Python 3.2a0:  3160  0x580c (add SETUP_WITH)
			 tag: cpython-32
Python 3.2a1:  3170  0x620c (add DUP_TOP_TWO, remove DUP_TOPX and ROT_FOUR)
			 tag: cpython-32
Python 3.2a2   3180  0x6c0c (add DELETE_DEREF)
```


## py 2 pyc

```python
# https://blog.csdn.net/a6225301/article/details/51437703
# 1.其中的 -m 相当于脚本中的import，这里的-m py_compile 相当于上面的 import py_compile 
# 2.-O 如果改成 -OO 则是删除相应的 pyo文件，具体帮助可以在控制台输入 python -h 查看

import py_compile 
py_compile.compile('path') #path是包括.py文件名的路径

python -m py_compile file.py 
python -O -m py_compile file.py  #编译成pyo文件。
```




## pyc 2 py



### uncomple2
uncompyle2 converts Python byte-code back into equivalent Python source. It accepts byte-code from CPython version 2.7 and runs on Python 2.7 only. 

0x07 有经验的py程序员会在发布程序的时候修改pyc的头8个字节，这8个字节是有特殊含义的

1. 四个字节的magic number
2. 四个字节的timestamp

头四个是magic number 很多pyc都在这个上面做文章，这修改成不合法的，然后你反编译就是败了，一板你可以找你自己编译成功的pyc头直接覆盖掉他的头8个字节就可以了， timestamp是文件的修改时间，主要是当源码有改变的时候python 就可以重新生成pyc 文件.

```bash
# https://github.com/wibiti/uncompyle2

python setup.py install --prefix=/usr/local
python setup.py install
```



### uncomple6
translates Python bytecode back into equivalent Python source code. It accepts bytecodes from Python version 1.3 to version 3.7, spanning over 22 years of Python releases. We include Dropbox’s Python 2.5 bytecode and some PyPy bytecode. 

install
```bash
pip install -e .  # set up to run from source tree
python setup.py install  # Or if you want to install instead
```

反编译指令：
```bash
/usr/local/bin/uncompyle6 -o decode *.pyc  #反编译当前目录下所有*.pyc到decode/目录中；
```

