{:layout :post, :title "Advent of Code 2021 - Puzzle 2", :date "2021-12-02" :draft? false}

## Puzzle 2 Step 1

For this puzzle, a sequence of instructions need to be interpreted and change position of a submarine. My solution here was to parse the instructions into a sequence of a custom type that encodes the instructions within the type, and then reduce over that list calculating the position of the submarine. The submarine movements can easily be interpreted via a pattern match.


Lets introduce two cutom types. One for the instructions (`direction`) and one for the submarine position.

``` ocaml
type direction = Forward of int | Up of int | Down of int | Unknown

type sub_pos = { hpos : int; depth : int }
```

Then we need a way to get the input values. The file has an instruction per line, and we need to split the instructions into their direction and "amount" components. We map over the lines.

``` ocaml
(* OCaml doesn't support \s+ ?!*)
let split_on_space x = Str.split (Str.regexp "[ ]+") x
```

For converting the strings into instructions, we can use a pattern match. In Haskell I'd have used a `Parser`, but I haven't done enough investigation in OCaml around specific support for this. This solution seems OK though. I've implemented an `Unknown` value to handle any instructions that don't match. None were encountered though.

``` ocaml
let direction_generator s i =
  match s with
  | "forward" -> Forward (int_of_string i)
  | "down" -> Down (int_of_string i)
  | "up" -> Up (int_of_string i)
  | _ -> Unknown

let convert_input (instruction::num::_) = direction_generator instruction num;
```

For handling the movement, we implement a simple match that will create a new `sub_pos` type from the move and the provided position.

``` ocaml
let move acc d = match d with
  | Forward i -> {hpos= acc.hpos + i; depth= acc.depth}
  | Up i -> {hpos= acc.hpos; depth= acc.depth - i}
  | Down i -> {hpos= acc.hpos; depth= acc.depth + i}
  | Unknown -> {hpos= acc.hpos; depth= acc.depth}
```

And then we just tie it all together

``` ocaml
let answer = List.map split_on_space input |> List.map convert_input |> List.fold_left move {hpos = 0; depth = 0} in
  answer.hpos * answer.depth
```

## Puzzle 2, Step 2

There are a couple of changes required for step 2. The first is that the custom type that represents the position of the sub needs a third datapoint to represent the aim. The second is that the movement calculation needs to be changed. No biggie.

``` ocaml
type sub_pos2 = { hpos : int; depth : int ; aim: int}

let move2 acc d = match d with
  | Forward i -> {hpos= acc.hpos + i; depth= acc.depth + (acc.aim * i); aim= acc.aim}
  | Up i -> {hpos= acc.hpos; depth= acc.depth; aim = acc.aim - i}
  | Down i -> {hpos= acc.hpos; depth= acc.depth; aim = acc.aim + i}
  | Unknown -> {hpos= acc.hpos; depth= acc.depth; aim = acc.aim}
```

And then, we just execute with this different move function.

``` ocaml
    let answer = List.map split_on_space input |> List.map convert_input |> List.fold_left move2 {hpos= 0; depth=0; aim=0} in
    answer.hpos * answer.depth
```
