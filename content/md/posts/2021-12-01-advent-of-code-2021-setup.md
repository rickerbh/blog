{:layout :post, :title "Advent of Code 2021 - Puzzle 1", :date "2021-12-01" :draft? false}

One of the issues with getting started with a new language in a time-pressured scenario is setting up environment and tooling. Last year I used VSCode and the [ReasonML plugin](https://github.com/reasonml-editor/vscode-reasonml). It installed relatively straight-forwardly (from what I remember), and supported type hints, and autocompletion. This year I decided I'd setup Emacs, given I use it for everything else. Long story short - what an utter shambles. I tried to follow the [installation guide](https://dev.realworldocaml.org/install.html) for the Real World OCaml book plus the Prelude OCaml support (I use Prelude with Emacs, due to the great Clojure support it has). It was generally a struggle to get the extensions to handle resolving imports of local files, as well as 3rd party libraries imported via opam. I gave up and went back to VSCode. The plugin setup was straightforward, but I still struggle with being able to actually run a program. I use the utop repl(like) interface to evaluate the statements in my program files, but even that wasn't that simple. Needing to deal with merlin and dune config files just to get my files recognised feels like a bunch of unnecessary complexity. I need to resolve this (and I'd still love to get it all working in emacs), but I can limp along with what I've currently got.

Anyway, puzzle 1, step 1 - my solution for this was to run a reducing function over the array, incrementing a counter when a new value is greater than the prior value. It required 2 functions to be created to execute and produce the answer. The input needs to be a list of ints as well.

``` ocaml
(* This function receives a tuple that consists of the prior number, and the
   count of "increments" that have been observed, as well a a new number for
   comparison. The match statement compares the prior with the new, and if the
   new is greater, it will return a tuple of the new and increment the count,
   otherwise it returns a tuple of the new and the old increment count. *)

let increment_counter (prior, inc_count) x =
  match x with _ when x > prior -> (x, inc_count + 1) | _ -> (x, inc_count)


(* This just performs a left fold oven the input list, applying the increment
   counter function to each input value. The result tuple is destructured and
   the increment count is returned *)

let problem_1 input =
  let _, result = List.fold_left increment_counter (List.hd input, 0) input in
  result
```
