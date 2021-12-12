{:layout :post, :title "Advent of Code 2021 - Puzzle 3", :date "2021-12-03" :draft? false}

## Preface

The OCaml dev tools are doing my head in. I know this must be 100% my fault, but it's so frustrating not being able to do seemingly simple things with the tooling. My main issues are that I can't get a bit of shared code actually shared, and I'm struggling to find a sensible way to structure the multiple days of puzzles into separate sets of code that don't have ridiculous build setups. Anyway, I took some time out and worked on this for a night. I'm backdating these posts for a while...

What I ended up doing to resolve the tooling issues was:
- Separate each days code into a separate directory with it's own dune file that produces an exectutable.
- At the root directory that contains each of these day dirs, I have a dune-project file.
- The shared code is in another dir, with it's own separate dune file that produces a library.

To compile, I can `dune build` from the root project dir and each of the subdirectories is built, as well as the library code.

To run (this is still painful), I need to `cd` to each directory and then `dune exec ./main.exe` the application. Mine are all called main for convenience. What I'd like to do is have a top level script that can execute each of the days programs, so they can all run one after each other.

This is good enough for now. There still have issues where I can't reference the library code from within the individual programs in VSCode, but it works when building.

## Puzzle 3 - Step 1

This puzzle involves manipulating a list of strings that represent binary data as a series of `0` and `1` characters. There are a bunch of manipulation functions that I needed to create to be able to convert the string into other types for simpler manipulation. I "borrowed" some of this code from stack overflow. I guess I didn't consider this to be the essence of the puzzle, but maybe it was 🤷

``` ocaml
type binary = bool list

let rec int_of_bin = function
  | [] -> 0
  | true :: bs -> 1 + (2 * int_of_bin bs)
  | false :: bs -> 2 * int_of_bin bs

let bool_of_bin_char = function '1' -> true | _ -> false

let int_of_bin_char xs = List.rev xs |> int_of_bin

let explode s = List.init (String.length s) (String.get s)

let bools_of_bin_string s = List.map bool_of_bin_char (explode s)
```
`int_of_bin` turns a list of `true` `false` values into an integer (in reverse). `explode` will take a string and turn it into a list of `char`. The `int_of_bin_char` function takes a list of true/false values, reverses it, and passes this into `int_of_bin`.

To actually solve the puzzle we need to calculate the most common bit in each position in each of the provided binary strings. The easiest way I could think about solving this was to do a matrix transformation on the binary strings, getting rows of each position. There were a couple of helper functions to do this, but the general gist was to get a list of all the first elements in the list, then all the rest of the elements.

``` ocaml
let rec first_elems ys =
  match ys with (x :: _) :: xss -> x :: first_elems xss | _ -> []

let rec rest_elems ys =
  match ys with (_ :: xs) :: xss -> xs :: rest_elems xss | _ -> []

let rec transpose_matrix xs =
  match xs with
  | [] -> []
  | _ ->
      let rest = rest_elems xs in
      first_elems xs :: transpose_matrix rest
```

Once we have the data rotated, we need to start thinking about how to calculate the 2 rates. The algorithm for the rate calculation is basically the same, except that the epsilon rate takes the most common bit, and the gamma rate takes the least common bit. I solved this by calculating the frequency of each bit type in each item (which because it's rotated, represents a bit position), and then sorting these frequencies by highest/lowest values for the 2 rates. Again, there were a couple of helper functions for this...

``` ocaml
let count_item acc v =
  let count = List.assoc_opt v acc in
  match count with
  | Some x -> (v, x + 1) :: List.remove_assoc v acc
  | None -> (v, 1) :: acc

let sort_tuple_second_int x y =
  let xv, xc = x in
  let _, yc = y in
  match (xc, yc) with
  | _ when xc > yc -> -1
  | _ when xc < yc -> 1
  | _ -> if xv then -1 else 1

let sort_reverse_tuple_second_int x y =
  let xv, xc = x in
  let _, yc = y in
  match (xc, yc) with
  | _ when xc > yc -> 1
  | _ when xc < yc -> -1
  | _ -> if xv then 1 else -1

let take_first xs = match xs with [] -> None | x :: _ -> Some x

let extract_first t =
  let a, _ = t in
  a
```

- `count_item` will update `acc` (list of tuples) by incrementing a count if `v` is already seen, otherwise it pops in `(v,1)` into the list.
- The 2x sorting functions are comparators that will return a sort result based on the second value of a tuple.
- `take_first` and `extract_first` will get out the first element of a list and the first element of a tuple respectively. These will also sort the `true` and `false` fields first respectively, if the second tuple elements are equal.

This gives us all the base pieces we need to solve the puzzle.

``` ocaml
let calculate_problem_3_1_rate counts sort_f =
  counts
  |> List.map (fun x -> List.sort sort_f x)
  |> List.map take_first |> Utils.deoptionalize |> List.map extract_first
  |> int_of_bin_char

let problem_3_1 input =
  let counts =
    List.map bools_of_bin_string input
    |> transpose_matrix
    |> List.map (fun x -> List.fold_left count_item [] x)
  in

  let epsilon_rate = calculate_problem_3_1_rate counts sort_tuple_second_int in
  let gamma_rate =
    calculate_problem_3_1_rate counts sort_reverse_tuple_second_int
  in
  epsilon_rate * gamma_rate
```

Previously I noted that the gamma and epsilon rate calculation only differ in sort order, so `calculate_problem_3_1_rate` is a solver for both, which differs based on a provided sorting function. It will receive a list of counts of each of the values, sort them according to the frequency, take the first of each set (highest/lowest), deoptionalize them (helper function to convert `Some x -> x` and remove the `None`s), then extract the first element from each tuple (true/false), and then convert that list into an int.

`problem_3_1` will convert the string input into the true/false lists, transpose the array, and perform the frequency calculation. It then calls the calculate function with each of the sorting algos, and takes these results and multiplies them together to give the answer.

## Puzzle 3 - Step 2

Again, this is a variant on the first puzzle. All the required helpers are the same, and the function that does the heavy lifing on calculation for each of the two values only differs by sort order. My solution for this feels a bit gross though. I'm sure there are more elegant ways to solve this. Anyway, here goes. The solution for this required calculating a result from the first bit position, and using this to filter the string values to then run the next result calculation over.

``` ocaml
let rec calculate_problem_3_2_rate sort_f remaining idx =
  match List.length remaining with
  | 0 -> []
  | 1 -> List.nth remaining 0
  | _ ->
      let counts =
        List.map (fun x -> Utils.drop idx x) remaining
        |> transpose_matrix
        |> List.map (fun x -> List.fold_left count_item [] x)
        |> List.map (fun x -> List.sort sort_f x)
        |> take_first
      in
      let first_set =
        match counts with Some xs -> take_first xs | _ -> Option.none
      in
      let first_one =
        (* gross solution - defaults to false, not that it should ever be used though *)
        match first_set with Some xs -> extract_first xs | _ -> false
      in
      let new_remaining =
        List.filter (fun x -> Bool.equal first_one (List.nth x idx)) remaining
      in
      calculate_problem_3_2_rate sort_f new_remaining (idx + 1)

```

What this function does is recursively calculate the most/least common bit, and uses this to filter the `remaining` to reduce the possible answer set. The idx value indicates which position we are up to, so we can drop that many "bits" from the input strings and then run the transform/count functions over them. `first_set` is the first item from an the list, and first_one is the first tuple from within that item. We then filter the `remaining` list based on checking the value in the converted string with the most/least frequent bit. There are 3 things I don't like about this.
1) It feels like it's too long. I think I should be able to extract the first value from the first tuple in the first element in an array in fewer than 7 lines of ocamlformat formatted code. This extraction mess obscures the algorithm. I could hide it out in a separate function I guess.
2) There are a couple of patterns in the matches that are only there to please the compiler. In the first one, the empty set should never be hit. In the second one (the one prefixed with `gross solution`), the `_ -> false` never executes as the item that goes into it always has data. I guess a dependently typed language could solve this, but maybe it's not possible without some more custom type work in OCaml.
3) The counts for all bits are calculated each iteration. There is a bunch of wasted cycles.

And to tie it all together...

``` ocaml
let problem_3_2 input =
  let xs = List.map bools_of_bin_string input in
  let oxy_rate =
    calculate_problem_3_2_rate sort_tuple_second_int xs 0 |> int_of_bin_char
  in
  let cos_rate =
    calculate_problem_3_2_rate sort_reverse_tuple_second_int xs 0
    |> int_of_bin_char
  in
  cos_rate * oxy_rate
```

Long solution, but the long code was mostly helper functions for sorting and extracting data.
