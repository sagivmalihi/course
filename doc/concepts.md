
# Functional / Lazy Programming Concepts in Python

---

## Decorators

For when you want to encapsulate logic that doesn't belong inside a function

    !python
    def fibonacci(n):
        if n < 2: return 1
        return fibonacci(n-1) + fibonacci(n-2)

    >>> fibonacci(10)
    89
    >>> fibonacci(40) # does not return

---

## Decorators
     
    !python
    def cached(f):
        f.cache = {}
        def cached_f(*args):
            if args not in f.cache:
                f.cache[args] = f(*args)
            return f.cache[args]
        return cached_f

---
    
## Decorators

    !python
    @cached
    def fibonacci(n):
        if n < 2: return 1
        return fibonacci(n-1) + fibonacci(n-2)

    >>> fibonacci(10)
    89
    >>> fibonacci(40)
    165580141    
    >>> fibonacci(1000)
    70330367711422815821835254877183549770181269836358732742604905087154537118196933579742249494562611733487750449241765991088186363265450223647106012053374121273867339111198139373125598767690091902245245323403501L

---

## Decorators

Other examples 

- Type checking
- Function registration
- Output formatting
- Retries

---

## Some Famous Decorators

    !python
    class A(object):
        _a = 1
        def __init__(self, a): self._a = a
        def foo(self):         return self._a
        @classmethod
        def bar(cls):          return cls._a
        @staticmethod
        def bla():             return 3
        @property
        def a(self):           return self._a
    >>> inst = A(2)
    >>> inst.foo()
    2
    >>> inst.bar()
    1
    >>> inst.bla()
    3
    >>> inst.a
    2

---

## Iterators

For whenever you feel like feeling any loop which isn't a "for loop"

    !python
    class N(object):
        def __init__(self):
            self.i = 0
        def __iter__(self): # This class is usable in 'for' statements
            return self
        def next(self): # iterator protocol
            self.i += 1
            return self.i

    >>> n = N()
    >>> n.next()
    1
    >>> n.next()
    2
    for i in N():
        if i > 2: break
        print i
    1
    2

--- 

## Generators

Easy way to create iterators

    !python
    def N2():
        i = 0
        while True:
            i += 1
            yield i
    
    >>> n = N2()
    >>> n.next()
    1
    >>> n.next()
    2
    >>> n.next()
    3
    for i in N2():
        if i > 3: break
        print i
    1
    2
    3

--- 

## Generator Comprehension

- For when you want to iterate over large amounts of data without reading into memory
- When declaring infinite data structures

Example:

    !python
    >>> [fibonacci(x) for x in range(10)]
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
    >>> all_fibonacci = (fibonacci(x) for x in N())
    >>> from itertools import takewhile
    >>> small_fibonacci = takewhile(lambda x: x < 100, all_fibonacci)
    >>> small_fibonacci
    <itertools.takewhile object at 0x101d9d098>
    >>> list(small_fibonacci)
    [1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

---

## co-routines

    !python
    def average(n):
        first = yield; lst = [first]
        while True:
            try:
                x = yield float(sum(lst)) / len(lst)
            except Exception as e:
                print "something horrible happened: {}".format(e)
            else:
                lst.append(x)
                if len(lst)>n:
                    lst.pop(0)

    >>> avg = average(5)
    >>> avg.next()
    >>> avg.send(1)
    1.0
    >>> avg.throw(Exception("boom!"))
    something horrible happened: boom!
    1.0

--- 

    !python
    def consumer(gen):
        def wrapper(*args, **kwargs):
            x = gen(*args, **kwargs)
            x.next()
            return x
        return wrapper

    @consumer
    def average(n):
        ...
    
    >>> avg = average(5)
    >>> avg.send(1)
    1.0
    >>> avg.throw(Exception("boom!"))
    something horrible happened: boom!
    1.0

    

---

## Context Managers

For encapsulating resource allocation / cleanup (i.e. try/except clauses)

Examples:

    !python
    with open("/tmp/bla") as f: # closes file on exit
        f.write("Hello, world!")
    with lock: # releases lock on exit
        list1.append(1)
        list2.append(1)

Instead of
    
    !python
    lock.acquire()
    try:
        ...
    finally:
        lock.release()

--- 

## Context Managers

Example protocol implementation

    !python
    class logged(object):
        def __init__(self, message):
            self.message = message
        def __enter__(self):
            print "{}: starting...".format(self.message)
        def __exit__(self, et, ev, tb):
            if et is not None:
                print u"{}: failed ({})".format(self.message, ev)
                return True # silence the exception
            else:
                print u"{}: Success!".format(self.message)

---
    
    !python
    >>> import time
    >>> with logged("sleeping"):
    ...     time.sleep(1)
    ... 
    sleeping: starting...
    sleeping: Success!

    >>> with logged("sleeping"):
    ...     time.sleep(0.5)
    ...     raise Exception(u"!לא רוצים לישון, רוצים להשתגע")
    ... 
    sleeping: starting...
    sleeping: failed (!לא רוצים לישון, רוצים להשתגע)
    >>> 

---

## @contextmanager

The easy way to create a Context Manager for a Generator and a Decorator

    !python
    from contextlib import contextmanager

    @contextmanager
    def logged(message):
        print "{}: starting...".format(message)
        try:
            yield
        except Exception as e:
            print "{}: failed :(".format(message)
        else:
            print "{}: Success!".format(message)

---

    !python
    class GeneratorCM(object):
        def __init__(self, gen):
            self._gen = gen
        
        def __enter__(self):
            return self._gen.next()

        def __exit__(self, exception_type, exception_value, exception_trace_back):
            if exception_type is None:
                try:
                    self._gen.next()
                except StopIteration:
                    pass
                else:
                    raise Exception("Generator didn't exit")
            else:
                try:
                    self._gen.throw(exception_value)
                except StopIteration:
                    return True 
                except Exception:
                    return False
                else:
                    raise Exception("Generator didn't exit")

    def contextmanager(func):
        def helper(*args, **kwargs):
            return GeneratorCM(func(*args, **kwargs))
        return helper

