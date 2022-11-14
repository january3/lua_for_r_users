# About this document

I wrote it mostly as a reminder to myself, should I ever feel the need to
code something in lua.

# Where to start

You don't need much.

 * [Learn lua in 15 minutes](https://learnxinyminutes.com/docs/lua/)
 * [Programming in lua](https://www.lua.org/pil/contents.html) if you are
   feeling fancy
 * [lua reference manual](https://www.lua.org/manual/) if you are a
   software engineer or some other kind of deviant

# Similar stuff

## Same thing, different name

Loads of stuff is really the same.

 * The `.` in R is just another variable
    name, and so is `_` in lua.
 * `dofile` is basically the same as `source`.
 * Instead of curly braces, you have use keywords: `function ... end`,
    `do ... end`, `if ... then ... elseif ... else ... end`.
 * You define tables
    (which are quite like lists in R, but see below) using curly braces and not
    square brackets, and square brackets are used for indices.
 * Indices of tables start, by default, at 1.
 * comments start with `--`, and not `#`. There are also multiline comments
   which start with `--[[` and end with `--]]`
 * you can put everything on one line, no semicolons necessary. `a = 1 b = 2` is perfectly fine.
 * `return` must be the last statement in a block (so last before `end`,
   `else`, `elseif`)

## Curly brackets and parentheses have a different meaning and I confuse them a lot

When programming lua and R both, you use curly brackets a lot. In lua,
however, they are equivalent to `list()` in R. When switching between R and
lua, you will find yourself writing, in R, a lot of constructions like `a
<- { 1, 2 }` where in fact you meant `a <- list(1, 2)` or `a <- c(1, 2)`.
Also `if(a < 2) then`, this is my favorite. In lua, `a <- 1`.

## You can use ellipsis for variable number of arguments

You can use `...` when defining a function in lua, just like in R. In the
body of function you have then to use `...` and put its return value into
a table,  which now holds
all the arguments, see the code below. You can also combine the ellipsis with regular
arguments:

    function foo(a, b, ...)

      print("a=", a)
      print("b=", b)
      for k, v in ipairs({...}) do
        print(k, v)
      end
    end

And we can pass the unspecified number of arguments to the next function
like this:

  function foo2(...)
    foo(table.unpack({...}))
  end

However, one big caveat: the above syntax changed between versions of lua.
Not sure exactly when. Basically, in older lua you use `arg` instead of
`{...}` and `unpack` instead of `table.unpack`.


## Functions are values

Just like in R, functions are values. Although the standard way of defining
a function is writing something like `function myfunc() ... end`, you can
also omit the function name and assign the result to a variable or used it
as a closure somewhere

  myfunc = function() ... end

You can pass functions as arguments to other functions, you can create
closures etc. etc.

# Really different stuff

## Everything is passed by reference

R has this neat feature that when you pass an object – whatever it is – to
a function, it is not passed by reference. That is, if you modify the
object in a function, then the object will be copied, and the copy will be
modified. That way you avoid any side effects of the function. For example,
this piece of code in R

    ff <- function(x) { x[1] = 0 }
    a <- c(2, 3, 4)
    ff(a)
    a

produces `[1] 2 3 4` – the original `a` value remains unchanged.

In many other programming languages, for example Python, and also in lua, this is not the
case. If you pass an array to a function and then the function modifies it,
then the array will stay modified. Consider this piece of lua code:

    function ff(x) x[1] = 0 end
    a = { 2, 3, 4 }
    ff(a)
    a[1]

produces 0, and not 2. That means, function ff modified the original array
a.

## Iterators

Don't let me start on iterators.


## Interface sucks

In R, we are used to work interactively. Personally, I consider interactive
work important when learning a language in almost any language. When
learning lua, I use interactive shell a lot.

Interactive shell of lua is a sad, sad thing. No tab completion, no `?` or
`??`. I think the idea is that you learn lua core by heart and google the
rest. This makes learning a process of switching between a browser and
terminal / editor.

Of course, you can use VS Code and have all the goodies.

## There is no vectorization (by default)

There is no lapply or anything similar in the default library. Writing
lapply is actually trivial

    function lapply(x, fun)

      res = { }
      for k, v in pairs(x) do
        res[k] = fun(v)
      end

      return res
    end

    foo = lapply({ 1, 2, 3, 4 }, function(x) return x/2 end)


There are several libraries out there that can give you more of
vectorizations, including the ability to simply divide an array by a number.



# Nice things big and small

## OO programming

OO in R is, let us not be shy about that, a despicable mess as anyone knows
who ever tried to use tidyverse in combination with BioConductor. OO in lua
is just nice. I'm not going to describe it in all detail, the "Programming
in lua" book gives a very nice introduction. Basically, an object is a
table in which some elements are functions. As simple as that, and you can
do it in R easily with lists (in fact, it has been done... multiple
times...), but there is just a sprinkle of syntactic sugar on top of it
which makes it really nice to work with: if you use `:` rather than `.`
when referencing a table element which is a function (i.e., an object
method), the first argument is the table itself. You can also use it in the
function (method) definition.

This is super simple, and yet powerful: you have methods, you have
inheritance, you can even have overloading, prototyping and private methods and fields.
However, you need to understand metatables (see below), which are weird.

## Multiple assignments, returning multiple arguments and swapping values

lua functions can return multiple arguments, so that can make things rather
simple:

    a, b = 4, 5

    function ff() return 1, 2 end

    a, b = ff()
    a, b = b, a

This last line shows that you can swap variables in one line. However watch
out. Also, you can assign multiple values to one value, the rest of the
values will then be discarded:

    a = ff() -- a is now 1

And you can put the results directly in an array:

    a = { ff() } -- a is now { 1, 2 }

## x = a or "default" ; x = a or b or 0 ; x = a > 0 and 0 or a

There is one important difference regarding logical operators between lua
and R. In lua, there are only two values which are interpreted as false:
`false` and `nil`. Everything else, including 0 and empty string is true.

Moreover, logical operators return the last value evaluated, and not `true`
or `false`. That is, the value of `false or "foo"` is `"foo"`. This makes
it very practical, for example to set a default value:

  a = arg1 or arg2 or "default"

There is also an idiom that corresponds to the `a ? b : c` idiom of some
other languages:

  max = a < max and max or a

This is a bit harder to understand, but it works like this: if a is smaller
than max, then `a < max and max` is true, and its value is `max`. Lua no
longer inspects `a`, so the value of `max` gets assigned to `max` and `max`
does not change. If, however, a is greater than max, then `a < max and max`
is false, so lua continues to evaluate a; since a is a value, it is true,
and so the result of the whole expression is the value of a.

## Comma at the end of an array

Defining `a <- c(1, 2, )` throws an error in R. Defining `a = { 1, 2, }` in
lua is perfectly normal. That makes copying and pasting blocks of code
so much easier.

## table.unpack

The table.unpack() function makes handling argments really easy. Basically, it
expands an array. So if a function takes three arguments, and you have the
three arguments stored in an array, then you can simply do as follows

    args = { 1, 2, 3 }
    function ff(a, b, c)
      print(a, b, c)
    end
    ff(table.unpack(args))

# Ugly, ugly stuff

## Package management

Only if you tried to install anything with
[luarocks](https://luarocks.org/) you can truly appreciate the work that
CRAN is doing. I thought before that `pip` sucks compared to
`install.packages()`. I was wrong. Pip merely vacuums. 

Installing lua modules is awful. It is a mess. A horrible, horrible mess.
Nothing is consistent, there are no dependencies, basically no
documentation and installing even the popular things is a major pain. When
it comes to documentation, again, everybody seems to be on their own: no
standards, no templates, no consistency. When poeple criticize lua for
using `1` as starting index of an array, they have no clue; however, module
management under lua is for me one of the top reasons not to use lua. Perl
had a better package management twenty years ago.

# Really weird stuff that takes getting used to

## Tables

The good news is, after a while you not only get used to tables in lua, but actually
start liking it. The bad news is, it may be a minefield, especially at the
beginning.

In lua, virtually every complex object type is a table. Tables function
both as R lists and R vectors. You can use them as vectors (and then lua
people tend to call them "arrays"), with numerical
indices, but you can also add string indices. You can mix them. You can
have non-contiguous tables with numerical indices. It is all valid.

Similar like it is with lists in R, assigning the nil value (similar to
NULL in R) to a table element removes it.

Tables are full of traps for the R user.

### Trap number one: table length

If you prepend a hash to a table, you will get the number of elements in a
table... provided the table has a numerical index. For
example, the following code will return `4`, and not `5` as might be
expected:

    a = { 10, 20, 30, 40 }
    a["foo"] = 50
    #a

So you have to somehow make sure that you use `#a` only when a has a
numerical index (or, in lua-speak, is an array).

The easiest way to ensure this is to exclusively use the `table` standard
library to manipulate tables which should work as "arrays" (and "array" is
what we call a "vector" in R). Use `table.insert` to insert an element,
`table.remove` to remove an element, `table.getn` to get the length of an
array and `table.setn` to set the length of an array.

### Trap number two: string indices

This is similar to a certain trap with R rownames. In lua, you can have
indices which are strings. Of course, that means that in the same array you
can have an index which is 2 (the number) and another one which is "2" (a
string).

    a = { }
    a["2"] = 10
    a[2] = 20

Oh, this may be so much fun.

### Trap number three: pairs and ipairs

One of the most frequently used constructions in lua is a for ... do loop
using an iterator. There are two standard iterators commonly used with
tables, and they are different. The `pairs` iterator goes over all elements
of a table, in unspecified order. The `ipairs` iterator starts with index 1
(numerical) and increases it by one until it finds the first nil element.
The following prints only two values:

  a = { 1, 2, nil, 4 }
  for k, v in ipairs(a) do
    print(v)
  end

## About these multiple return values...

Here are traps, too. First, putting a parentheses around a function call
which returns multiple values results in one value only. These two
print statements are different:

  function foo() return 1, 2 end

  print(foo())
  print((foo()))

Same goes for putting parentheses around the return statement. The foo1
returns only one value despite looking similar to foo:

  function foo1() return(1, 2) end

## metatables

So OK, this is weird as heck. Basically, you can set a table as a metatable
for another table: `setmetatable(t1, t2)`. Then, the elements of `t2` will
be used in certain situations when `t1` is processed. For example, you can
create an element with key `__add` - it should be a function. This element
then will be used if you attempt adding two tables with the same
metatables. Another one is `__index`, which is called when an element of
`t1` is nil:

    > t1 = { a = 1 }
    > setmetatable(t2, t1)
    table: 0x55d946eed700
    > t1.__index = function(table, key) return "default" end
    > t2[1]
    default
    >

Alternatively, `__index` can be an array with default values:

    t1.__index = { a = "default" }
    t2["a"]   --> returns default


## Named parameters: function{a = 1} .... excuse me, WHAT?

Lua has some unexpected syntactic sugar here and there, and also stuff that
looks like quick hacks to achieve certain functionality. Here we are
talking about named arguments.

So, lua does not have named parameters. One way of dealing with this sad
fact is to use only one parameter, which is an array containing all the
(named now) parameters. In fact, we can even mix named an positional
parameters and make some of them optional. However, we need to do some
footwork:

    function stranger(t)

      a = t[1] or t.a or error("a is required")
      b = t[2] or t.b or "default"
      print(a, b)

    end

We can then call `stranger({1, 2})` (no names), `stranger({1, b=2})` or
`stranger({b=2, a=1})`. This is so common that lua authors springled some
syntactic sugar on top of it and you can omit the parentheses:

    stranger{1,2}
    stranger{1, b=2}
    stranger{b=2, a=1}

Looks almost like a real function call, doesn't it? :-)

# Typical errors

    attempt to perform arithmetic on a table value                                                                                                                                       

Usually means you forgot you are in lua and did `a <- 1`.

    attempt to call a table value

Probably wrote `for k, v in tab do` instead of `for k, v in pairs(tab) do`.

    attempt to index a nil value

You did `tab.val` but tab is `nil`.


