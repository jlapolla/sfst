# Software reuse

## The mental model

Imagine that there is something called a **_symbolspace_**. A
symbolspace is a lot like a dictionary data structure: you can look up
the value associated with a given key. For a symbolspace, the keys are
**_symbols_** and the values are **_software entities_**.

What is a **_software entity_**? Conceptually, a **_software entity_**
has a unique id, and holds a value. You can think of it as a memory slot
for storing a value.

A **_symbol_** is the textual name that we use to refer to a software
entity.

These definitions will become more clear as we go through the examples
in this section.

## Single symbolspace programs

Consider a simple programming language for a graphing calculator. The
language:

- Supports basic mathematical operators like `+`, `-`, `*`, and `/`.
- Supports flow of control statements like `if` and `while`.
- Supports basic comparison operators like `<`, `>`, `==`, and `!=`.
- Is limited to 26 variables: the letters `A` through `Z`.
- Has an assignment operator for storing the value of an expression in a
  variable.
- Allows the user to define and call named subroutines.
  - All subroutines share the same set of 26 global variables.
  - A subroutine can only modify global variables; it cannot return a
    value.
  - You cannot pass values to a subroutine. They only operate on global
    variables.

Let's take a look at factorial subroutine for our graphing calculator:

*Note: I'll write these examples in C++, but I'll be sure to follow the
restrictions of the graphing calculator language.*

```cpp
// Language: C++

void FACT() {

  Y = 1;
  while (X != 0) {
    Y = Y * X;
    X = X - 1;
  }
}
```

The subroutine `FACT` computes the factorial of `X` and stores the
result in `Y`.

Let's try out our `FACT` function for `X = 5`:

```cpp
// Language: C++

// Line 1
X = 5; // Line 2
FACT(); // Line 3
```

Now we'll sketch our mental model of the program. Consider the state of
the graphing calculator at line 1:

TODO: Insert diagram.

Notice that "X", "Y" and "FACT" are all symbols in the global
symbolspace. Each symbol maps to a single software entity in the
entities table. "X" and "Y" map to entities of "numeric" type, and
"FACT" maps to an entity of "subroutine" type.

The values for all numeric entities are unknown at this point, because
we don't know what values the graphing calculator currently has in
memory.

Now, let's step through the program. When we get to the end of line 2,
we have:

TODO: Insert diagram.

The value stored in `X` has updated to 5, as expected.

Stepping to the end of line 3, we have:

TODO: Insert diagram.

The value stored in `Y` is 120, which is the factorial of 5. The value
stored in `X` is 0.

To summarize, here is how we use the `FACT` subroutine:

- Store the value you want to take the factorial of in `X`.
- Run `FACT`.
- Retrieve the result from `Y`.

### Reuse

Let's say we're working on another subroutine, `FOO`, and we want to
reuse our `FACT` subroutine in `FOO`. At some point in `FOO`, we'd like
to take the factorial of `Z` and store the result in `A`, but we're
already using `X` and `Y` to store other results. We have two options:

- Swap variables around to clear out `X` and `Y`, then store the value
  of `Z` in `X` and run `FACT`.
- Copy the text of `FACT` to a new subroutine `FACT2`, which takes the
  factorial of `Z` and stores it in `A`, leaving `X` and `Y` alone.

The thing I'd like to point out is: if we want to reuse the `FACT`
algorithm on a different software entity (i.e. we want to compute the
factorial of something other than `X`), we have to copy and paste the
code of `FACT` into a new subroutine, `FACT2`, and then modify the text
of the code to substitute the symbols `Z` and `A` for `X` and `Y`.

### Connection: CPU instructions

The language we've described is similar to the way CPU instructions work
at the hardware level: you load a value in CPU register `X`, you run the
instruction, and you retrieve the result from register `Y`. Each
instruction has its own set of registers that it uses for different
purposes, all instructions share the same register bank, and you cannot
apply an instruction to a different set of registers than the set it was
designed to work with.

Actually, our graphing calculator language has one feature that CPU
instructions lack: a user can define new subroutines in the graphing
calculator, but a user cannot define new instructions for a CPU; they
are limited to the existing CPU instruction set.

