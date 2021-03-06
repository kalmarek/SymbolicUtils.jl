@def title = "SymbolicUtils.jl — Symbolic programming in Julia"
@def hasmath = false
@def hascode = true
<!-- Note: by default hasmath == true and hascode == false. You can change this in
the config file by setting hasmath = false for instance and just setting it to true
where appropriate -->

~~~
<h1>SymbolicUtils.jl &mdash; home</h1>
~~~

~~~
<p style="font-size: 1.25em; line-height: 1.67em; text-align: center; margin: 1em 0; color: #111;">
<a href="https://github.com/JuliaSymbolics/SymbolicUtils.jl">SymbolicUtils</a> is an practical symbolic programming utility in Julia. It lets you <a href="#symbolic_expressions">create</a>, <a href="#rule-based_rewriting">rewrite</a> and <a href="#simplification">simplify</a> symbolic expressions, and <a href="/codegen/">generate Julia code</a> from them.
</p>
~~~

**Features:**

- Fast expressions
- A [combinator library](#composing_rewriters) for making rewriters.
- A [rule-based rewriting language](#rule-based_rewriting).
- Type promotion:
  - Symbols (`Sym`s) carry type information. ([read more](#symbolic_expressions))
  - Compound expressions composed of `Sym`s propagate type information. ([read more](#symbolic_expressions))
- Set of extendable [simplification rules](#simplification).


~~~
<br>
<h1>
 Getting Started
</h1>
~~~

**Table of contents**

\tableofcontents <!-- you can use \toc as well -->

## Creating symbolic expressions

First, let's use the `@syms` macro to create a few symbols.

```julia:syms1
using SymbolicUtils

@syms w z α::Real β::Real

(w, z, α, β) # hide

```

Type annotations are optional when creating symbols. Here `α`, `β` behave like Real numbers. `w` and `z` behave like `Number`, which is the default. You can use the `symtype` function to find the type of a symbol.

```julia:symtype
using SymbolicUtils: symtype

symtype(w), symtype(z),  symtype(α), symtype(β)
```

Note however that they are not subtypes of these types!

```julia:symtype2
@show w isa Number
@show α isa Real
```

(see [this post](https://discourse.julialang.org/t/ann-symbolicutils-jl-groundwork-for-a-symbolic-ecosystem-in-julia/38455/13?u=shashi) for why they are all not just subtypes of `Number`)

You can do basic arithmetic on symbols to get symbolic expressions:

```julia:expr
expr1 = α*sin(w)^2 +  β*cos(z)^2
expr2 = α*cos(z)^2 +  β*sin(w)^2

expr1 + expr2
```


**Function-like symbols**

Symbols can be defined to behave like functions. Both the input and output types for the function can be specified. Any application to that function will only admit either values of those types or symbols of the same `symtype`.

```julia:syms2
using SymbolicUtils
@syms f(x) g(x::Real, y::Real)::Real

f(z) + g(1, α) + sin(w)
```


```julia:sym3
g(1, z)
```


This does not work since `z` is a `Number`, not a `Real`.

```julia:sym4
g(2//5, g(1, β))
```

This works because `g` "returns" a `Real`.


## Expression interface

Symbolic expressions are of type `Term{T}`, `Add{T}`, `Mul{T}` or `Pow{T}` and denote some function call where one or more arguments are themselves such expressions or `Sym`s.

All the expression types support the following:

- `istree(x)` -- always returns `true` denoting, `x` is not a leaf node like Sym or a literal.
- `operation(x)` -- the function being called
- `arguments(x)` -- a vector of arguments
- `symtype(x)` -- the "inferred" type (`T`)

See more on the interface [here](/interface)


## Term rewriting

SymbolicUtils contains [a rule-based rewriting language](/rewrite/#rule-based_rewriting) for easy pattern matching and rewriting of expression. There is also a [combinator library](/rewrite/#composing_rewriters) to combine rules to chain, branch and loop over rules.

## Simplification

By default `*` and `+` operations apply the most basic simplification upon construction.

The `simplify` function applies a built-in set of rules to rewrite expressions in order to simplify it.

```julia:simplify1
simplify(2 * (w+w+α+β + sin(z)^2 + cos(z)^2 - 1))
```

The rules in the default simplify applies simple constant elemination, trigonometric identities.

If you read the previous section on the rules DSL, you should be able to read and understand the [rules](https://github.com/JuliaSymbolics/SymbolicUtils.jl/blob/master/src/simplify_rules.jl) that are used by `simplify`.

## Code generation

**Experimental feature**

It is common to want to generate executable code from symbolic expressions and blocks of them. We are working on experimental support for turning Symbolic expressions into executable functions with specific focus on array input and output and performance which is critical to the Differential Equations ecosystem which is making heavy use of this package.

See [Code generation](/codegen/) for more about this.

## Learn more

If you have a package that you would like to utilize rule-based rewriting in, look at the suggestions in the [Interfacing](/interface/) section to find out how you can do that without any fundamental changes to your package. Look at the [API documentation](/api/) for docstrings about specific functions or macros.

Head over to the github repository to ask questions and [report problems](https://github.com/JuliaSymbolics/SymbolicUtils.jl)! Join the [Zulip stream](https://julialang.zulipchat.com/#narrow/stream/236639-symbolic-programming) to chat!

