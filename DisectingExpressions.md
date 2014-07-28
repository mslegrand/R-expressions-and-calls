
Dissecting R Calls and Expressions
========================================================
left: 80%
author: S Legrand
date: 2014.07.30


<div style='text-align: bottom;'>
    <img height='160' src='figures/useR.png' />
</div>

The Parse and Evaluate Idiom
===
To evaluate a character string as R-code, the usual thing to do
is to use ***eval(parse(text=myString))***. For example

```r
eval(parse(text="1+2"))
```

```
[1] 3
```

But this begs the question, what does ***parse*** return?

Parse Returns Expressions
===
Parse returns an object called an **expression**

```r
ex1<-parse(text="1+2")
mode(ex1)
```

```
[1] "expression"
```


Two Ways to Create Expressions
===


```r
#directly
 expression(1+2)
```

```
expression(1 + 2)
```
***

```r
#using parse
 parse(text="1+2")
```

```
expression(1 + 2)
```


Disecting Expressions
===
We examine the components of expressions using the [[]] operator

```r
 ex1<-expression(1+2)
 ex1[[1]]
```

```
1 + 2
```

```r
 mode(ex1[[1]])
```

```
[1] "call"
```

Expressions are special Lists
===

```r
cl<-call("+",1,2)
el<-list(cl)
mode(el)<-"expression"
identical(el, expression(1+2))
```

```
[1] TRUE
```
So an **expression** is a specialized list with mode ***expression*** 

The Expression Building Blocks
===
The building blocks of an **expression** can be any of the following:
- calls
- symbols 
- constants

Some of Simple Expressions
===

```r
 ex1<-expression(1+2)
 mode(ex1[[1]])
```

```
[1] "call"
```

```r
 ex2<-expression(x)
 mode(ex2[[1]])
```

```
[1] "name"
```

```r
 ex3<-expression(3)
 mode(ex3[[1]])
```

```
[1] "numeric"
```

Some checks for the Mode
===

```r
  c(is.expression(expression(1+2)),
  is.call(expression(1+2)[[1]]),
  is.symbol(expression(x)[[1]]),
  is.numeric(expression(3)[[1]]))
```

```
[1] TRUE TRUE TRUE TRUE
```

Expressions with 2 Calls
===
**Expressions** can contain multiple components.
For example the following **expression** contain 2 **calls**

```r
ex1<-expression(x<-1,x+2)
ex1
```

```
expression(x <- 1, x + 2)
```

```r
ex2<-parse(text="x<-1;x+2")
ex2
```

```
expression(x <- 1, x + 2)
```


Different Ways to Create a Call
===

```r
parse(text="1+2")[[1]]
```

```
1 + 2
```

```r
call("+",1,2)
```

```
1 + 2
```

```r
substitute(1+2)
```

```
1 + 2
```
***

```r
as.call(list(as.name("+"),1,2))
```

```
1 + 2
```

```r
quote(1+2)
```

```
1 + 2
```

```r
fn<-function(){1+2}
body(fn)[[2]]
```

```
1 + 2
```

Calls are Special Lists
===

```r
list(as.name("+"),1,2)->cl
mode(cl)<-"call"
identical(cl, call("+",1,2))
```

```
[1] TRUE
```
**Calls** are specialized lists with mode ***call***

Evaluating Calls
===
**Calls** can also be evaluated using **eval**

```r
cl1<-parse(text="1+2")[[1]]
eval(cl1)
```

```
[1] 3
```

Dissecting a Call
===
We dissect a call by examinin the components of **call** using the [[]] operator

```r
cl1[[1]]
```

```
`+`
```

```r
cl1[[2]]
```

```
[1] 1
```

```r
cl1[[3]]
```

```
[1] 2
```

Calls can even be Modified!
===

```r
cl1<-quote(1+2)
cl1
```

```
1 + 2
```

```r
fn<-function(x,y){
  1+x^y
}
```
***

```r
cl1[[1]]<-as.name("fn")
cl1 # Replaced + with fn
```

```
fn(1, 2)
```

```r
eval(cl1) 
```

```
[1] 2
```

A slightly more complex example
=====


```r
cl<-quote(1*2+3)
cl
```

```
1 * 2 + 3
```

Note: we now have both * and + inside our **call**

Drilling down Reveals a Tree
===

```r
cl[[1]]
```

```
`+`
```

```r
cl[[2]]
```

```
1 * 2
```
***

```r
cl[[2]][[1]]
```

```
`*`
```

```r
cl[[2]][[2]]
```

```
[1] 1
```

```r
cl[[2]][[3]]
```

```
[1] 2
```

A Call is a Tree
===
- A **Call** is a tree consisting of **terminal** and **non-terminal** nodes.
- A **non-terminal node** is list consisting of a name (usually a function)
followed by it's children.
+ Ex: call('+',1,2)

- A **terminal node** is a value or the name of a function with no args.

- We call this  tree an **abstract syntax tree (AST)**


The Tree Structure
===
- cl is the root node: Non-terminal =(+, 1 * 2, 3)
- cl[[1]] is the root node label: Label   = +
- cl[[2]] is the first child: Non-terminal  = (*, 1, 2)
- cl[[3]] is the second child: = 3
- cl[[2]][[1]] is the label of the first child node: Label  = *
- cl[[2]][[2]] is the first child of [[1]][[2]]: Terminal  = 1
- cl[[3]][[2]] is the second child of [[1]][[2]]: Terminal = 2 

The Tree Structure of the Call cl
===

Accessor | Position | Role | Description
---------|---------|---------|---------
cl | root | Non-terminal | (+, 1 * 2, 3)
cl[[1]] |root label|  Label | +
cl[[2]] | 1st child of root| Non-terminal | (*, 1, 2)
cl[[3]] | 2nd child of root| Terminal | 3
cl[[2]][[1]] |  root 1st child label |  Label  | *
cl[[2]][[2]] | 1st child of cl[[1]][[2]]| Terminal  | 1
cl[[3]][[2]] | 2nd child of cl[[1]][[2]]| Terminal | 2 

Displaying the tree
===
The package ***pryr*** makes it a little easier to see this structure

```r
library(pryr)
call_tree(cl)
```

```
\- ()
  \- `+
  \- ()
    \- `*
    \-  1
    \-  2
  \-  3 
```

Graphicaly
===
![plot of chunk unnamed-chunk-21](DisectingExpressions-figure/unnamed-chunk-21.png) 
***
Here we used the package ***diagram*** to do our rendering :)

Manipulating a Call Tree
===
We can even manipulate to rearrange precedence!

```r
cl.orig<-parse(text="1*2+3")[[1]]
cl.orig
```

```
1 * 2 + 3
```

```r
tmp<-cl.orig
cl.mod<-cl.orig[[2]]
cl.mod[[3]]->tmp[[2]]
cl.mod[[3]]<-tmp
cl.mod
```

```
1 * (2 + 3)
```

The Original vs Modified Tree
===

```r
call_tree(cl.orig) 
```

```
\- ()
  \- `+
  \- ()
    \- `*
    \-  1
    \-  2
  \-  3 
```
***

```r
call_tree(cl.mod)
```

```
\- ()
  \- `*
  \-  1
  \- ()
    \- `+
    \-  2
    \-  3 
```

The Body of a Function is a Call
===

```r
fn<-function(){1+2}
cl<-body(fn)
mode(cl)
```

```
[1] "call"
```
***

```r
call_tree(cl)
```

```
\- ()
  \- `{
  \- ()
    \- `+
    \-  1
    \-  2 
```

Calls may contain Calls
===

```r
cl<-body(
  function(){
  1+2
  }
)
mode(cl)
```

```
[1] "call"
```

```r
length(cl)
```

```
[1] 2
```
***

```r
mode(cl[[2]])
```

```
[1] "call"
```

```r
cl[[2]]
```

```
1 + 2
```
Here, the call cl, contains
 the call cl[[2]]

The body of a 2 line function
===

```r
fn<-function(x){
  y<-x+1
  2*y
}
cl<-body(fn)
length(cl)
```

```
[1] 3
```
***

```r
call_tree(cl)
```

```
\- ()
  \- `{
  \- ()
    \- `<-
    \- `y
    \- ()
      \- `+
      \- `x
      \-  1
  \- ()
    \- `*
    \-  2
    \- `y 
```

Piece-wise Function Construction
===
We can construct a new function in steps:

1. Start with an empty skeleton function 
2. Insert a body
3. Specify the arguments
4. Specify an environment


A Skeleton Function 
===

```r
fn<-function(){}
fn
```

```
function(){}
```


Inserting a Body
===


```r
body(fn)<-call("{",quote(y<-x+1),quote(2*y) )
fn
```

```
function () 
{
    y <- x + 1
    2 * y
}
```
The function body now contains 2 lines.

Specifying  Arguments
===

```r
formals(fn)<-alist(x=)
fn
```

```
function (x) 
{
    y <- x + 1
    2 * y
}
```
The function now has an argument ***x***. If we wanted a default value of ***x=2***,
we would have specified ***alist(x=)***

Specifying am Environment
===

```r
environment(fn) <- .GlobalEnv
environment(fn)
```

```
<environment: R_GlobalEnv>
```
Here we set the environment of the function to be the Global environment

Creating Functions On the Fly
===
When we don't know what ot how many statements the function may contain, then 
to create the body we need to generate
a ***list of calls***  and this is when **expressions** become useful

To illustrate this, we take an example motivated by continued fractions.
Given n, we want to generate a function that contains n copies of the 
line 
```
x<-1/(1+x)
```

We illustrate 2 approaches:

Calls to Expression to Body
===

```r
n<-4
fn1<-function(x){} #A skeleton
cl<-quote(x<-1/(1+x))
ex<-c(rep(list(cl),4),quote(x))   
mode(ex)<-"expression"
body(fn1)<-as.call(c(as.name("{"),ex))
fn1
```

```
function (x) 
{
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x
}
```

Strings to Expression to Body
===

```r
n<-4
fn2<-function(x){} #A skeleton
cl<-"x<-1/(1+x)"
ch<-paste(c(rep(list(cl),4),"x"),collapse=";")   
ex<-parse(text=ch)
body(fn2)<-as.call(c(as.name("{"),ex))
fn2
```

```
function (x) 
{
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x <- 1/(1 + x)
    x
}
```
Comparing and Running fn1, fn2
===

```r
identical(fn1,fn2)
```

```
[1] TRUE
```

```r
fn1(2)
```

```
[1] 0.6364
```

```r
fn2(2)
```

```
[1] 0.6364
```

Summary of Expressions
===
- **Expressions** are return by **parse** and wrap **calls** in a list.
- Use **expressions** when a list of **calls** is required

Summary of Calls
===

- **Calls** and **Expressions** are specialized lists
-  **Calls** are trees,  **Abstract Syntax Trees** coded using lists
- A node on the **AST** is either a call-list or a value
+ The first element of a **call** list is  name (naming a functions)
+ Any subsequent elements occuring in a **call** list are children
- **calls** occur in  **function bodies**, are returned by **substitute**, etc.
- **calls** can be manipulated and evaluated using **eval**

