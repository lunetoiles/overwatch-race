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

**state 0 - initial positions**

Rule type: Ongoing - global

    A <= [{initial node positions}]
    state <= 1

**state 1 - initial sizes**

Rule type: Ongoing - global

    B <= [{initial node sizes}]

**State 2 - editor initialization**

Rule type: Ongoing - global

    M <= A[0]
    N <= 1
    P <= 0
    
    Q <= 1
    
    state <= 3


**State 3 - create hud text**

Rule type: Ongoing - global

    Create hud text ( //detail edit size display
        text: "Current: {Q} M"
        visible: All players
        index: 10
    )
    Create hud text ( //index, position, and size display
        text: "#{P}: {M}, {N}
        visible: filtered array( all players, P < count of(A) )
        index: 10
    )
    Create hud text ( //Add new display
        text: "#{P}: Next
        visible: filtered array( all players, P == count of(A) )
        index: 10
    )
    Create hud text (
        text: "checkpoint on you"
        visible: filtered array( all players, state == 20 )
        index: 11
    )
    create hud text (
        text: "left/right"
        visible: filtered array( all players, state == 30 )
        index: 11
    )
    create hud text (
        text: "forward/back"
        visible: filtered array( all players, state == 31 )
        index: 11
    )
    create hud text (
        text: "up/down"
        visible: filtered array( all players, state == 32 )
        index: 11
    )
    create hud text (
        text: "Sphere"
        visible: filtered array( all players, state == 33 )
        index: 11
    )
    create hud text (
        text: "Starting"
        visible: filtered array( all players, state == 34 )
        index: 11
    )
    create hud text (
        text: "pick checkpoint"
        visible: filtered array( all players, state == 40 )
        index: 11
    )  
    
    state <= 4
    
**state 2 - create node value hud**

Rule type: Ongoing - global

    (repeat x20)
    Create hud text (
        Text: "{index}: {A[{index}]} {B[{index}]}"
        sort: {index}
        visible to: all players
        reevaluate: all
    )
    
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
        position: A[P]
        color: blue
        visible to: All players
        reevaluate: all
    )
    Create effect (
        Type: orb
        position: A[0] + (B[0] * 1.6)
        size: 0.5
        color: white
        visible to: all players
        reevaluate: all
    )
    state <= 20 //edit effect set position


**state 20 and primary fire - place node and player**

    Cond: is pressed(ep, ultimate) == false
    
    M <= pos of( event player )

**state 20 and interact - switch to detail edit mode**

    Y <= 30
    State <= 91

**state 30 and primary fire - move node left**
    
    I <= horizontal facing angle of (ep)
    J <= I + 90
    K <= direction from angles ( J, 0)
    M += K * Q //update node position
    wait .25
    loop if conditions true

**state 30 and secondary fire - move node right**
    
    I <= horizontal facing angle of (ep)
    J <= I - 90
    K <= direction from angles ( J, 0)
    M += K * Q //update node position
    wait .25
    loop if conditions true

**State 30 and ability 1 - switch to forward/back edit**

    Y <= 31
    State <= 91


**state 31 and primary fire - move node foreward**
    
    I <= horizontal facing angle of (ep)
    K <= direction from angles ( I, 0)
    M += K * Q //update node position
    wait .25
    loop if conditions true

**state 31 and primary fire - move node back**

    I <= horizontal facing angle of (ep)
    K <= direction from angles ( I, 0)
    M -= K * Q //update node position
    wait .25
    loop if conditions true

**State 31 and ability 1 - switch to up/down edit**

    Y <= 32
    state <= 92
    
**state 32 and primary fire - move node up**
    
    M += Up * Q
    wait .25
    loop if conditions true

**state 32 and secondary fire - move node down**
    
    M += down * Q
    wait .25
    loop if conditions true

**state 32 and ability 1 and index != 0 - change to size edit**

    Cond: P != 0
    
    Y <= 33
    State <= 91
    
**state 32 and ability 1 and index == 0 - change to initial facing edit**

    Cond: P == 0
    
    Y <= 34
    State <= 91
    
**state 33 and primary fire - increase size**
    
    N += Q
    wait .25
    loop if conditions true

**state 33 and primary fire -decrease size**

    N -= Q
    wait .25
    loop if conditions true

**State 33 and ability 1 - switch to left/right edit**

    State <= 30

**State 33 and ability 1 - switch to left/right edit**

    B[0] <= facing direction of(ep)
    
**state 33 or state 34 and ability 1 - change to X detail edit mode**

    Y <= 30
    state <= 91

**state is >=30 and state <= 34 and interact - change to place mode**

    state <= 20

**state is >= 20 and sate <= 39 and ultimate - go to index change mode**

    state <= 40


**state 40 and primary fire - increase index**

    abort if( P == count of(A) )
    abort if( P == 19 ) //max size
    P += 1


**state 40 and secondary fire - decrease index**

    abort if ( P == 0 )
    P -= 1


**state 40 and ultimate  and P < count of (A) - goto edit existing node**

    Cond: P < count of (A)
    
    Y <= 43
    state <= 91


**state 40 and ultimate  and P == count of (A) - goto append node**

    Cond: P == count of (A)
    
    skip if ( count of (A) < 20 ) {
        play effect( ep, pos of(ep), explosion sound, 200)
        abort
    }
    
    Y <= 44
    state <= 91

**state 40 and interact and P < (count of (A)-1)- goto insert node**

    Cond: P < ( count of (A) - 1 )
    
    skip if (count of (A) < 20 ) {
        play effect( ep, pos of(ep), explosion sound, 200)
        abort
    }
    
    P += 1
    Y <= 45
    state <= 91


**state 40 and interact and P >= (count of (A)-1) - goto append node**

    Cond: P >= ( count of (A) - 1 )
    
    skip if (count of (A) < 20 ) {
        play effect( ep, pos of(ep), explosion sound, 200)
        abort
    }
    
    P <= count of (A)
    Y <= 44
    state <= 91


**state 40 and ability 1 and O < count of (S) - goto delete node**
    
    Cond: P < count of(A)
    
    skip if (P != 0 ) {
        play effect( ep, pos of(ep), explosion sound, 200)
        abort
    }
    
    Y <= 46
    state <= 91


**State 43 - edit existing node**

    M <= A[O]
    N <= B[O]
    state <= 20


**State 44 - append node at end**

    append A <= (0,0,0)
    append B <= 3
    M <= (0,0,0)
    N <= 3
    state <= 20

**State 45 - insert node at index**

    M <= (0,0,0)
    N <= 3
    I <= Slice(A,0,P)
    J <= slice(B,0,P)
    Append I <= (0,0,0)
    Append J <= 3
    Append I <= A[P:]
    Append J <= B[P:]
    A <= I
    B <= J
    state <= 20

**State 46 - delete node at index**

    I <= Slice(A,0,P)
    J <= slice(B,0,P)
    Append I <= A[P+1:]
    Append J <= B[P+1:]
    A <= I
    B <= J
    state <= 40

**state 91 - release all the buttons state**

    Cond: is button held(ep, interact) == false
    Cond: is button held(ep, ultimate) == false
    Cond: is button held(ep, ability 1) == false

    state <= Y

**Keep position array updated**

    Cond: State >= 20
    Cond: State <= 39
    M != A[P]
    
    A[P] <= M
    
**Keep size array updated**

    Cond: State >= 20
    Cond: State <= 39
    P != 0 //don't overwrite initial spawn position value
    N != B[P]
    
    B[P] <= N
    
**Change edit size**

    cond: is button held(ep, jump) == true
    
    Skip if( not( Q == 1 ) ) {
        Q <= 0.3
        abort
    }
    skip if( not( Q == 0.3 ) ) {
        Q <= 0.1
        abort
    }
    Q <= 1


