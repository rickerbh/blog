{:layout :post, :title "Advent of Code 2021 - Puzzle 5", :date "2021-12-05" :draft? false :tags ["aoc" "aoc2021" "ocaml"]}

## Puzzle 5 - Step 1

Todays puzzle is about mapping a bunch of lines that have start/end coordinates, and checking for intersections. We need to count the number of intersections in the map. We are also to only consider horizontal or vertical lines.

To model this, I implemented two new types to represent points, and lines.

``` ocaml
type point = int * int

type line = point * point
```

Lines are to be defined by start/end points, that look like this: `((0, 9), (5, 9))`.

To include only horizontal or vertical lines, we need a checker. To check for horizontal or verticalness, we need to ensure that either the first 2 values or the second 2 values of the points are the same.

``` ocaml
let is_horizontal_or_vertical line =
  let (x1, y1), (x2, y2) = line in
  match (x1, y1, x2, y2) with
  | a, _, b, _ when a = b -> true
  | _, a, _, b when a = b -> true
  | _, _, _, _ -> false
```

To calculate the intersections, we need to consider all points along a line. So, lets generate them. I developed a function that given 2 ints, will move them closer to each other (`to_target`). We can then use this by passing in the first 2 and last 2 values from the start/end of the line, and they'll converge. We recursively generate the points of a line, until they meet. The `expanded_lines` function will take an input and generate the points for each line (i.e., the full board).

``` ocaml
let to_target a b =
  match (a, b) with
  | _, _ when a < b -> a + 1
  | _, _ when a > b -> a - 1
  | _ -> a

let rec generate_line_points line =
  let start_point, end_point = line in
  match start_point = end_point with
  | true -> [ start_point ]
  | _ ->
      let (x1, y1), (x2, y2) = line in
      let next_x = to_target x1 x2 in
      let next_y = to_target y1 y2 in
      start_point :: generate_line_points ((next_x, next_y), end_point)

let expanded_lines input = List.map generate_line_points input
```

This gives us all the tools we need to generate the list of all lines. Next step is to determine the "dangerous" points on the board i.e., where the lines intersect. To do this, I use a `frequency` function to count how many lines are at each point, and then match each of these and emit true when there are more than 1.

``` ocaml
let is_dangerous_vent t = match t with _, f when f > 1 -> true | _, _ -> false
```

And tying it all together...
``` ocaml
et problem_5_1 input =
  let lines =
    List.filter is_horizontal_or_vertical input
    |> expanded_lines |> List.flatten
  in
  let freqs = Utils.frequencies lines in
  let dangerous_count = List.filter is_dangerous_vent freqs |> List.length in
  dangerous_count
```

This works, but it is **SLOW**.

## Puzzle 5 - Step 2

The twist with step 2 is that we need to include the diagonal lines. I think I cheated a bit here. The instructions were explicit with only 45º lines being supported, but I just removed the filter check for horizontal and vertical, and I was done 😅

``` ocaml
let problem_5_2 input =
  let lines = expanded_lines input |> List.flatten in
  let freqs = Utils.frequencies lines in
  let dangerous_count = List.filter is_dangerous_vent freqs |> List.length in
  dangerous_count
```

Also **SLOW**.

## Parser

This parser is a bit clunky. I needed to turn a string of `0,9 -> 0,5` into a `((0, 9), (0, 5))`. This is just a series of nested maps, and some fun with optionals.

``` ocaml
let parse_line_to_ints lines =
  let pairs =
    List.map (fun l -> Str.split (Str.regexp " -> ") l) lines
    |> List.map (fun l ->
           List.map
             (fun p ->
               let t = Str.split (Str.regexp ",") p |> List.map int_of_string in
               match t with a :: b :: _ -> Option.some (a, b) | _ -> None)
             l)
    |> List.map Utils.deoptionalize
    |> List.map (fun xs ->
           match xs with a :: b :: _ -> Option.some (a, b) | _ -> None)
    |> Utils.deoptionalize
  in
  pairs
```

And the familiar function to read some lines from a file.
``` ocaml
let parse_input filename =
  let lines = Utils.read_lines filename in
  parse_line_to_ints lines
```

## Slow

<img src="https://memegenerator.net/img/instances/49324497.jpg" width="512" height="380" />

Haven't figured it out yet. Obviously one of these steps is taking a very long time, or maybe many are, or maybe there are just a lot of steps. I started to investigate potential performance tweaks with the help of [Performance and Profiling on OCaml.org](https://ocaml.org/learn/tutorials/performance_and_profiling.html). The low hanging fruit was to de-genericise some of the functions by providing type hints to the compiler. I'm not sure if this actually helped, as it was taking upwards of 220 seconds to run on my machine, so I feel that unless there was a massive change in speed, this wasn't the issue. There must be a fundamentally different way to solve this problem.

I also looked at generating the annotated execution stats with ocamlprof, but I'm already behind with these puzzles and couldn't immediately figure out how ocamlprof interacts with dune. 🤷
