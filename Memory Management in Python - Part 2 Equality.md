This is the second of a three-part series which covers various aspects of Python’s memory management. It started life as a conference talk I gave in 2021, titled ‘Pointers? In My Python?’ and the most recent recording of it can be found [here](https://youtu.be/EMWKQt53rUQ?t=2760).

Check out [Part 1](https://anvil.works/articles/pointers-in-my-python-1) of the series, or read on for an discussion of equality in Python, and why it’s more complicated than you might think!

## Object IDs, and why they matter

We ended Part 1 with the following question: how do we know when two Python objects are _really_ the same object in memory? If we do `b = deepcopy(a)`, how can we know for sure that it didn’t just create a new pointer instead of a whole new object? Well, Python has a couple of options, and they are `is` and `==`.

So, how do each of them work? Let’s dive in!

## The `is` comparator

One way we can ask ‘are `a` and `b` the same?’ in Python is to write this:

This will return us either `True` or `False` If the two objects really are the same object in memory, then `is` will tell us just that. One very useful thing that can help us to understand this, and to see it in action, is looking at the `id` of an object in Python.

### Object `id`s

Python has a built-in `id` function with the following properties:

-   `id(x)` is an integer
-   `id(x) == id(y)` if and only if `x` and `y` refer to the same object in memory
-   `id(x)` is constant for the **lifetime** of `x` - that is, as long as `x` remains in memory

There are many implementations of Python, and while the above three things should be true of the `id` function in each of them, they don’t all do it in the same way under the hood. Some implementations, such as CPython (the python interpreter written in C), use the object’s memory address as its `id` - but don’t assume that all implementations will!

For example, another Python implementation is Skulpt, the Python-to-JavaScript compiler which Anvil uses to run [Python client code](https://anvil.works/docs/client/python) in the browser (so that you can develop for the web without having to write JavaScript). Objects in JavaScript don’t have stable memory addresses, and JavaScript doesn’t expose a stable idenitifer for every object. So, Skulpt can’t use the object’s address in memory as its `id`.

For the rest of this article, we’ll be using examples generated using CPython, which does use an object’s memory address as its `id`. CPython is able to do this because, once an object in CPython exists, it can’t be moved around in memory. That means that using an object’s address as its `id` will guarantee a stable value for the lifetime of that object, unlike (for example) Skulpt.

So, let’s look at what happens when we check an object’s `id`!

Below, we define a list `a`, and create a new pointer to it by setting `b = a`. When we check their `id`s, we can see that they’re the same - `a` and `b` point to the same thing.

```
>>> a = ["a", "list"]
>>> id(a)
139865338256192

>>> b = a
>>> id(a), id(b)
(139865338256192, 139865338256192)
```

Trying the same thing with `c = a.copy()` shows that this creates a new list object; `a` and `c` have different `id`s.

```
>>> c = a.copy()
>>> id(a), id(c)
(139865338256192, 139865337919872)
```

So, now that we understand the `id` of an object in Python, we can see that the `is` comparator is actually really simple:

### `is` uses `id(x)`

Saying `a is b` is directly equivalent to saying `id(a) == id(b)`. When you call `is` on two objects, Python takes their `id`s and directly compares them. That’s it!

However, that isn’t the end of the story; Python has another, more flexible, way of defining object equality.

## The `==` comparator

Another way of asking ‘are `a` and `b` the same?’ in Python is to write the following:

Just like `is`, this will return us either `True` or `False`. But the way Python decides which to return is _different_ than the `is` operator, and this means that `==` and `is`can give us different results for the same objects.

To see this in action, let’s consider our familiar example, with `a` pointing to a list object list, `b` another pointer to that object, and `c` a pointer to a copy of that object:

```
>>> a = ["my", "list"]
>>> b = a
>>> c = a.copy()
```

With this setup, we can do the following comparisons, and see that `a ==c` is `True` while `a is c` is `False`:

```
>>> a == b
True
>>> a is b
True

>>> a == c
True
>>> a is c
False
```

Once again: what is going on here? How can two things be the same and not the same? The answer is that `is` and `==` are designed to serve two different purposes. `is` is for when you want to know if two pointers are pointing at the exact same object in memory; `==` is for when you want to know if two objects should be _considered_ to be equal.

### `is` uses `id(x)`

The answer is that `is` and `==` are designed to serve two different purposes. `is` is for when you want to know if two pointers are pointing at the exact same object in memory; `==` is for when you want to know if two objects should be considered to be _equivalent_.

### `==` uses `__eq__`

When you write `a == b`, you’re actually calling a **magic method**, also known as a **dunder method** (named for the **d**ouble-**under**score on each side). You might be familiar with some magic methods already, such as:

-   `__init__`, called when an instances of a Python class is initialised
-   `__str__`, called when you use the `str` function - e.g. `str(some_object)`
-   `__repr__`, similar to `__str__` but also called in other circumstances such as error messages

Magic methods are simply methods on a Python class, and the double underscores indicate that they interact with built-in Python functions. For example, overriding the `__str__` method on a Python class changes how the `str` function will behave if you call it on an instance of that modified class.

When it comes to `==` and `__eq__`, it’s easiest to understand with some examples. Let’s take a look!

### Overriding the `__eq__` method

Below, we define a class with its own `__eq__` method. Every `__eq__` method takes two arguments: `self` - the instance of the class in question - and the second argument (in the below code snippet we call it `other`) which is whatever object `self` is being compared to.

```
class MyClass:
  def __eq__(self, other):
    return self is other
```

So, if we evaluated the following:

```
>>> a = MyClass()
>>> a == 'some string'
False
```

… then this invokes MyClass’s `__eq__` method, with `a` as the `self` parameter and the string `'some string'` as the `other` parameter. Note that `other` doesn’t actually have to be an object of the same type as `self`! You can choose to make your `__eq__` method check for that if you like, but it’s not automatic.

In the above example, `MyClass.__eq__` uses the `is` comparator, which is equivalent to comparing the `id`s of each object. As it happens, this is actually the default behaviour for any user-defined class in Python.

So, what happens if we define some non-default behaviour?

Below, we define a class which takes a `name` argument (which will help us keep track of our instances in these examples) and has an `__eq__` method which indiscriminately returns `True`.

```
class MyAlwaysTrueClass:
  def __init__(self, name):
    self.name = name

  def __eq__(self, other):
    return True
```

Because we’ve overridden the `__eq__` method to _always_ return `True`, that means that all instances of this class will be considered equal under the `==` comparator - even when their names have different values!

```
>>> jane = MyAlwaysTrueClass("Jane")
>>> bob = MyAlwaysTrueClass("Bob")
>>> jane.name == bob.name
False

>>> jane == bob
True
```

Moreover, we can also see instances of `MyAlwaysTrueClass` behaving weirdly when compared to objects of different types:

```
>>> jane = MyAlwaysTrueClass("Jane")

>>> jane == "some string"
True
>>> jane == None
True
>>> jane == True
True
```

When comparing an instance of `MyAlwaysTrueClass` to _anything_ - not just instances of the same class - it’ll return `True`. This is because, as we said earlier, the `__eq__` method doesn’t, by default, check the types of the objects being compared.

So, we can make objects be equivalent to anything by overriding `__eq__`. We can also do the opposite:

```
class MyAlwaysFalseClass:
  def __init__(self, name):
    self.name = name

  def __eq__(self, other):
    return False
```

You might think this is more sensible, but consider:

```
>>> a = MyAlwaysFalseClass("name")
>>> a == a
False
```

Moreover, because the behaviour of `__eq__` is dependent on which object is `self` and which is `other`, we can get the following behaviour by using one instance of `MyAlwaysTrueClass` and one `MyAlwaysFalseClass`:

```
>>> jane = MyAlwaysTrueClass("Jane")
>>> bob = MyAlwaysFalseClass("Bob")
>>> jane == bob
True

>>> bob == jane
False
```

This is because the order in which we compare `jane` and `bob` matters. When we do `jane == bob`, we’re using the `__eq__` method defined in `MyAlwaysTrueClass` - `jane` is `self` and `bob` is `other`, and that method always returns `True`. The opposite is true when we write `bob == jane`, so we get a `False` value for that expression.

In summary: magic methods are a fun way to make Python do things that seem very strange, and should be tinkered with at your own risk!

___

So, what have we learned? We’ve covered `is` and `==`, the two ideas of equality in Python, and learned that `is` uses object `id`s and `==` uses the `__eq__` magic method. We’ve also explored how we can override `__eq__` to define our own ideas of equivalency, and seen why doing so can lead to some weird behaviour.

Earlier, we mentioned that `id(x)` is constant and unique ‘for the lifetime of `x`’, which is equivalent to saying ‘as long as `x` remains in memory’. That raises the question: once we’ve created an object `x`, how and why does its ‘lifetime’ end? Check out [Part 3](https://anvil.works/articles/pointers-in-my-python-3) of this series for the answer!

## More about Anvil

If you’re new here, welcome! [Anvil](https://anvil.works/) is a platform for building full-stack web apps with nothing but Python. No need to wrestle with JS, HTML, CSS, Python, SQL and all their frameworks – just **build it all in Python**.

## Learn More