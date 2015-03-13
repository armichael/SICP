---
author: Andrew Michael
title: SICP
...

------------------------------------------------------------------------

Building Abstractions with Procedures
=====================================

1.1: Elements of Programming
----------------------------

### Exercise 1.8 - Approximating cubic roots

$$\frac{\frac{x}{y^3}+2y}{3}$$

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-session="*guile*"}
  (define (cbrt x)
    (cbrt-iter 1.0 x))
  (define (cbrt-iter guess x)
    (if (good-enough? guess x)
        guess
        (cbrt-iter (improve guess x) x)))
  (define (improve guess x)
    (/ (+ (/ x (* guess guess)) (* 2 guess)) 3))
  (define (good-enough? guess x)
    (< (abs (- (expt guess 3) x)) 0.001))

  (cbrt 39837)
```

### Lexical Scoping - Internal Definitions

The above exercise, but organized under one definition:

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-session="*guile*"}
  (define (cbrt x)  
    (define (cbrt-iter guess x)
      (let ((improve (lambda (guess x)
                       (/ (+ (/ x (expt guess 2)) (* 2 guess)) 3)))
            (good-enough? (lambda (guess x)
                            (< (abs (- (expt guess 3) x)) 0.001))))
        (if (good-enough? guess x)
            guess
            (cbrt-iter (improve guess x) x))))
    (cbrt-iter 1.0 x))

    (cbrt 39837)
```

1.2: Procedures and Processes
-----------------------------

Procedural abstraction is the first big idea introduced by this book.
When people say "computer science isn't necessarily about computers or
science", this is what they mean.

### Recursion and Iteration

This chapter examines linear recursion and iteration, as well as the
term *procedure* and what it means for local and global transformations
of patterns. Specifically, recursive processes and linear processes have
different "shapes". They relate to the elements of computation
differently in how the procedure carries out through time and (memory)
space.

Recursive processes defer a chain of operations and then rapidly
contract. A function that calls itself as an operation generate a
recursive process.

Example: for computing a factorial ($n!$), the number of steps would
grow linearly with $n$.

An iterative process would not "contract" in this manner-- it simply
keeps track of a fixed number of variable values and then re-iterates,
generating a number of steps in finite-memory in which the variables are
updated according to a fixed rule.

#### Recursive Process vs Recursive Procedure

A function defined in terms of itself is not necessarily recursive. It
could be iterative as well. The difference is whether the function calls
itself as an argument in an expression (recursive) or simply calls
itself with adjustments to variable values (iterative). Ultimately, a
recursive process has to negotiate an increasing amount of information
(steps *and* memory grow proportional to the input) whereas an
interative process captures the state all at once in each step and
starts fresh. The information remains fixed, only the number of steps
remain proportional to input.

In other words, there is a difference between a recursive *procedure*
and a recursive *process* (SICP, p.45). A procedure being recursive
merely refers to the fact of its syntactical arrangement-- it is defined
in terms of itself. Whereas a *process* being recursive refers to its
behavior as it evolves.

An example:

``` {.scheme}
   ;; a recursive factorial function
    (define (fact-r n)
      (if (= n 0)
          1
          (* n (fact-r (- n 1)))))
    ;; an iterative factorial function
    (define (fact-i n)
      (letrec ((iter
             (lambda (n product)
               (if (= 1 n)
                   product
                   (iter (- n 1) (* product n))))))
        (iter n 1)))
```

`letrec` was not introduced at this point in the book. I used `letrec`
in place of a "helper" iteration function.

##### An aside on binding constructs:

binding
:   binding is the relationship between the name of something and its
    location in memory.

`let`
:   a derived form of `lambda`. `let` directly assigns an identifier to
    the result of an expression; `lambda` is passed identifiers. These
    are equivalent:

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  ((lambda (param1 param2 ... ) body) var1 var2 ... )
  (let ((param1 var1) (param2 var2) ... ) body)
```

`let*`
:   evaluates all declared bindings sequentially, that is, with respect
    to those that were declared before it. Normal `let` binds ids in
    parrallel.

`letrec`
:   allows the binding of recursive functions. See:
    <http://www.r6rs.org/final/html/r6rs/r6rs-Z-H-14.html>

As an aside, proper dialects of Scheme are *tail-recursive*-- that is, a
compiler trick allows for computationally cheap recursive function
calls. This is different from many imperative programming languages in
which iteration is almost always preffered to recursion.

Some Schemes feature a useful macro called "named let" that mirrors
particular uses of `letrec`.

#### Ackermann's function

This exercise happens to be the one that deters a lot of people from
continuing on. It can be maddening, for sure, if you attempt working it
out on paper for too long. The Ackermann function is defined as such:

``` {.scheme}
  (define (A x y)
    (cond ((= y 0) 0)
          ((= x 0) (* 2 y))
          ((= y 1) 2)
          (else (A (- x 1) (A x (- y 1))))))
(A 2 4)
```

Mathematically it can be represented as such:

$$ A(n) = \begin{cases} 0 &\mbox{if } y = 0 \\
                       2y & \mbox{if } x = 0 \\
                       2 & \mbox{if } y = 1 \\
    A((x-1),(A(x,y-1))) & \mbox{otherwise } \end{cases}$$

And as with any problem, imagine first the most simple cases, taking 0
or 1 for the variables. You see that it ends without recurse if both of
the variables are less than 2.

Now let's take 1 for $x$ and 2 for $y$. Here's an execution tree:

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  (A 1 2)
  (A 0 (A 1 1))
  (A 0 2)
  (* 2 2)
```

So $A(1,2)=4$.

Let's try 2 and 2.

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  (A 2 2)
  (A 1 (A 2 1))
  (A 1 2)
  ...
  (* 2 2)
```

Also 4.

Let's try 2 and 3.

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  (A 2 3)
  (A 1 (A 2 2))
  (A 1 (A 1 (A 2 1)))
  (A 1 (A 1 2))
  (A 1 4) ;; we know (A 1 2) -> 4
  (A 0 (A 1 3)) ;; here's where it gets interesting
  (A 0 (A 0 (A 1 2)))
  (A 0 (A 0 4))
  (A 0 8)
  (* 2 8) ;; -> 16
```

So the process appears to be exponential. But that's still not enough to
evoke fear in us, 16 is a small number.

But that's where many an unwitting soul has been lost-- try to evaluate
`(A 2 3)` on paper with pencil.

It turns out to be 65536.

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
 (A 2 4)
 (A 1 (A 2 3)
 (A 1 (A 1 (A 2 2)))
 (A 1 (A 1 (A 1 (A 2 1))))
 (A 1 (A 1 (A 1 2)))
 (A 1 (A 1 (A 0 (A 1 1))))
 (A 1 (A 1 (A 0 2)))
 (A 1 (A 1 4))
 (A 1 (A 0 (A 1 3)))
 (A 1 (A 0 (A 0 (A 1 2))))
 (A 1 (A 0 (A 0 (A 0 (A 1 1)))))
 (A 1 (A 0 (A 0 (A 0 2))))
 (A 1 (A 0 (A 0 4)))
 (A 1 (A 0 8))
 (A 1 16)
 (A 0 (A 1 15))
 (A 0 (A 0 (A 1 14)))
 (A 0 (A 0 (A 0 (A 1 13))))
 (A 0 (A 0 (A 0 (A 0 (A 1 12)))))
  ;; ... we eventually obtain 1024 at the far end
  ;; ... and then 'pop' the zeroes which multiply it by 2
  ;; ...
 (A 0 (A 0 (A 0 (A 0 4096))))
 (A 0 (A 0 (A 0 8192)))
 (A 0 (A 0 16384))
 (A 0 32768) ;; -> 65536

```

Try entering numbers too large and your scheme interpreter will likely
hang. This function produces enormous output with minimal input.

The one nice thing, and you'll have figured this out if you tried with
paper, if that you can repeatedly perform subsitutions, treating
expressions as their eventual results. Of course, you can do this with
most any recursively-defined structure.

It might help to imagine this process visually, or kinetically. Think of
the process whittling down the `x` values, each `x` having a
'petite-recursion' whittling down `y` and ultimately obtaining an
evaluation of `2` the deepest depth of its recursion, yielding `(A 0 2)`
as it begins to wind out of that depth. Now the function rapidly
contracts, 'popping' the zeroes, doubling this large number for each
accumulated `(A 0 ... )`. Our final evaluation will obviously be related
to 2, and because of the evaluation to `2` once $y=1$.

And because of the repeated doubling we can reasonably venture to guess
that the final solution will always be a number related to powers of 2
in some way.

The exercise asks us to consider the following procedures:

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  (define (f n) (A 0 n))
  (define (g n) (A 1 n))
  (define (h n) (A 2 n))
```

How would we give a mathematical definition for these functions?

Well, we know that $f(n)$ will always yield $2n$. It simply refers to
the base case, and no further proof is needed.

Now, $g(n) = A(1,n)$. For $n=1$, $g(1) = A(1,1) = 2$. Now we solve for
any $n$. `(A 1 n)` will evaluate the `else` branch of the conditional in
`A`, so we have `(A (- 1 1) (A 1 (- n 1)))`.

``` {.scheme .rundoc-block rundoc-language="scheme" rundoc-results="no"}
  (A (- 1 1) (A 1 (- n 1)))
  (A 0 (A 1 (- n 1)))
  (A 0 (A 0 (A 1 (- n 2))))
  (A 0 (A 0 (A 0 (A 1 (- n 3)))))
  ;; this continues until n reaches 1, and evaluates the expression to 2
  ;; thus, 2 is multiplied by 2 n times
```

So following the evaluation above, $g(n) = 2^{n}$.

Now we examine $h(n) = A(2,n)$. For $n=1$, $h(1) = A(2,1) = 4$, as
derived earlier. Now we solve for any $n$. Because $h(n)$ and $g(n)$ are
both derived from $A(x,y)$, it's likely we can use substitution to find
out how they are related.

``` {.scheme}
  (h n) ;; for n > 1
  (A 2 n)
  (A 1 (A 2 (- n 1)))
  ;; which is equivalent to
  (g (h (- n 1)))
  ;; so the evaluation would be 2 to the (h (- n 1))
  ;; we examine (h (- n 1))
  (g (g (h (- n 2))))
  (g (g (g (h (- n 3)))))
```

So we see clearly that since $h(n) = g(h(n-1))$, the ultimate expression
would evaluate as $g(n)$ nested $n$ times, with the inner-most argument
being 2: $g(g(g(...g(2)))$. So 2 to the 2 to the 2 to the 2... This is
called **tetration**, or iterated exponentiation (read more
[here](http://en.wikipedia.org/wiki/Tetration)). It is an operation that
outputs numbers *exponentially larger than exponentiation*. That is a
nearly incomprehensible rate of growth. It can be represented
symbolically as such:

$$Tet(a, n) = {}^na =\underbrace{a^{a^{a^{\dots^{a}}}}}
_{n \> times}$$

At this point you might be realize, as I did, that SICP is a partly a
math book in disguise.

### Fibonacci Numbers and Tree Recursion

Recall the definition of the Fibonacci sequence:

$$F(n) = \begin{cases} 0 &\text{if } n = 0 \\
                       1 & \text{if } n = 1 \\
    F(n-1) + F(n-2) &\text{otherwise } \end{cases}$$ So:
$1,1,2,3,5,8,13,21,34,55,89,\dots$

A recursive process to find the \$n\$th Fibonacci number:

``` {.scheme}
  (define (fibonacci n)
    (cond ((= 1 n) 1)
          ((= 2 n) 1)
          (else
            (+ (fibonacci (- n 1))
               (fibonacci (- n 2))))))
  (fibonacci 8)
```

And an linear iterative process (using a 'named let' construct):

``` {.scheme}
  (define (fibonacci n)
    (let loop ((a 1)(b 0)(count n))
      (if (= 0 count)
          b
          (loop (+ a b) a (- count 1)))))
  (fibonacci 8)
```

The recursive process is redundant, with a lot of needless computation,
growing exponentially in size with $n$.

We recall that successive Fibonacci numbers approximate $\varphi$.
Particularly, $F(n)$ is the closet integer to $\varphi^{n}/\sqrt{5}$
[^1] and $\varphi$ has the characteristic equation
$\varphi^{2} = \varphi + 1$, which tells us concretely right at once
that a process used to find Fibonacci numbers with recursive reference
to their definition will exponentiate and perform a lot of needless
computation.

Tree-recursive processes are useful for hierarchical data, but not for
numbers. Here, the iterative process introduces three state variables
and resolves in much fewer steps.

#### Counting Change

### Tree Recursion

### Orders of Growth

### Exponentiation

### GCD

### Primality

1.3 Higher-Order Procedures
---------------------------

### Procedures as Arguments

### Lambda

### General Methods

### Returned Values

Data Abstraction
================

Modularity, Objects, and State
==============================

[^1]: ``` {.example}
    SICP 2nd ed., page 49
    ```
