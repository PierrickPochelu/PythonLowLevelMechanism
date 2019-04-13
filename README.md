<h1> Python low level mechanism </h1>
Learning how Python works allows you to know how your code exactly works in order to create faster and memory efficient code.
Information about Python mechanism on the web is very sparse. That’s why I have created this presentation on its general principles.




<h2> Comparison Python, Java, C++ </h2>
If you don’t know Python, here is a brief comparison between Python and other object-oriented languages.
Main features

<table>
	<tr>
		<td></td>
	<td>Python</td>
	<td>Java</td>
	<td>C++</td></tr>
	<tr><td>Time ratio of native languages compared to C++ (O2-optimization)*</td>
	<td>25 (python 2.7)<br/>
		18 (python 3.7)<br/>
		1.6 (python 2.7+Pypy)</td>
	<td>12 (java 8)<br/>
		1.15 (java 8 + non std library)</td>
	<td>1</td></tr>
	<tr><td>Run with VM [1]</td>
	<td>yes</td>
	<td>yes</td>
	<td>no</td></tr>
	<tr><td>Dynamically typed [2]</td>
	<td>yes</td>
	<td>no</td>
	<td>no</td></tr>
	<tr><td>Garbage collected</td>
	<td>yes</td>
	<td>yes</td>
	<td>no</td></tr>
	<tr><td>Memory immediately released</td>
	<td>yes [3]</td>
	<td>no</td>
	<td>yes</td></tr>
</table>

[1] The benchmark here is not complete but it allows us to feel Python standard language is slower than Java which is itself slower than C++.
[2] Features which increase running time the most.
[3] Reference cycles are not immediately released


[source]
Benchmark : https://blog.famzah.net/2016/09/10/cpp-vs-python-vs-php-vs-java-vs-others-performance-benchmark-2016-q3/
Variables
For Python, all data even primitive types like integer or float are objects.



<table>
	<tr>
		<td></td>
	<td>Python</td>
	<td>Java</td>
	<td>C++</td></td>
	<tr><td>Declare x=1</td>
	<td>Instantiates object in memory heap</td>
	<td>Creates primitive value 1</td>
	<td>Creates primitive value 1</td></tr>
	<tr><td>What x contains</td>
	<td>reference to x value</td>
	<td>binary value 1</td>
	<td>binary value 1</td></tr>
	<tr><td>declare x=1, y=1</td>
	<td>Same object in memory heap (memory optimization)</td>
	<td>two different memory locations</td>
	<td>two different memory locations</td>
	<td>Python virtual machine</td></tr>
	
Python runs on a virtual machine called CPython which interprets the developer’s code. Interpreted language allows coders to abstract the complexity of programming in machine language. The interpreted languages ​​are translated into machine language during running which is slower than compiling languages that translate into machine language only once. 


In principle, CPython of Python work exactly in the same way as Java’s JVM.
Interpretation process
We can draw the interpretation process as follows :
1. There is Python code to run (.py files)
2. CPython translates Python into bytecode (.pyc files)
3. CPython interprets bytecode


The Python virtual machine (CPython) compiles bytecode and save it as cache on disk. This compilation mechanism allows to prevent CPython from having to recompile from scratch everytime code is interpreted which is much faster. Therefore Python is not purely an interpreted language : one part is compiled into bytecode and bytecode is interpreted on the fly.
Bytecode language
This section explains what bytecode is.


Example of Python instruction :
d = (a+b)*c

Equivalence in bytecode :
 Memory      Instruction
Location   
--------------------------
  0         LOAD_FAST 0
  3         LOAD_FAST 1
  6         LOAD_FAST 2
  9         BINARY_MULTIPLY
 10         BINARY_ADD
 11         STORE_FAST 3


The operand gives the position of the variable in the line. The variable ‘a’ is in position 0, b in 1, c in 2 and d in 3.


Bytecode instructions are written with 1 or 3 bytes :
* The first byte codes the operation (ex:LOAD_FAST). 
* The next two bytes code the operand if present. Therefore Python cannot have more than 2^16 variables in a scope.


<h2> Bytecode interpreter </h2>
Python uses a stack-based virtual machine. CPython interpreter runs the algorithm below which is a classic approach in computer science.


The instruction stack initially contains bytecode instructions.
pc is init to 0,

(1) Read bytecode instruction in position pc : Read the byte operation, read two bytes operand if present.
(2) Compute next pc : if operand is present, pc += 3 otherwise pc += 1
 (3) Run the instruction
 (4) Go to step 1


[source]
py_compile official doc : https://docs.python.org/3/library/py_compile.html
stack vs. register blog : https://markfaction.wordpress.com/2012/07/15/stack-based-vs-register-based-virtual-machine-architecture-and-the-dalvik-vm/
Bytecode disassembler
It’s possible to see the bytecode of any Python code. The module dis allows to see the bytecode associated to any function. 


My function to disassemble
def my_function(a):
    b=a*3
    return b


Function to read bytes code
import dis
def func_obj(fo):
    print(fo.__name__)
    print('  co_varnames:',fo.__code__.co_varnames)
    print('  co_consts  :',fo.__code__.co_consts)
    print('Source Line  m  operation/byte-code      operand (useful name/number)')
    print('---------------------------------------------------------------------')
    dis.dis(fo)


Call it
func_obj(my_function)


Output
('  co_varnames:', ('a', 'b'))
('  co_consts  :', (None, 3))
Source Line  m  operation/byte-code      operand (useful name/number)
---------------------------------------------------------------------
  2           0 LOAD_FAST                0 (a)
              3 LOAD_CONST               1 (3)
              6 BINARY_MULTIPLY     
              7 STORE_FAST               1 (b)


  3          10 LOAD_FAST                1 (b)
             13 RETURN_VALUE        

<h2> Memory management </h2>
Like many other programming languages, Python uses two memory locations : 
* private heap memory : contains objects. The management of the heap is ensured by the Memory Manager and not directly by the OS Memory Manager.
* stack memory : contains the code of functions.


Memory management in Python is done by :
* Memory Manager which is an interface between Python world and C library functions to allocate/free memory
* Garbage Collector which detects when an object is not used anymore
Memory Manager
Memory Manager is made of different components which deal with various dynamic storage management aspects like sharing, segmenting, preallocating or caching.


The private heap memory is managed by Memory Manager (and not directly by the OS). The allocation layers are as follows :
1. Object Specific Allocator : implements distinct memory management policies adapted to the specificities of every object type
2. Raw memory interface : called for other objects. Contains a thread-safe interface to malloc() and free().
3. Memory Interface :   in version 3.6: the default allocator is now pymalloc instead of system malloc().
4. pymalloc allocator : optimized for small objects with a short lifetime called “arena”. Users must not use those functions or else it will result in mixed calls between C allocator and Python Memory Manager with fatal consequences because they implement different algorithms and operate on different heaps.
5. OS allocator : C library : malloc(), calloc(), realloc() and free(). 


[source]
memory management : https://docs.python.org/3/c-api/memory.html
Garbage Collector
The main goal of garbage is to detect unusable variables and destroy them to free memory.


Python has a garbage collector a little bit tricky made of 2 different algorithms : Reference Counting + Generational Garbage Collector. Reference counting is a very efficient garbage collector but it has a major issue : cycles cannot be automatically deleted. That’s why another garbage collector is implemented to search cycles and delete them. 
Garbage Collector 1 : Reference Counting
Garbage Collector, based on Reference Counting algorithm, counts the number of object references. When this counter is 0, it means the object is not used anymore and it is immediately deleted from memory.


Objects can be referenced either by variables or by other objects. “del” keyword allows the coder to sweep variables so the mechanism is useful to immediately delete big objects.


To see when an object is destroyed, we create a specific object destructor which prints something on screen. Object destructors are called at the end of the object’s lifetime just before the memory is released (mechanism presents in C++ too).


class MyClass():
    def __del__(self):
        print("destroy")


Example 1 - variable mechanism
a=MyClass() # object is referenced 1 time
b=a # object is referenced 2 times (a->object and b->object)
print("delete a")
del a # object is referenced 1 time (b->object)
print("delete b")
del b # object is no more referenced and destroy from memory
print("program finish")
exit()




Example 2 - function and variable scope
def F():
    a=MyClass() # object is referenced by variable. ref count : 1
    # when exist function object is not referenced. ref count : 0. So it is destroyed.
F()
print("program finish")
exit() # object is destroy




Example 3 - cycle reference
class MyList(list): # This object is a list with specific destructor
    def __del__(self):
        print("destroy")
a=MyList()
b=MyList()
a.append(b)
b.append(a)
del a
del b
print("program finish")
exit() # object is not destroy but lost in the python heap memory ! # Object are sweep when python process is finished, the destructor is not called.






To summary the pros/cons of Reference Counting algorithm :
Pros :
* Objects can be immediately destroyed when no longer needed
Cons :
* Circular references are not managed 
* Thread locking caused by the update of counters
* Memory and performance overhead caused by update of counters

[source]


   * blog : http://arctrix.com/nas/python/gc/
   * doc : http://cms.digi.com/resources/documentation/digidocs/90001537/#references/r_python_garbage_coll.htm?Highlight=garbage
Garbage Collector 2 : Garbage Collection
This an optional garbage collector which can be enabled/disabled by the coder. The only goal of this garbage collector is to find cycles by scanning objects.  Because reference cycles take time to calculate, this garbage collection algorithm has to be scheduled. Coders can  call it manually to scan cycles or schedule the frequency of those scans.
Manually call
In order to perform well, the algorithm often scans new objects and rarely old objects.


The objects still useful are sent to the next generation whereas the useless ones are swept from memory. The higher the generation is, the less frequent the scan is.


Example 4 - Use generational garbage collector manually
def make_one_cycle():
    a=list()
    a.append(a)
    return a
a=make_one_cycle()
b=make_one_cycle()
print("1) garbage.collect()="+str(gc.collect()))
del a
del b
print("2) garbage.collect()="+str(gc.collect()))# 2 cycle was deleted
print("3) garbage.collect()="+str(gc.collect()))
exit()




print("2) garbage.collect()="+str(gc.collect()))
del a
del b
print("3) garbage.collect()="+str(gc.collect()))
time.sleep(3)
print("4) garbage.collect()="+str(gc.collect())) # cycle was deleted
exit()


Schedule calls : Generational Garbage Collection
All alloc() operations should finish by a free(). The difference between the allocation count and the release count slowly grows because of cycles or objects with a long lifetime. When this delta is high enough in one generation, the algorithm scans objects in this generation. The frequency for each generation is a parameter of the algorithm.



  

Example 5 - generational garbage collector
def make_one_cycle():
    a=[]
    a.append(a)


# init generations
gc.collect()
gc.set_threshold(5,3,2)
print("start with = "+str(gc.get_count())) # we start with object in memory


# create 10 cycles
for i in range(10): # variable i is an object too
    make_one_cycle()
    print(gc.get_count()) # counter pass to 5 to 0. It means 5 objects were trashed automatically.


# see remaining objects
print("end with = " +str(gc.get_count())) # some cycles are still here
exit()


[source]
doc : http://cms.digi.com/resources/documentation/digidocs/90001537/#references/r_python_garbage_coll.htm?Highlight=garbage
Memory profiler
There are many ways to profile memory python code. Profile memory management can be useful to efficiently manage big objects, increase running time or correct a memory leak issue.
objgraph module
   * Lists objects in memory
   * Draws dependency graph
   * Shows delta between two-time steps
[source]
official doc : https://mg.pov.lt/objgraph/
memory_profiler module
   * Traces memory consumption depending on time
   * Profiles code : memory use for each instruction
[source]
official doc : https://pypi.org/project/memory_profiler/
heapy module
   * Displays an explanation of memory consumption by dumping stats


[source]
official doc : http://guppy-pe.sourceforge.net/#Heapy
blog : http://smira.ru/wp-content/uploads/2011/08/heapy.html
gc module
   * The garbage collector allows to check the number of objects in memory and can detect memory leak issues.
[source]
gc : https://docs.python.org/3/library/gc.html
