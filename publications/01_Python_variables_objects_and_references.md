# Python Variables, Objects and References: The Mental Model You Actually Need

## Introduction

A common way of thinking about a Python variable is as a box that stores a value. You write `x = 10`, and the idea is that Python creates a box called `x` and puts `10` inside it. It is a useful model for getting started, but it becomes misleading when you need to understand how Python actually handles objects and references. 

```py
a = [1, 2, 3] 
b = a 
b.append(4) 
print(a) # [1, 2, 3, 4] 
print(b) # [1, 2, 3, 4] 
```

If `a` were simply a box containing the list, changing `b` should have left `a` untouched. But it did not.

The better mental model is that **Python variables are names bound to objects**. `a` is not a container holding the list. It is a name that refers to a list object. When we write `b = a`, Python does not create another list. It binds another name, `b`, to the same object.

![][image1]

So when `b.append(4)` runs, the list itself is mutated. Since both `a` and `b` refer to that same object, the change is visible through either name.

This distinction between names, objects, and references is the foundation for understanding Python's behaviour around assignment, mutation, rebinding, identity, and function arguments. Once this mental model is clear, many Python behaviours that initially seem surprising become straightforward to reason about.

## How Python Names and Objects Really Work

A useful starting point is to separate **names, objects, and references**. A name is an identifier in a namespace dictionary that stores a reference (memory address) to a value, while an object is the actual memory allocation containing data, type information, and a reference count that Python evaluates at runtime. A binding connects the name to the object. 

Consider:

```py
x = 10
y = {"name": "John"}
z = y	
z["age"] = 30
print(y) #{'name': 'John', 'age': 30}

y = {"age": 36}
print(y) #{"age": 36}
```

When Python executes an assignment such as `x = 10`, it is useful to think in terms of **binding**, not storage. Python evaluates the expression on the right, obtains or creates an object, and then binds the name on the left to that object. The same happens with `y`: the name `y` is bound to a dictionary object. 

Python keeps track of these binding relationships through **namespaces**, which can be thought of as mappings between names and the objects to which they are bound. 

> You can access the namespace lists in Python using the `dir()`, `globals()`, and `locals()` built-in functions.

The important part happens at `z = y`. Python does not create another dictionary or copy the existing one. It binds `z` to the **same object** that `y` already refers to.

This means two names can refer to one object. This is known as **aliasing**. We can observe shared identity with `is`: `y is z` returns `True`.

Now `z["age"] = 30` changes the dictionary object itself. Since `y` and `z` refer to that same object, the change is visible through both names. This is **mutation**. 

But `y = {"age": 36}` does something fundamentally different. The original dictionary is not changed. Instead, `y` is **rebound** to a different object.

This is where **mutation** and **rebinding** become important. Mutating an object changes the object. Rebinding changes what a name refers to. Once those two operations are distinguished, behaviour that might otherwise seem surprising becomes much easier to trace. The next step is to understand why some objects can be mutated while others cannot, which brings us to **mutable and immutable objects**. 

## Mutable and Immutable Objects

Objects are designed to behave differently when it comes to change. Some can be modified after creation, while others cannot. This distinction is useful because programs need both mutable objects for data that changes and immutable objects for values that should remain stable, with different implications for sharing, performance, and safety. 

To say an object is **mutable** means that the object can be changed after it has been created. A list is a good example:

```py
items = [1, 2, 3]
items.append(4)
```

The name `items` still refers to the same list object, but the contents of that object have changed. That is mutation. 

An **immutable** object, on the other hand, cannot be changed after it is created. Common immutable objects include integers, floats, strings, tuples, and booleans. This does not mean they are somehow fixed values stored inside variables. It means that the particular object itself cannot be modified. 

With an immutable object, an operation that appears to change the value must instead produce or refer to another object:

```py
a = 10
b = a
a += 5
```

Here, `a += 5` does not modify the integer object `10`. Since integers are immutable, `a` ends up bound to `15`, while `b` continues to refer to `10`.

This is why statements such as `x += ...` can sometimes be misleading if we only look at the syntax. The result depends partly on the object's behaviour. For a mutable object, an augmented operation may modify the existing object. For an immutable object, it generally results in a new object and a rebinding of the name.

In Python, `type()` tells us an object's type, `id()` gives us its identity, and `is` lets us test whether two names refer to the same object: 

```python
a = [1, 2]
b = a
c = [1, 2]

print(type(a))  # <class 'list'>

print(a is b)   # True
print(a is c)   # False
print(a == c)   # True

print(id(a))    # 139895621863808
print(id(b))    # 139895621863808
print(id(c))    # 139895621865728
```

Starting with `a = [1, 2]`, Python creates a list object and binds `a` to it. When we write `b = a`, Python does not create another list. It simply binds `b` to the same object. That is why `a is b` returns `True`. Both names refer to the same object. 

Now look at `c = [1, 2]`. Although `c` refers to a list containing the same values, Python creates a **separate list object**. Therefore, `a is c` is `False`: they are different objects. But `a == c` is `True` because the two lists contain equal values. 

> **Note**: The `==` operator (Equality) compares the actual data or values inside objects, while the `is` operator (Identity) compares the memory addresses of two objects (using the `id()` function). The actual numbers returned by `id()` are not important and can differ between runs.

`id(a)` is equal to `id(b)` because `a` and `b` point to the same object in memory, but `id(a)` is not equal to `id(c)` because they point to two different memory objects, even though their values are the same.

> [!TIP]
> **Python Optimization Note:** If you test `a = 10; b = 10; a is b`, you will get `True`. This is because Python optimizes memory by caching small integers in the range `-5` to `256` and certain strings (known as interning), pointing them to the same pre-allocated memory addresses. If you try the same experiment with larger numbers (e.g., `300`), you will see `False` as expected.

Now we can move to copying. If assigning another name can make two names refer to the same object, the natural question is: how do we create a separate object instead? 

## Copying Objects in Python

So far, assignment has shown that giving one name to another does not create a new object. If `b = a`, both names refer to the same object. That is useful when shared state is intentional, but sometimes `b` needs its **own object** so that changes to it do not affect `a`. This is where copying becomes important. 

To create a separate object, it must be explicitly copied:

```py
a = [1, 2, 3]
b = a.copy()
b.append(4)

print(a)  # [1, 2, 3]
print(b)  # [1, 2, 3, 4]
```

However, `copy()` creates a **shallow copy**. It creates a new outer object, but objects nested inside it may still be shared.

```py
a = [[1, 2], [3, 4]]
b = a.copy()

b[0].append(5)
print(a)  # [[1, 2, 5], [3, 4]]
```

The outer lists are different, but the inner list `[1, 2]` is shared. Changing that inner object therefore affects what is observed through both names.

When the nested objects also need to be independent, a `deepcopy()` can be used:

```py
import copy

a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)

b[0].append(5)

print(a)  # [[1, 2], [3, 4]]
print(b)  # [[1, 2, 5], [3, 4]]
```

## Common Misconceptions and Mistakes

Most confusion around Python variables comes from mixing up **names, objects, and bindings**. The common mistakes are easier to see once that distinction is clear.

* **"Variables store values."** More accurately, names are bound to objects. This explains why multiple names can refer to the same object.  
* **"`b = a` copies `a`."** It does not. It binds `b` to the same object as `a`. A copy must be explicitly created.  
* **"Assignment changes an object."** Assignment changes a **binding**. Mutation changes the object.  
* **"Python passes variables by reference."** This is an unhelpful description. When a function is called, its parameter is bound to the same object supplied by the caller. Whether the caller observes a change depends on what happens next.  
* **"Equal objects are the same object."** `==` tests equality; `is` tests identity. Two separate objects can be equal without being the same object.  
* **"Immutable means the variable cannot change."** Immutability describes the **object**, not the name. A name referring to an integer can still be rebound to another integer.  
* **"`id()` determines equality."** `id()` identifies an object. It is not a replacement for `==`. Use `==` for equality and `is` when identity matters.  
* **"`+=` always mutates the object."** Not necessarily. Its behaviour depends on the object's type. A list can be modified in place, while an integer is immutable, so the name is rebound to another object.  
* **"A mutable default argument creates a fresh object for every call."**  It does not. Default values are created when the function is defined, so a mutable default can be shared across calls.

The recurring mistake behind all of these is the same: treating a name as if it were the object itself. A better habit is to ask: *What object does this name refer to, and is this operation mutating that object or changing the binding?*

## Why This Mental Model Matters

The distinction between **names, objects, references, mutation, and rebinding** may seem basic, but it becomes increasingly important as Python code becomes more complex. It is the difference between knowing what a line of code does and being able to **predict what a larger program will do**.

This mental model becomes especially valuable when working with:

1. **Functions and arguments.** Understanding what happens when an object is passed into a function prevents confusion about whether a function will change the caller's data or only rebind its local name.

2. **Collections and shared state.** Lists, dictionaries, sets, and nested data structures are frequently passed around different parts of an application. Knowing when objects are shared helps prevent unexpected changes.

3. **Copying and data manipulation.** Understanding shallow copies, deep copies, and nested references makes it easier to decide when data should be shared and when it should be independent.

4. **Classes and object oriented programming.** Attributes, instance state, methods, and object relationships all build on the same name object model.

5. **Closures and scope.** Local variables, enclosing variables, `global`, `nonlocal`, and late binding become much easier to understand when names are seen as bindings rather than containers.

6. **Debugging.** When Python produces an unexpected result, this model provides a practical way to trace what happened: identify the object, identify the names referring to it, and determine whether the object was mutated or a name was rebound.

7. **Advanced Python and frameworks.** Decorators, callbacks, generators, caching, data pipelines, concurrency, and many framework behaviours rely on objects being passed around and references being shared.

Understanding what object exists, which names refer to it, and whether an operation mutates the object or rebinds a name makes advanced Python easier to understand and debug.

These concepts become especially important in production systems, libraries, APIs, data pipelines, and machine learning applications, where objects are frequently passed, shared, modified, and copied.

## Practical Experiments

The accompanying notebook contains the concepts covered in this article, along with additional tests and experiments that allow each behaviour to be observed directly in Python. 

[Link](http://link).

## Conclusion

Python becomes much easier to reason about once names, objects, and bindings are kept distinct. Assignment binds names to objects, multiple names can refer to the same object, mutation changes an object, and rebinding changes what a name refers to. Mutability, copying, identity, and equality all follow naturally from these relationships. The accompanying notebook provides additional experiments to reinforce these concepts through direct observation.

## References

[https://docs.python.org/3/reference/executionmodel.html](https://docs.python.org/3/reference/executionmodel.html)  
[https://docs.python.org/3/library/copy.html](https://docs.python.org/3/library/copy.html)  
[https://docs.python.org/3/reference/datamodel.html](https://docs.python.org/3/reference/datamodel.html)

[image1]: <data:image/png;base64,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>