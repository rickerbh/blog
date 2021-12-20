{:layout :post, :title "Advent of Code 2021 - Puzzle 4", :date "2021-12-04" :draft? false :tags ["aoc" "aoc2021" "ocaml"]}

## OCaml - How it's going

I'm feeling much more confident with OCaml now, and becoming familiar with the standard List module. The helper functions I've been developing have had a surprising amount of reuse. Things I've discovered so far:
- tiny functions are great in OCaml - single line functions are your friend.
- pattern matching is pervasive - I'm not sure if that's just because I only know the one tool, but it's a great facility, and the compiler really helps with the requirement for exhaustive matching.
- type hints - when I'm struggling a bit to get the types to line up on a function, it really helps to provide the typehints on the function. This ensures that the compiler complains about the code in the function not matching the signature, rather than inferring the "wrong" signature and complaining about the callers of the code.

## Puzzle 4 - Step 1

The puzzle today is to build a bingo card solver/checker. To solve this, I first considered the types we'd need to represent the card structure. These cards are made up of a 5x5 grid of integers. We also need to have a columnar view of the card, so we can check for winning cards. We can transform the rows to columns with the `transpose_matrix` function developed in the last puzzle.

``` ocaml
type row = int list
type card = row list

let columns card =
  Utils.transpose_matrix card
  |> List.filter (fun x -> 0 == List.length x |> not)
```

To find if a card is a winner, either a full row or full column must be marked as seen.

``` ocaml
let row_won numbers row = List.for_all (fun x -> List.mem x numbers) row

let any_true xs =
  List.fold_left (fun acc x -> match acc with true -> true | _ -> x) false xs

let card_won numbers card =
  let rows_won = List.map (row_won numbers) card in
  let columns_won = List.map (row_won numbers) (columns card) in
  any_true (List.append rows_won columns_won)
```

We are also dealt a set of cards, so we must be able to check many cards. This uses Option to wrap the winning cards, and then filter_map to unwrap the option and remove the Nones. To determine the winning card, we also need to be able to extract the first winning card. This wraps the result in Option.

``` ocaml
let winning_cards numbers cards =
  List.filter_map
    (fun c ->
      let has_won = card_won numbers c in
      match has_won with true -> Some c | false -> Option.none)
    cards

let first_winning_card numbers cards =
  let (winning_cards : card list) = winning_cards numbers cards in
  match winning_cards with
  | [] -> Option.none
  | _ -> Some (List.hd winning_cards)
```

Finally we can assemble all this into a game playing function. This function will reduce over the numbers provided, and when it first encounters a winning card, will continue to return the number sequence to that point and the winning card (via the match for Some).

``` ocaml
let play_game numbers cards =
  List.fold_left
    (fun (priors, result) x ->
      match result with
      | Some _ -> (priors, result)
      | None ->
          let new_priors = x :: priors in
          let c = first_winning_card new_priors cards in
          (new_priors, c))
    ([], Option.none) numbers
```

The scoring mechanism needs to determine which numbers on the winning card were not seen, as well as the last number in the sequence that caused this card to be the winner. This list is summed, and multiplied by the last number. To figure out the numbers not see, we can calculate the difference between the numbers on the card and the numbers that made up the presented sequence.

``` ocaml
let difference l1 l2 = List.filter (fun x -> not (List.mem x l2)) l1
```

Then the score calculation just takes the difference, the last number in the sequence, and multiplies them together.

``` ocaml
let calculate_score numbers card =
  let missing_numbers_sum =
    List.map (fun x -> difference x numbers) card |> List.flatten |> Utils.sum
  in
  let last_number = List.hd numbers in
  missing_numbers_sum * last_number
```

And we plug it all together...

``` ocaml
let problem_4_1 input =
  let numbers, cards = input in
  let winning_numbers, winning_card = play_game numbers cards in
  match winning_card with
  | None -> -1
  | Some c ->
      let score = calculate_score winning_numbers c in
      score
```

## Puzzle 4 - Step 2

The change with step 2 is that all cards need to be solved, and the last card to be solved is to be used to calculate the score. With the `play_game` function from part 1, I implemented the game solver as a fold, which would run over the entire set of numbers but keep the winning sequence and card. This solution does not work with the second step, as the order of solving is important, and running the solution for step 1 to the end doesn't show the order that the cards were solved in.

The game runner for step 2 was implemented as a recursive solver, which keeps the cards solved in solving order, and reduces the unsolved card set as the cards get solved. This way, when the unsolved card set is empty, we can return the last solved card.

``` ocaml
let play_game_part_2 numbers cards =
  let rec finder priors winners numbers cards =
    match List.length cards with
    | 0 -> (priors, winners)
    | _ -> (
        match numbers with
        | [] -> (priors, []) (* unsolvable *)
        | x :: _ ->
            let new_priors = x :: priors in
            let these_winners = winning_cards new_priors cards in
            let new_winners = winners @ these_winners in
            let remaining_cards = difference cards new_winners in
            finder new_priors new_winners (Utils.drop 1 numbers) remaining_cards
        )
  in
  finder [] [] numbers cards
```

And the function to play then score:

``` ocaml
let problem_4_2 input =
  let numbers, cards = input in
  let winning_numbers, winning_cards = play_game_part_2 numbers cards in
  let winning_card = List.rev winning_cards |> List.hd in
  let score = calculate_score winning_numbers winning_card in
  score
```

## Parsing

The parsing solution for the input file was a bit different than prior puzzles as the file had multiple "sections" representing the selected numbers, and the cards, and then within the card section all the different cards were listed. I'm not that happy with my solution here, but it works and I consider it secondary to the puzzle solving logic.

To parse out a card we split the lines that represent the numbers on the card by space, and then convert them into a list of integers. To then parse out the cards, we split the input into block of 5 lines (5 rows per card), parse the card, then skip the card and space separator by dropping 6 lines from the input.

``` ocaml
let parse_card lines =
  List.map
    (fun l -> Str.split (Str.regexp "[ ]+") l |> List.map int_of_string)
    lines

let rec parse_cards lines =
  match lines with
  | [] -> []
  | _ ->
      let card_lines = Utils.take 5 lines in
      let card = parse_card card_lines in
      card :: parse_cards (Utils.drop 6 lines)
```

To finalise the parsing, the total input includes the list of numbers in the first line, and then an empty line as the separator. Then, the cards.

``` ocaml
let parse_input filename =
  let lines = Utils.read_lines filename in
  let numbers =
    List.hd lines |> Str.split (Str.regexp "[,]") |> List.map int_of_string
  in
  let cards = parse_cards (Utils.drop 2 lines) in
  (numbers, cards)
```
