{:layout :post, :title "Advent of Code 2021 - Puzzle 7", :date "2021-12-07" :draft? false :tags ["aoc" "aoc2021" "ocaml"]}

## Puzzle 7 - Step 1

This puzzle is about calculating the minimum distance to a common integer for a set of integers. OK - the puzzle talks about crab submarines and whales, but that's not that important.

There is a sneaky trick to calculate the midpoint in a set of integers. You can sort the list, and then look at the middle value. This is the minimum distance to the rest of the numbers in the set.

``` ocaml
let middle input =
  let list = List.sort compare input in
  let half_length = List.length list / 2 in
  List.nth (Utils.drop half_length list) 0
```

Then, we need to calculate the actual distances to this middle.

``` ocaml
let fuel_usage position crabs =
  List.map (fun x -> if x > position then x - position else position - x) crabs
  |> Utils.sum
```

And that's it!

``` ocaml
let problem_7_1 input = fuel_usage (middle input) input
```

## Puzzle 7 - Step 2

Step 2 is a bit trickier. It changes the fuel calculation function, so that the minimum distance is no longer the sorted-middle number. The further away a point is from the minimum distance number, it gets more "expensive" at a greater than linear rate. Eg, if you are 4 away, the cost is 10 = 1+2+3+4.

So, a new fuel calculation function that works with this increased cost per step of distance, as well as a function to calculate for each crab.

``` ocaml
let rec crabby_fuel i = match i with 0 -> 0 | _ -> i + crabby_fuel (pred i)

let fuel_usage2 position crabs =
  List.map
    (fun x ->
      if x > position then crabby_fuel (x - position)
      else crabby_fuel (position - x))
    crabs
  |> Utils.sum
```

To figure out the minimum distance, I tried a number of options. The middle is obviously wrong because the test data fails, but the average works with the test data. However, this method doesn't work on the full data set (more on this later). So, I just brute-forced it.

All possible answers must be in the range of the minimum value in the list to the maximum value in the list. Lets get the min and max values, and generate a range of numbers.

``` ocaml
let list_min xs = List.sort compare xs |> List.hd

let list_max xs = List.sort compare xs |> List.rev |> List.hd

let range s e = List.init (e - s) (fun x -> pred s + x + 1)
```

Then we just brute-force it by folding over the range, and keeping the min fuel usage calculation.

``` ocaml
let problem_7_2 input =
  let xs = range (list_min input) (succ (list_max input)) in
  List.fold_left
    (fun acc v ->
      let this_fuel = fuel_usage2 v input in
      if acc < this_fuel then acc else this_fuel)
    Int.max_int xs
```

This takes about 8 seconds to execute on my machine. So, not fast, but not impossibly slow.

## Parser

The parser for this is the same as yesterday - just a list of ints on a single line separated by `,`.

``` ocaml
let parse_input filename =
  let lines = Utils.read_lines filename in
  List.map
    (fun l -> Str.split (Str.regexp ",") l |> List.map int_of_string)
    lines
  |> List.flatten

let input = parse_input "problem_7.input"
```

## Average doesn't work

I have a suspicion that using the average is the key here. For the data set I was given, the using the average yeilded a result that was only 2 off the actual answer. I wonder if there is a trick here that would solve this for me. Eg, distance to the average as a float, rather than an int.
