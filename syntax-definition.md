# Syntax Definition

This document defines the format used fo the design documents in this repository. The syntax and layout
is meant to make state machine based logic easy to read and write, while still being easy to mentally
convert to the in game drop-down boxes when entering the code. Maybe someday there will be a compiler and
an autohotkey script for entering the code, but for now its just a semi-standardized format.

## Values and actions syntax

The syntax used for actions and values. They can also simply be written in the function
call form that is displayed once you close the edit window. Things in `< >` brackets are other values.

### Values

The following list uses A as a stand in for A-Z.

`gA` - global variable A

`<player>:A` - Player variable A belonging to `<player>`

`A` - Unless otherwise noted, this means `gA` in "Ongoing - Global" rules, and `ep:A` in all other rule types

`state` - Unless otherwise noted, this means `gZ` in "Ongoing - Global" rules, and `ep:Z` in all other rule types

`ep` - Event player

`cu` - Current array element

`<array>[<index>]` - Value in array(`<array>`, `<index>`)

`<array>[<start>:<end>]` - Array slice from `<start>` to `<end>` inclusive

`<array>[<start>:]` - Array slice from `<start>` to the end of the array

`[]` - Empty array

`<value 1> + <value 2>`, `<value 1 - <value 2>`, etc:

The functions `add(<value 1>, <value 2>)`, `subtract(<value 1>, <value 2>)`, etc

`<value 1> == <value 2>`, `<value 1> >= <value 2>`, etc:

The functions `compare(<value 1>, ==, <value 2>)`, `compare(<value 1>, >=, <value 2>)`, etc

`"Checkpoint {<value 1>}: {<value 2>}, {<value 3>} M"`:

A corresponding tree of string values to make the text in the quotes, using the specified values.

### Actions

The following list used A as a stand in for global or player variable A-Z, using the corresponding action for each.

`A <= <value>` - Set variable to value.

`A += <value>`, `A -= <value>`, etc  - Modify variable with action add, subtract, etc and value.

`append A <= <value>` - Modify global with action append and value.

`A[<index>] <= <value>` - Set variable at `<index>` to value

`A <= [<value 1>, <value 2>, ...]`

Set variable to be an array with the sequnce of values.
In game it will be setting A to empty array and then a sequence of appends. 

    skip ( <condition> ) {
        <action 1>
        <action 2>
        ...
    }

The action `skip if true ( <condition>, <number of actions in brackets> )`.

## Other syntax

    A += 1 //increment the score counter

Anything after `//` is a comment.

    {make the thing}

Anything written in curly brackets stands in for something customizable, or that just
hasn't been written out explicitly yet.

    (repeat 0 to 10)
    A[{index}] <= gB[{index}] + 1

Make copies of the actions after the repeat line for each number specefied. In each copy, replace
`{index}` with the corresponding number.

## Full workshop rule syntax

Below is an example of a full workshop rule as used in the design document. The text in bold is the rule title. After that
is the type of the rule, if it is different from the typical type used in the section. Next is the conditions
for the rule, if any. Each condtion is prefixed with `cond:`. After the conditions is the sequence of actions in the rule.

The below example has an implied condtion of `state == {state in rule name}` as is the case in most sections of the
design documents.

**State 20 and team 2 in checkpoint**

Rule type: Ongoing - Each player, team 2, all

    Cond: distance between( ep, gA[gP] ) < 10  //player inside the checkpoint

    S += 1 //increase the players score
    gP += 1 //advance the checkpoint index

    state <= 30 //create next checkpoint state

