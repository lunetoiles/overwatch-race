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
        visible: filtered array( all players, state == 30 )
        index: 11
    )
    create hud text (
        text: "foreward/back" //or as close as I can get
        visible: filtered array( all players, state == 31 )
        index: 11
    )
    create hud text (
        text: "up/down" //or as close as I can get
        visible: filtered array( all players, state == 32 )
        index: 11
    )
    create hud text (
        text: "Size" //or as close as I can get
        visible: filtered array( all players, state == 33 )
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


**state 20 and primary fire - place node and player**

    Cond: is pressed(ep, ultimate) == false
    
    M <= pos of( event player )
    A[P] <= M


**state 20 and interact - switch to detail edit mode**

    State <= 30


**state 30 and primary fire - move node left**

    Cond: is pressed(ep,ability 1) == false
    
    I <= facing angle(ep)
    J <= I * Left //get x component of facing angle
    K <= I * Forward //get z component of facing angle
    L <= (K,0,J) //Get right angle to facing angle
    I <= unit vector(L) //convert to unit vecotr to make up for missing Y component
    M += I * Q //update node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**state 30 and primary fire - move node right**

    Cond: is pressed(ep,ability 1) == false
    
    I <= facing angle(ep)
    J <= I * Left //get x component of facing angle
    K <= I * Forward //get z component of facing angle
    L <= (K,0,J) //Get right angle to facing angle
    I <= unit vector(L) //convert to unit vecotr to make up for missing Y component
    M -= I * Q //update node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**State 30 and ability 1 - switch to forward/back edit**

    State <= 31


**state 31 and primary fire - move node foreward**

    Cond: is pressed(ep,ability 1) == false
    
    I <= facing angle(ep)
    J <= I * Left //get x component of facing angle
    K <= I * Forward //get z component of facing angle
    L <= (J,0,K) //remove y component from facing angle
    I <= unit vector(L) //convert to unit vecotr to make up for missing Y component
    M += I * Q //update node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**state 31 and primary fire - move node back**

    Cond: is pressed(ep,ability 1) == false
    
    I <= facing angle(ep)
    J <= I * Left //get x component of facing angle
    K <= I * Forward //get z component of facing angle
    L <= (J,0,K) //remove y component from facing angle
    I <= unit vector(L) //convert to unit vecotr to make up for missing Y component
    M -= I * Q //update node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**State 31 and ability 1 - switch to up/down edit**

    State <= 32
    
**state 32 and primary fire - move node up**

    Cond: is pressed(ep,ability 1) == false
    
    M += Up * Q //modify node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**state 32 and primary fire - move node right**

    Cond: is pressed(ep,ability 1) == false
    
    M += down * Q //modify node position
    A[P] <= M //save change
    wait .25
    loop if conditions true

**State 32 and ability 1 - switch to size edit**

    State <= 33
    
**state 33 and primary fire - increase size**

    Cond: is pressed(ep,ability 1) == false
    
    N += Q //modify size
    B[P] <= N //save change
    wait .25
    loop if conditions true

**state 33 and primary fire -decrease size**

    Cond: is pressed(ep,ability 1) == false
    
    N -= Q //modify size
    B[P] <= N //save change
    wait .25
    loop if conditions true

**State 33 and ability 1 - switch to left/right edit**

    State <= 30


**state is >=30 and state <= 33 and interact - change to place mode**

    state <= 20

**state is >= 20 and sate <= 39 and ultimate - go to index change mode**

    state <= 40


**state 40 and primary fire - increase index**

    Cond: P < count of(A)
    
    P += 1


**state 40 and secondary fire - decrease index**

    Cond: P > 0
    
    P -= 1


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




