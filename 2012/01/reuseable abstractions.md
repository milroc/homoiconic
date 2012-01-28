# Reusable Abstractions in CoffeeScript and JavaScript

[David Nolan][swan] made a really interesting observation:

[swan]: https://github.com/swannodette

> Wow. HashMaps in ClojureScript are functions! Now this may look like some special case provided by the language but that's not true. ClojureScript eats its own dog food - the language is defined on top of reusable abstractions. [Comparing JavaScript, CoffeeScript & ClojureScript](http://dosync.posterous.com/comparing-javascript-coffeescript-clojurescri)

HashMaps in ClojureScript are functions. That's a really powerful idea. Why? Well, let's think about it. Right now, in CoffeeScript (and JavaScript), we have two things that behave in almost the same way but have different syntax:

```coffeescript
a = b[c] # b is an array or an object

e = f(g) # f is a function
```

There is a good thing about this: The two different syntaxes signal to the reader whether we are dealing with arrays or functions.

There is a bad thing about this: None of the tools we develop for functions work with the syntax for arrays.

For example, we can *compose* any two functions with Underscore's `compose` function:

```coffeescript
f = (x) -> x + 1
g = (x) -> x * 2

h = _.compose(f, g)
  # h(3) === f(g(3)) === 7
```

But we can't compose a function with an array reference or two array references:

```coffeescript
a = [2, 3, 5, 7, 11, 13, 17, 19]
b = (x) -> x * 2

h = _.compose(a, b)
  # => TypeError: Object 2,3,5,7,11,13,17,19 has no method 'apply'
       at /Users/raganwald/Dropbox/development/cafeaulife/node_modules/underscore/underscore.js:593:26
```

And this is just the beginning. You can `.map` over an array, but you can't pass an array to `.map` as an argument. Same for standard objects (a/k/a "hashes"). We can go though our toolbox, and find hundreds of places where we have a special tool for functions that we can't use on arrays or objects. How annoying. I suppose we could "monkey-patch" `Array` to support `.apply` and `.call` to get around some of these errors, but the cure would be worse than the disease.

### Why CoffeeScript is an Acceptable ClojureScript

There are tremendous benefits to a language making these two things equivalent "all the way down." But for many practical purposes, we can reap the benefits of having arrays and objects be first-class functions by wrapping arrays and objects in a function. Here's one such implementation:

```coffeescript
dfunc = (dictionary) ->
  (indices...) ->
    _.reduce indices, (a, i) ->
      a[i]
    , dictionary
```

`dfunc` takes an array or object you want to use as a "dictionary" and turns it into a function. So you can write:

```coffeescript
address = dfunc
  street: "1010 Foo Ave."
  apt: "11111111"
  city: "Bit City"
  zip: "00000000"
```

And now you have a function, just like one of ClojureScript's use cases. `dfunc` is also useful for encapsulating the choice of using an object or array lookup for implementing a  function. In [Cafe au Life][cafe], rules for [life-like games][ll] are represented as an array of arrays of neighbour counts. For example, the rules for Conway's Game of Life are represented as:

```coffeescript
[
  [0, 0, 0, 1, 0, 0, 0, 0, 0, 0] # A cell in state 0 changes to state 1 if it has exactly 3 neighbours
  [0, 0, 1, 1, 0, 0, 0, 0, 0, 0] # A cell in state 1 changes to state 0 unless it has 2 or 3 neighbours
]
```

And the rules for the life-like game [Maze][maze] are represented as:

```coffeescript
[
  [0, 0, 0, 1, 0, 0, 0, 0, 0] # A cell in state 0 changes to state 1 if it has exactly 3 neighbours
  [0, 0, 1, 1, 0, 0, 0, 0, 0] # A cell in state 1 changes to state 0 unless it has 1 to 5 neighbours
]
```

Naturally, the code for actually *processing* the rules could use `[]` to look things up. But why should it know how the rules are represented internally? Instead, we wrap the rule array with `dfunc`,making a `rule` function out of an array representation of the rules, and from that, make a `succ` or "successor" function that computes the success for for any cell in a matrix of cells:

```coffeescript
rule = dfunc [
  # ... rules for the current game ...
]

succ = (cells, row, col) ->
  neighbour_count = cells[row-1][col-1] + cells[row-1][col] +
    cells[row-1][col+1] + cells[row][col-1] +
    cells[row][col+1] + cells[row+1][col-1] +
    cells[row+1][col] + cells[row+1][col+1]
  rule(cells[row][col], neighbour_count)
```

Of course, `succ` could be written to depend on the array implementation of the rules. But turning it into a function factors it cleanly. We can change `rule` and `succ` independently, which is what we expect from *encapsulating* the array in a function.

### Reusable Abstractions

Within a single function, it's good CoffeeScript and JavaScript to implement certain things with arrays and objects as dictionaries. Naturally! But when *exposing* properties as part of an API, functions are preferred, because functions are more easily reused abstractions and they preserve a read-only contract. JavaScript and CoffeeScript don't actually implement arrays and "hashes" as functions, but it's easy to wrap them in a function and obtain many of the benefits.

[cafe]: https://github.com/raganwald/cafeaulife
[ll]: http://www.conwaylife.com/wiki/Cellular_automaton#Well-known_Life-like_cellular_automata

---

Recent work:

* [Kestrels, Quirky Birds, and Hopeless Egocentricity](http://leanpub.com/combinators), all of my writing about combinators, collected into one e-book.
* [What I've Learned From Failure](http://leanpub.com/shippingsoftware), my very best essays about getting software from ideas to shipping products, collected into one e-book.
* [Katy](http://github.com/raganwald/Katy), a library for writing fluent CoffeeScript and JavaScript using combinators.
* [YouAreDaChef](http://github.com/raganwald/YouAreDaChef), a library for writing method combinations for CoffeeScript and JavaScript projects.

---

[Reg Braithwaite](http://reginald.braythwayt.com) | [@raganwald](http://twitter.com/raganwald)