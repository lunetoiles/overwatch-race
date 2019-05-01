# Code Syntax

This file describes the syntax for Overwatch workshop rules used in this project. As all these documents must be manually re-written in the Overwatch interface it is not a completely formal language. It is designed around making writing rules easy, not around looking exactly like the actual result inside Overwatch.

`gA` - Global variable A
`{value}:A` - Player variable A etc belonging to player `{value}`. 
 

    Example:
    A <= first of(All players)
    A:B <= 1

`A` - Global variable A in "Ongoing - Global" rules, event player's variable A otherwise
`State` - Shortcut for `Z`
`ep` - Event Player
`cu` or `current` - current array element
`A[{value}]` - Value in array at index `{value}`
`A[{start}:{end}]` - Array slice from `{start}` to `{end}` inclusive.
`A[{start}:]` - Array slice from `{start}` to end of the array.
`A <= 5` - Sets variable on the left to value on right, using appropriate action.
`A <= []` - Set variable to empty array
`A <= [1,2,3,4]` - Set variable to array values. Will be a set to empty array and sequence of append actions in game.
`A += 1`, `A -= 1`, etc - Modify player variable with correct action
`skip { (actions) }` and ` skip if ({value}) { (actions) }` - shortcut for skip and skip if, with the number to skip being the number of actions in the brackets.


