# Map Editor

Design document for the map editor. Also useful as an editor for placing nodes for any game type.

## User Interface Flow
![User interface flow chart](https://raw.githubusercontent.com/lunetoiles/overwatch-race/master/map-maker/map-maker-flow.png)

## Global Variable Definitions

    A, B, C, D - intermediate values
    
    J - map node pos list
    K - map node size list
    L - world icon for current index
    M - edit map node pos
    N - edit map node size
    O - edit map node index
    P - edit node effect
    Q - adjustment size
    R - Adjustment delta
    S - map node effects
    T - mode quit
    W - facing display
    X - next index
    Y - Prev state
    Z - State


## State Machine

All below rules are of rule type "Ongoing - Each player" unless otherwise noted.

Unmarked variables are global variables, even in event player rules

All below rules have an implied condition of `"gZ == {state in rule name}"`

All rules with a button listed in rule name have an implied condition of `is pressed(ep, {button}) == true`

**State 0 - initialization**
Rule type: Ongoing - global

    J <= [(0,0,0)]
    K <= [3]
    L <= []
    S <= [null]
    
    M <= (0,0,0)
    N <= 3
    O <= 0
    
    state <= 1


**State 1 - create hud texts**
Rule type: Ongoing - global


    Create checkpoint place text
    Create x edit text
    Create y edit text
    Create z edit text
    Create size edit text
    Create index edit text
    Create world icon for current index
    Create “NEW” text
    
    state <= 2

**state 2 - create node display 0 through 9**
Rule type: Ongoing - global

    Do the thing, however many times and rules
    state <= 10
etc to make all the display nodes

**State 10 - Set up edit effect**
Rule type: Ongoing - global


    create effect(
	    pos: M
	    size: N
	    vis: all
	    reeavluate: pos and size
    )

    P <= last created entity
    state <= 20 //edit effect set position


**state 20 and primary fire - move effect**

    M <= pos of( event player )
    Y <= 20
    state <= 91


**state 20 and interact - switch to detail edit mode**

    R <= Left
    goto state 30


**state >= 30 and state <= 32 and primary fire - add vector delta**

    M += R * Q
    Y <= state
    wait .25
    loop if conditions true


**state >= 30 and state <= 32 and secondary fire - subtract vector delta**

    M -= R * Q
    Y <= state
    wait .25
    loop if conditions true


**state is 33 and primary fire - increase size**

    N += R
    Y <= state
    wait .25
    loop if conditions true


**state is 33 and primary fire - decrease size**

    N -= R
    Y <= state
    wait .25
    loop if conditions true


**state is 30 and ability 1 - change to Y detail edit mode**

    R <= Up
    Y <= 31
    state <= 91


**state is 31 and ability 1 - change to Z detail edit mode**

    R <= Forward
    Y <= 33
    state <= 91


**state is 32 and ability 1 - change to size detail edit mode**

    Y <= 33
    state <= 91

**state is 33 and first skill - change to X detail edit mode**

    R = Left
    Y <= 30
    state <= 91


**state is >=30 and state <= 33 and interact - change to place mode**

    state <= 21

**state is >= 20 and sate <= 39 and ultimate - go to index change mode**

    state <= 41



**State 41 - put edit node into list**

    do the things to put edit node into display list as generic white sqhere
    delete edit effect
    state <= 40


**state 40 and primary fire - increase index**

    abort if O = count of (S)
    O+=1


**state 40 and secondary fire - decrease index**

    abort if O = 0
    O-=1


**state 40 and ultimate  and O < count of (S) - goto edit existing node**

    Y <= 43
    state <= 91


**state 40 and ultimate  and O == count of (S) - goto append node**

    Y <= 44
    state <= 91


**state 40 and interact and O < (count of (S)-1)- goto insert node**

    O += 1
    Y <= 45
    state <= 91


**state 40 and interact and O >= (count of (S)-1) - goto append node**

    O <= count of (S)
    Y <= 44
    state <= 91


**state 40 and ability 1 and O < count of (S) - goto delete node**

    Y <= 46
    state <= 91


**State 43 - edit existing node**

    delete effect( S[O] )
    S[O] <= null
    M <= J[O]
    N <= K[O]
    state <= 10


**State 44 - append node at end**

    append null to S
    append (0,0,0) to J
    append 3 to K
    M <= (0,0,0)
    N <= 3
    state <= 10


**State 45 - insert node at index**

    M <= (0,0,0)
    N <= 3
    A <= Slice(J,0,O)
    B <= slice(K,0,O)
    C <= slice(S,0,O)
    Append A <= (0,0,0)
    Append B <= 3
    Append C <= null
    Append A <= J[O:]
    Append B <= K[O:]
    Append C <= S[O:]
    J <= A
    K <= B
    S <= C
    state <= 10


**State 46 - delete node at index**

    Destroy S[O]
    A <= Slice(J,0,O)
    B <= slice(K,0,O)
    C <= slice(S,0,O)
    Append A <= J[O+1:]
    Append B <= K[O+1:]
    Append C <= S[O+1:]
    J <= A
    K <= B
    S <= C
    state <= 40



**state 91 - release all the buttons state**

    Cond: all buttons i'm checking == false
    state <= Y




