{:layout :post, :title "Advent of Code 2021 - Puzzle 6", :date "2021-12-06" :draft? false :tags ["aoc" "aoc2021" "ocaml"]}

## Puzzle 6 - Step 1

For todays puzzle we're modelling the lifecycle of fish. The short version is that a fish's timer goes from 8->0, and after 0 it regenerates itself as 6, and spawns a new 8, increasing the count of fish by 1. In this world, fish are represented as an integer with the value of the int being their timer. To advance a school of fish, we reduce over each of the fish, and when we see one that's a 0, we regenerate it as a 6 and add an 8.

``` ocaml
let advance school =
  List.fold_left
    (fun acc f -> match f with 0 -> 8 :: 6 :: acc | _ -> pred f :: acc)
    [] school
```

The solution of the puzzle is figuring out how many fish there are on a given day in the future. To do this, we need to run advance once for each day the fish are alive. I grabbed a bit of code from SO for this.  This will just recursively apply the function `f` `i` times, with input `acc`. And we can wrap it with something more domain based.

``` ocaml
let rec foldi i f acc = if i <= 0 then acc else foldi (pred i) f (f acc)

let days_count school days = foldi days advance school
```

That's all we need. The runner is very simple, calculating the number of fish alive after 80 days.

``` ocaml
let problem_6_1 input = days_count input 80 |> List.length
```

## Puzzle 6 - Step 2

Step 2 is the same as step 1, just running it for 256 days. No worries.

**BLAMMO**

Uses up all the RAM. We need a different approach.

The issue is that generation of the lists of fish just results in lists that are too large, each with an increasing longer duration to calculate the next step. We need to consider a different data structure to hold the fish details.

I opted for a frequency based approach, mapping a school of fish into a structure that counts how many fish are in each timer state. OCaml has a nice list structure for this, called an association list. It's a tuple list where the first field of a tuple is a key, and the second is the value related to that key. Interestingly, this is exactly the same structure that the frequencies function I developed uses. Lets transform a school of fish into counts of the number of fish at each state.

``` ocaml
let school input = Utils.frequencies input
```

Then we need to figure out how to advance the data structure to the next day. Given any state (e.g., `(5,7)` representing 7 fish with timer 5), the next state for this is `(4,7)`, so each step can just decrement the first number in the tuple, and keep the value the same. This is true for states 8, 6, 5, 4, 3, 2, 1. Not so much for 7, and 0. The reason why is that at 0, we spawn another fish at 8, and regenerate the current one at 6. So, some special handling is needed. If we didn't have the special handling, we'd get a double up in the keys. E.g., we process `(7,5)` -> `(6,5)`, and then we process `(0,2)` -> `[(6,2);(8,2)]` resulting in two tuples with a key of 6. The OCaml List module has a function `assoc` that we can use to lookup how many fish there are of a given state. This function also handily throws an exception when the key is not found. So, when we see state 0, we need to check to see how many 7's there were, and add the number of new 6's from the 0's, to the number of new 6's from the 7's. When processing the 7's, we need to generate the 6's if and only if there are no 0's in the input (otherwise we'd be double generating the 6's). This is all a bit convoluted, but it works.

``` ocaml
let advance2 (school : (int * int) list) =
  List.sort (fun (a, _) (b, _) -> compare b a) school
  |> List.map (fun (k, v) ->
         match k with
         | 0 ->
             let new_sixes = try List.assoc 7 school with Not_found -> 0 in
             [ (8, v); (6, new_sixes + v) ]
         | 7 -> (
             let has_zeros =
               try Option.some (List.assoc 0 school)
               with Not_found -> Option.none
             in
             match has_zeros with None -> [ (6, v) ] | _ -> [])
         | _ -> [ (pred k, v) ])
  |> List.flatten
```

Then we run it.

``` ocaml
let problem_6_2 input =
  let school = school input in
  foldi 256 advance2 school |> List.map (fun (_, v) -> v) |> Utils.sum
```

RUNS LIKE LIGHTNING! ⚡️ ⚡️ ⚡️

The RAM utilisation here is tiny compared with the last approach. This data structure has at most 9 elements in it, each with 2 ints inside. Granted, they can be very large ints, but there are only 9.

## Parsing

The parsing solution for this was very simple. Single line input, separated by commas.

``` ocaml
let parse_input filename =
  let lines = Utils.read_lines filename in
  List.map
    (fun l -> Str.split (Str.regexp ",") l |> List.map int_of_string)
    lines
  |> List.flatten

let input = parse_input "problem_6.input"
```

I really enjoyed this puzzle. The first parts solution was elegant and simple to understand. The second parts complexity against a naive solution was a good challenge, and really satisfying to improve performance by such a massive amount. Fun.
