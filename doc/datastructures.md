
# Python's Basic Data Structures

---

## Ints / Floats 

No surprises here (almost):

    !python
    >>> (1+7)*30
    240
    >>> 1/3
    0
    >>> from __future__ import division
    >>> 1/3
    0.3333333333333333
    >>> 1//3
    0
    >>> 1.0/3
    0.3333333333333333

---

## Strings

Again - no surprises

    !python
    >>> "Hello" + ' ' + "World"
    'Hello World'
    >>> " and ".join(["me", "my friend"])
    'me and my friend'

Strings are *immutable*
    
    !python
    >>> a = "hello"
    >>> b = a
    >>> a += "world" # a new object is created!
    >>> a
    'helloworld'
    >>> b
    'hello'

---

## Dict

Stores mappings

    !python
    >>> phone_numbers = {"sagiv": "0526087182", "alon": "12345678"}
    >>> phone_numbers['sagiv']
    '0526087182'
    >>> phone_numbers['sagiv'] = "000000" # dicts are *mutable*
    >>> phone_numbers
    {'alon': '12345678', 'sagiv': '000000'}
    >>> phone_numbers = dict(sagiv="123", alon="456") 
    >>> {x: x**2 for x in (1,2,3)} # "dict comprehension"

Only *immutable* types can act as dictionary keys (why?)

---

## Tuple

Stores 'records' or 'structs'

    !python
    >>> sagiv = ("sagiv", "malihi", 30, 1.62)
    >>> first, last, age, height = sagiv
    >>> age
    30

When you want to pass around records but a class is an overkill.
Later we'll see some in-between options (namedtuple, __slots__)

---

## Lists

For holding variable-sized collections 

    !python
    >>> l = [1,"a", {2:3}] # (non-homogenous)
    >>> l.append(4) # lists are *mutable*
    >>> l
    [1, 'a', {2: 3}, 4]
    >>> range(10)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >>> range(10)[1:-2]
    [1, 2, 3, 4, 5, 6, 7]
    >>> range(10)[:-2:3]
    [0, 3, 6]
    >>> range(10)[::-1]
    [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
    >>> [x**2 for x in range(10)] # "list comprehension"
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
    >>> [x**2 for x in range(10) if x%2==0]
    [0, 4, 16, 36, 64]

**Not** to be confused with tuples!

---

## Sets
    
When order & count don't matter

    !python
    >>> set(range(10) + range(5,15))
    set([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14])
    >>> set([1,2]) | (set([2,3])) # supports set operations
    set([1, 2, 3])
    >>> set([1,2]) - (set([2,3]))
    set([1])
    >>> set([1,2]) & (set([2,3]))
    set([2])


---

## Functions

Are first-class citizens.

    !python
    >>> def f(x,y):
    ...     return x+y
    ... 
    >>> f
    <function f at 0x10fd2e140>
    >>> g = lambda x, y: x * y
    >>> g
    <function <lambda> at 0x10fd2e1b8>
    >>> f(g(1,2), g(3,4))
    14
    >>> f(y='world', x='hello ')
    'hello world'

---

## args (*)

Allows printf style variadic functions.

**variadic function - a function that gets a variable number of arguments.**

Usage example:

	!python
	logger.info("%s, I am your %s", jedi, relationship)

Implementation example:

	!python
	def f(*args): # you can call it in a different name if you want, the * is a must
		print type(args), args
	    print 

	>>> f(1, 2, 3)
	tuple (1, 2, 3)
	
---

The reverse operation - pass a sequence as the arguments:

	!python
	>>> f(*(1,2,3))
	tuple (1, 2, 3)

    >>> f(*[1,2,3]) # we can put any iterable here
	tuple (1, 2, 3)

    >>> f(*'abc') # and I mean, any iterable
	tuple ('a', 'b', 'c')

When unpacking in a non variadic function call, make sure the number of args match:

	!python
	def add(x, y):
		return x + y
		
	>>> add(*[1, 2, 3])
	Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: add() takes exactly 2 arguments (3 given)
	
---
	
## kwargs (\*\*)

Examples:

	!python
	def foo(**kwargs):
		print kwargs
		
	>>> foo(a=1, b=2)
	{'a': 1, 'b': 2}
	
	>>> foo(**{'a': 1, 'b': 2})
    {'a': 1, 'b': 2}
	
	def foo(a, b=None):
		print a, b
		
    >> foo(**{'a': 1, 'b': 2})
	1 2

---

## Classes

Allows us to create new types. 

Beware of old-style classes (bug from Python's *dark* past):

    !python
    class A:
        def f(self): pass

Python supports multiple inhertiance!

	!python
	class Person(object): pass
	class Student(Person): pass
	class Teacher(Person): pass
	class TeachingStudent(Student, Teacher): pass
	>>> TeachingStudent.mro()
    [<class '__main__.TeachingStudent'>, 
     <class '__main__.Student'>, 
     <class '__main__.Teacher'>, 
     <class '__main__.Person'>, 
     <type 'object'>]		

---

## super
'super' is used to call the method on the class which is next on the MRO 

    !python
	class Student(object):
		def __init__(self):
			super(Student, self).__init__()
			print "Student"
	class Teacher(object):
		def __init__(self):
			super(Teacher, self).__init__()
			print "Teacher"
	class TeachingStudent(Student, Teacher):
		def __init__(self):
			super(TeachingStudent, self).__init__()
			print "TeachingStudent"

	>>> t = TeachingStudent()
	Teacher
	Student
	TeachingStudent

Notice how Teacher's __init__ actually called Student's __init__!

---

## Pro tips

1. always use super when inheritance is involved (if someone in the chain forgets, everything breaks)
2. avoid old-style classes (always inherit from some base type, e.g. object)

--- 

## Light-weight classes

`__slots__` let us avoid a dictionary allocation per instance.
by defining a list of member names, a small and fixed amount of memory per object is allocated.

Without `__slots__`:

	!python
	>>> import sys
	>>> class A(object):
	...     pass
	>>> sys.getsizeof(A())
    64
	>>> sys.getsizeof(A().__dict__)
	280

	
With `__slots__`:

	!python
	>>> class A(object):
	...     __slots__ = ()
	>>> sys.getsizeof(A())
	16
	
---

No attributes can be added to classes with `__slots__`:

	!python
	>>> class A(object):
	...     __slots__ = ('a', 'b')
	>>> a = A()
	>>> a.a = 10
	>>> a.c = 100
	Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'A' object has no attribute 'c'
	
Note: `__slots__` and inheritence - `__dict__` allocation will be prevented only if all classes in the inheritence tree define `__slots__`.


