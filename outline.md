# 1. Modelling

## Variable

### Definition

A **variable** is an abstract entity which has a name and holds a value.

### Notation

A variable is denoted by its name. For example:

- A variable named "a" is denoted by `a`.
- A variable named "foo" is denoted by `foo`.

A variable with a value is denoted as a name / value tuple. For example:

- A variable named "a" with value "0" is denoted by `(a, 0)`.
- A variable named "foo" with value "bar" is denoted by `(foo, bar)`.

## Model

### Definition

A **model** is a set of relevant variables.

### Notation

A model is denoted by a list of variables in set notation. For example:

- A model consisting of three variables, `a`, `b`, and `c`, is denoted
  by `{a, b, c}`.
- A model consisting of two variables, `foo` and `bar`, is denoted by
  `{foo, bar}`.

## Modelling

### Introduction

It is often helpful to define a model to more effectively analyze a
piece of software. Models vary in their level of abstraction and detail.

For instance, consider a non-recursive factorial function written in
Python:

```python
def factorial(n):
    result = 1
    while n != 0:
        result = result * n
        n = n - 1
    return result
```

To anaylze this function for correctness, a model `{n, result}` will
suffice.

Now consider a recursive factorial function written in Python:

```python
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)
```

A model `{n, return_value}` is not sufficient to analyze the recursive
function for correctness. We need a model that includes the value of `n`
in each recursive call, and accounts for the return value from each
recursive call. For example, a model consisting of a call stack where
each stack frame contains two variables `n` and `return_value` will
suffice.

We can go deeper and model the Python runtime engine itself. For
example, we could model it as a call stack, a free store, and a program
under execution.

If we were interested in analyzing the memory usage of a particular
Python implementation, for example CPython, we might model the software
as an array of memory elements and a program counter, where each memory
element contains either program data or processor instructions (much
like assembly language).

All of these are potentially useful models for analyzing the above
factorial functions. The selection of a model is subjective. As a
practical matter, you should select the simplest model that allows you
to analyze the software properties you are interested in.

### Definition

**Modelling** is the act of defining a model within which to analyze a
piece of software. 

## State

### Definition

A **state** is an instance of a model where each variable holds exactly
one value.

### Notation

A state is denoted by a list of variables with values in set notation.
For example:

- A state with variables `(a, 0)`, `(b, 1)`, and `(c, 3)` is denoted by
  `{(a, 0), (b, 1), (c, 3)}`.
- A state with variables `(foo, "foo_value")`, `(bar, "bar_value")` is
  denoted by `{(foo, "foo_value"), (bar, "bar_value")}`.

## Code

### Definition

A **code** is a unit of software.

## Synthesis

You can think of a **model** as an abstract machine with a fixed number
of **variable** elements. When a **model** executes a **code** it
changes **state** in response to the executed **code**.

The number of **variables** in the model does not change in response to
executed **code**; it remains fixed. The only things that change are the
values held by the **variables**.

## Terminology

### Variables

A variable **holds** a value. Alternatively, we can say a variable
**has** a value.

### Model coincidence

Alternative models of the same software system are said to **coincide**.
Coinciding models may differ in level of detail, level of abstraction,
or type of abstraction.

### Superset model

Model `A` is a **superset** of model `B` if and only if every element of
`B` is also an element of `A`.

### Subset model

Model `A` is a **subset** of model `B` if and only if every element of
`A` is also an element of `B`.

### Equal model

Model `A` is **equal** to a model `B` if all of the following are true:

- `A` is a superset of `B`.
- `B` is a superset of `A`.

In other words, `A` and `B` are equal if they contain the same
variables.

### State and model relationships

State `x` is **in** model `A` if and only if state `x` assigns one and
only one value to each variable in model `A`. Notice that state `x` may
include variables that are not in model `A`.

State `x` is **contained in** model `A` if and only if all of the
following are true:

- State `x` assigns one and only one value to each variable in model
  `A`.
- All varaibles in state `x` are in model `A`.

### State coincidence

If all of the following are true:

- State `x` is in model `A`.
- State `y` is in model `B`.
- Models `A` and `B` coincide.
- States `x` and `y` represent the same situation.

Then states `x` and `y` **coincide**.

State coincidence is an abstract concept. For example, consider the case
where:

- Model `A` is `{foo, bar}`.
- State `x` is `{(foo, 0), (bar, 1)}`.
- Model `B` is an array of memory elements and a symbol table pointing
  into the memory array.
- State `y` has a symbol table with `foo` pointing to a memory element
  with value `0` and `bar` pointing to a memory element with value `1`.

In this case, even though models `A` and `B` do not have a strict
superset / subset relationship, the two states `x` and `y` still
represent the same situation. State `y` contains additional information
(e.g. the memory location of the symbols `foo` and `bar`), but
conceptually states `x` and `y` coincide.

# 2. Roles

## Supplier and client

Code that is used by other code is a **supplier** of the other code.

Code that uses other code is a **client** of the other code.

For example:

```python
# Python
def f(x):
    return x + 1

def g(x):
    return f(x) * 2

def h(x):
    return f(x) + g(x)
```

In this example:

- `f` is a supplier of `g`. `g` is a client of `f`.
- `f` is a supplier of `h`. `h` is a client of `f`.
- `g` is a supplier of `h`. `h` is a client of `g`.

Notice that the terms "supplier" and "client" are relative. In some
contexts, a code acts as a supplier. In other contexts, that same code
acts as a client. For example:

- In the context of `f`, `g` is a client.
- In the context of `h`, `g` is a supplier.

## Developer and consumer

A **developer** is a person who writes code that other people use.

A **consumer** is a person who uses code written by other people.

Developer and consumer are the human counterparts to supplier and
client. Like supplier and client, developer and consumer are relative
terms.

## Undefined behavior

To understand what **undefined** means in software, it is helpful to
first review what undefined means in mathematics.

### Undefined in mathematics

Consider a piecewise function `f` in mathematics:

```
f(x) = {
  0, if x < 0
  1, if 0 <= x < 1
}
```

From the definition of `f`, we know that `f(-12) = 0` and `f(0.5) = 1`,
but what is `f(2)`? We say that `f(2)` is **undefined** because there is
no value defined for `x = 2` in the piecewise definition of `f` (in
mathematical terms: `2` is not in the "domain" of `f`).

Notice that `undefined` is not a value; it is not correct to say `f(2) =
undefined`. All we can say is that `f(2)` cannot be evaluated to a
value.

A more realistic example is a function `div` that divides two numbers:

```
div(x, y) = {
  x / y, if y != 0
}
```

You've probably heard that "dividing by zero is undefined". The
piecewise definition of `div` makes this fact explicit. We cannot
evaluate `div(3, 0)`.

We could make a new version of `div`, `div2`, that arbitrarily defines a
value for when `y == 0`:

```
div2(x, y) = {
  x / y, if y != 0
  4, if y == 0
}
```

With this definition, `div2(3, 0) = 4`. There is nothing special about
the value `4`; we could have used `5` or `6` just as easily. Of course,
our made up `div2` function has no mathematical significance.

### Undefined in software

In software, **undefined** behavior means "any arbitrary thing can
happen."

For example, let's translate our mathematical function `f` into Python:

```python
def f(x):
    if x < 0:
        return 0
    if x < 1:
        return 1
    # Some arbitrary behavior here.
```

Consider again our mathematical specification for `f`. If the consumer
calls `f(-12)` the consumer expects the return value to be `0` according
to the specification. If the consumer calls `f(0.5)` the consumer
expects the return value to be `1` according to the specification.

But if the consumer calls `f(2)`, then it's impossible for the developer
to know what behavior the consumer desires, because the developer did
not specify what `f(2)` was supposed to do. Since the value of `f(2)` is
not defined in the specification, the developer is free to do anything
he wants when the consumer calls `f(2)`. Therefore, we say that calling
`f(2)` results in **undefined** behavior.

The simplest alternative for the developer is to just leave the
undefined branch blank:

```python
def f(x):
    if x < 0:
        return 0
    if x < 1:
        return 1
    # Blank.
```

The developer could also return an arbitrary value:

```python
def f(x):
    if x < 0:
        return 0
    if x < 1:
        return 1
    return 4 # Return arbitrary value.
```

Or, the developer could loop forever:

```python
def f(x):
    if x < 0:
        return 0
    if x < 1:
        return 1
    while true: # Loop forever
        pass
```

The developer could also make `f(2)` do the any of following:

- Put the program in an invalid state (i.e. a state that violates some
  invariant).
- Terminate the program.
- Consume all available memory.
- Shut down the computer.
- Crash the airplane (in avionics software).
- Launch the missiles (in missile control software).

The possibilities of undefined behavior are endless.

## Errors and faults

A **fault** is an instance of an **error**. In other words, an error is
a type of fault.

For example, "division by zero" is an **error**. Executing a code that
divides by zero produces a "division by zero **fault**".

### Examples

#### Instructions (examples 1 - 2)

Identify the faults and errors in the following codes.

#### Example 1

```python
# Python
x = 3 / 0
y = 4 / 0
```

At runtime, this code produces two faults:

- Division by zero fault on line 2.
- Division by zero fault on line 3.

This code has one error:

- Division by zero.

#### Example 2

```python
# Python
def add_3(x):
    return x + 3

x = add_3(1) / 0
y = add_3()
z = 3 / 0
```

At runtime, this code produces three faults:

- Division by zero fault on line 5.
- Wrong number of arguments fault on line 6.
- Division by zero fault on line 7.

This code has two errors:

- Division by zero.
- Wrong number of arguments.

## Defects, errors, and faults

# 3. Specification

# Development by contract

# Trivial implementation

# Error handling

Erase from your mind the notion of "error handling". The only true
"error" that can occur is when a client invokes a supplier in a state
that causes undefined behavior. By definition, this represents a defect
in the client. There is nothing the developer can do to "handle" the
client's defect; it is the consumer's responsibility to fix the client.

Any so-called "errors" that are handled expicitly in code are better
thought of as full-fledged features. For example, catching an exception
is an intentional feature. Throwing an exception is an intentional
feature. Setting `errno` or some other error indication is an
intentional feature. These are all well-defined behaviors, behaviors
that the consumer can rely upon, and behaviors that the developer must
continue to support. Therefore, they are features, not defects.

As for the various "error handling" mechanisms available in programming
languages, it is best to think of these as additional means of
communication between supplier and client. Throwing an exception is
really just another flow of control mechanism. `errno` is just another
variable available for communication. These are not errors.

To reiterate: the only true "error" in code is when undefined behavior
occurs due to a defect in the code.

There is another type of conflict that is often incorrectly classified
as an "error": when a supplier does not do what the consumer expects. In
this case, the developer has two options:

- Add a feature to do what the consumer expects. This means more work
  and maintenance for the developer.
- Explicitly declare that the supplier shall produce undefined behavior
  when it is invoked the way the consumer is invoking it. This restricts
  the supplier's domain, and it means more work and maintenance for the
  consumer.




## Defects, errors, and faults

There is only one error:

- Undefined behavior.

# UNORGANIZED

## Change code into content

Example: recipe book. Initially "Pea Soup" is a class that inherits from
"Recipe". Eventually we need to define new recipes at runtime (they may
be hard-coded, or loaded from persistent storage), rather than making a
class for each new recipe.

## 