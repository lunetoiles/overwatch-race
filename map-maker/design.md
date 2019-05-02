# Map Editor

Design document for the map editor. Also useful as an editor for placing nodes for any game type.

## User Interface Flow
![User interface flow chart](https://raw.githubusercontent.com/lunetoiles/overwatch-race/master/map-maker/map-maker-flow.png)

## Global Variable Definitions

    A - map node position list
    B - map node size list
    
    I,J,K,L - intermediate values

    M - edit map node pos
    N - edit map node size
    
    P - edit map node index

    Q - adjustment size
    
    W - facing display

    Y - Prev state
    Z - State


## State Machine

All below rules are of rule type "Ongoing - Each player" unless otherwise noted.

Unmarked variables are global variables, even in event player rules

All below rules have an implied condition of `"gZ == {state in rule name}"`

All rules with a button listed in rule name have an implied condition of `is pressed(ep, {button}) == true`

**State 0 - initialization**

Rule type: Ongoing - global

    A <= [(0,0,0)]
    B <= [3]
    
    M <= (0,0,0)
    N <= 3
    P <= 0
    
    Q <= 1
    
    state <= 1


**State 1 - create hud edit node text**

Rule type: Ongoing - global
    
    Create hud text ( //index, position, and size display
        text: "#{P}: {M}, {N}
        visible: filtered array( all players, O < count of(M) )
        index: 10
    )
    Create hud text ( //Add new display
        text: "#{P}: New!
        visible: filtered array( all players, O == count of(M) )
        index: 10
    )
    Create hud text (
        text: "place checkpoint" //or as close as I can get
        visible: filtered array( all players, state == 20 )
        index: 11
    )
    create hud text (
        text: "left/right" //or as close as I can get
        visible: filtered array( all players, state == 32 )
        index: 11
    )
    create hud text (
        text: "up/down" //or as close as I can get
        visible: filtered array( all players, state == 33 )
        index: 11
    )
    create hud text (
        text: "Size" //or as close as I can get
        visible: filtered array( all players, state == 34 )
        index: 11
    )
    create hud text (
        text: "choose checkpoint" //or as close as I can get
        visible: filtered array( all players, state == 40 )
        index: 11
    )  
    
    state <= 2
    
**state 2 - create node value hud**

Rule type: Ongoing - global

    (repeat x20)
    Create hud text (
        Text: "{index}: {A[{index}]}, {B[{index}]}"
        sort: {index}
        visible to: all players
        reevaluate: all
    )
    
    state <= 3

**state 3 - create extra hud text**

Rule type: Ongoing - global

    {make facing text}
    {make edit size text}
    
    state <= 10

**state 10 - create node display**

Rule type: Ongoing - global

    (repeat x20)
    Create effect (
        Type: sphere
        position: A[{index}]
        size: B[{index}]
        color: white
        visible to: filtered array( All players, state == 40 || P != {index} )
        reevaluate: all
    )
    
    state <= 11


**State 11 - Set up edit effect**

Rule type: Ongoing - global

    Create effect (
        Type: sphere
        position: M
        size: N
        color: purple
        visible to: filtered array( All players, state >= 20 && state <= 39 )
        reevaluate: all
    )
    Create icon (
        Type: circle
        position: M
        color: blue
        visible to: All players
        reevaluate: all
    )
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




