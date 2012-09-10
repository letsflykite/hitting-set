# hitting-set

A Clojure library to find minimal hitting sets and set covers.

## What’s this?

This library deals with two closely related problems in graph theory: the hitting set problem and the set cover problem. These are explained pretty well [by Wikipedia](https://secure.wikimedia.org/wikipedia/en/wiki/Set_cover_problem). To summarize, if we have a hypergraph *H* consisting of a set of vertices and a set of edges, then

* a *hitting set* of *H* is a set of vertices that has a nonempty intersection with each edge, and
* a *set cover* of *H* is a set of edges which, taken together, contain all of the vertices of *H*.

Usually people are interested in the hitting set or set cover of minimum possible cardinality: the *minimal hitting set* or *minimal set cover*.

---

Before we go into the relationship between these two problems let’s talk about how to express a hypergraph in Clojure. For this library we’ll use a hash map where the keys are the edge names and the values are sets containing the vertex names. Suppose we construct a hypergraph where the vertices are colors and the edges are national flags—a color vertex is a member of a flag edge if and only if that color is used on that flag. Here’s a truncated example:

```clj
{"Australia" #{:white :red :blue},
 "Tanzania" #{:black :blue :green :yellow},
 "Norway" #{:white :red :blue},
 "Uruguay" #{:white :blue :yellow},
 "Saint Vincent and the Grenadines" #{:blue :green :yellow},
 "Ivory Coast" #{:white :orange :green},
 "Sierra Leone" #{:white :blue :green},
 "United States" #{:white :red :blue}}
```

Here I have used strings for the keys and keywords for the elements of the sets, but you can use strings, keywords, or pretty much any other non-sequence data type for the edges or for the vertices.

---

Hopefully this example has made our hypergraphs a little easier to visualize. Now let’s talk about *inverting* hypergraphs. By this I mean that instead of mapping edge names to sets of vertex names, we’ll map vertex names to the set of names of the edges in which those vertices are contained:

```clj
{:black #{"Tanzania"},
 :white #{"Australia" "Norway" "Uruguay" "Ivory Coast"
   "Sierra Leone" "United States"},
 :orange #{"Ivory Coast"},
 :red #{"Australia" "Norway" "United States"},
 :blue #{"Australia" "Tanzania" "Norway" "Uruguay" "United States"
   "Saint Vincent and the Grenadines" "Sierra Leone"},
 :green #{"Tanzania" "Saint Vincent and the Grenadines"
   "Ivory Coast" "Sierra Leone"},
 :yellow #{"Tanzania" "Uruguay" "Saint Vincent and the Grenadines"}}
```

Note that this is still a perfectly valid hypergraph! The relationship between the hitting set and the set cover is that the hitting set of the first hypergraph above is a set cover for the second set, and vice versa. Inverting a hypergraph is easy and fast, so depending upon the situation, finding the hitting set may be easiest by inverting the hypergraph and then finding the set cover.

## Usage

The `project.clj` specifies a minimum Leiningen version of 2.0.0, although there probably isn’t anything in this library that actually requires that version.

1. Add `[hitting-set "0.9.0"]` to the `:dependencies` vector in your `project.clj`.
2. `(:use hitting-set :only [minimal-hitting-sets])`, or whichever of the functions below you need.

Following are descriptions of the “end-user” functions in the library.

### Utility functions

* `reverse-map [h]`

    “Inverts” the hypergraph `h` in the sense shown in the example above.

### Hitting set

* `hitting-set? [h s]`

    Returns true if `s` is a hitting set for `h` (i.e. if `s` has a nonempty intersection with each edge in `h`). This function does not check to ensure that `s` is of minimal size.

* `hitting-set-exists? [h k]`

    Returns true if a hitting set of size `k` exists for the hypergraph `h`. See the caveat below.

* `enumerate-hitting-sets [h] [h k]`

    Returns a set containing the minimal hitting sets of `h` and possibly (but not necessarily! see the caveat below) some *non*-minimal hitting sets. If `k` is passed then only sets of size `k` or smaller will be returned; if `k` is not included then sets will be returned without regard to their size.

* `minimal-hitting-sets [h]`

    Returns a set containing the minimal hitting sets of `h`. For example, if the minimal possible hitting set has size two and there are three hitting sets of size two, all three will be returned. If you want just one hitting set and don’t care about uniqueness, use `(first (minimal-hitting-sets h))`.

* `approx-hitting-set [h]`

    Returns a hitting set of `h` generated by inverting `h` and using the greedy cover algorithm on it. The returned set is guaranteed to be a hitting set, but it is not guaranteed to be minimal.

### Set cover

* `cover? [h s]`

    Returns true if the set `s` is a cover for `h`.

* `greedy-cover [h]`

    Uses the “greedy” algorithm (described [at Wikipedia](http://en.wikipedia.org/wiki/Set_cover_problem#Greedy_algorithm)) to generate a set cover for `h`.

## Caveats

* Due to limitations in the algorithms in this library, you may experience the following oddities:

  1. `hitting-set-exists?` will correctly return `false` if you pass it a size `k` below that of the minimal hitting set, and it will correctly return `true` when you pass it a `k` that is equal to the size of the minimal hitting set. However, it may return `false` for larger values of `k`, even when the values are less than the number of vertices in the hypergraph. (Recall that if the minimal hitting set has size *a* and there are a total of *c* vertices in the hypergraph, then we can form a hitting set of any size *k* between *a* and *c*, inclusive, by adding elements to the minimal hitting set. The resulting set will still have the hitting set property that it has a nonempty intersection with each hyperedge, and so it will still be a hitting set.)

  2. `enumerate-hitting-sets` has a similar problem: It will always (correctly) return all possible *minimal* hitting sets, but it may not return hitting sets that are larger than minimal.

  I intend to fix both of these issues in a future version of the library. Note that if you’re only looking for *minimal* hitting sets then neither of these oddities is an issue.

* The functions `hitting-set-exists?` and `enumerate-hitting-sets` (and by extension `minimal-hitting-sets`) are recursive but do not make use of Clojure’s `recur` and friends. This opens the possibility of a stack overflow. In practice, though, the recursion goes no deeper than `k` levels, where `k` is the maximum edge size. Since the hitting set problem is NP-complete, the computation for small edge sizes will take so long that you’ll give up (or run out of heap) long before the stack fills up.

## Further reading

* [An article (containing a hitting-set example) by the author of this library](http://www.bdesham.info/2012/09/olympic-colors)
* [The first chapter of *Parameterized Complexity Theory* by J. Flum and M. Grohe](http://www2.informatik.hu-berlin.de/~grohe/pub/pkbuch-chap1.pdf), which contains a discussion of the hitting set (among other problems) and which provided the algorithms which were adapted for this library

## History

Version numbers are assigned to this project according to version 1.0.0 of the [Semantic Versioning](http://semver.org/) specification.

* Version 0.9.0 (2012-09-09)
  - Added support for the set-cover formulation of the problem via `cover?` and `greedy-cover`
  - Used the new set-cover functions to implement `approx-hitting-set` to find approximate hitting sets
  - Modified `hitting-set?` so that the hypergraph is the first argument to every function in the library
* Version 0.8.0 (2012-09-05): Initial release.

## License

Copyright © 2012 Benjamin D. Esham (www.bdesham.info).

This project is distributed under the Eclipse Public License, the same as that used by Clojure. A copy of the license is included as “epl-v10.html” in this distribution.
