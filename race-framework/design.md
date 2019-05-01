# Overwatch Race Framework

## Global Variable Definitions


    A - checkpoint positions
    B - checkpoint sizes
    
    C - spawn room settings [
        0: Race start facing
        1: gather point
        2: gather point size
        3: gather spawn facing
        4: Leaderboard position
        5: Leaderboard scale
        6: poll option 1 position
        7: poll option 2 position
        8: moderator spot
    ]
    D - misc settings [
        0: Checkpoint distance adjustment
        1: Team const - which team to teleport
        2: Number of capture points
        3: Number of payload checkpoints
        4: Floor is lava mode?
        5: Number of ground touches allowed
        6: Ground time allowed
    ]
    
    E-H - intermediate values
    
    I
    J
    K
    L - Leaderboard
    M
    N - finish line checkpoint index
    O
    P - code table (P for puzzle)
    Q - Debug
    R - Enable/disable debug hud
    S
    T
    U
    V
    W - Fastest time
    X -
    Y - Initialization completed
    Z - Global state machine state

## Player Variable Definitions

    A-D - Main loop intermediate values
    E - Edit mode adjustment scale
    F - Poll mode vote choice
    G - Ground touch counter
    H - Ground touch start time
    I
    J
    K
    L
    M
    N
    O
    P - Checkpoint index
    Q - Attempts
    R - Best time array
    S - Last times
    T - Play time
    U - finishes
    V - available, should use for a statistic
    W - Best time
    X - Run time
    Y -
    Z - Main loop state

## Miscellaneous rules

**Keep goal index updated**
Rule type: Ongoing - Global

    Cond: N != Count of(A) - 1
    N <= count of(A) -1

**Player killed event - Reset match**
Rule type: Player Died

    State <= 90

  

**Ultimate pressed - quick reset**
Rule type: Ongoing - Each Player

    Cond: State >= 30
    Cond: Player state <= 49 // in race states
    Cond: Button held( Event player, Ultimate) == true

    Kill(ep)

**Rebuild Leaderboard loop**
Rule type: Ongoing - Global

    Cond: Y == true //init complete
    Cond: Q == false //don't run during debug to keep inspector clean

    E <= filtered array(
	    Array: all players
	    Condition: current:W != 0 && current:W != 999
    )
    L <= sorted array(
	    Array: E
	    Sort value: current:W
    )
    Wait 5
    Loop

  

**Make players phase**
Rule type: Ongoing - Each Player

    Cond: Has status(ep, phased out) == false
    Cond: Q == false //don't run during debug to keep inspector clean
    
    Apply status( Event playser, phased out, 999)

## Global State Machine:
All below rules are of rule type "Ongoing - Global" unless otherwise noted.

All below rules have an implied condition of `"gZ == {state in rule name}"`

**Global state 0 - checkpoint position creation**

    
    A <= [{checkpoint positions}]
    State <=1
    
**Global state 1 - checkpoint size creation**

    
    B <= [{checkpoint sizes}]
    State <=2
    
**Global state 2 - spawn room config**

    
    C <= [{spawn settings}]
    State <= 3
    
**Global state 3 - misc config**

    
    Q <= {true/false} //debug state
    Start forcing spawn room(team 1, {room})
    Start forcing spawn room(team 2, {room})
    
    D <= [{config settings}]
    State <= 10
    
**Global State 10 - Unlock map**

    
    Skip if (D[2] == 0 AND D[3] == 0) { //skip if no capture points for payload, so skirmish maps
    	state <= 11
    	Abort
    }
    
    State <= 19
    
**Global state 11 - get in game and prep unlocking**

    
    E <= D[2] //number of capture points
    F <= D[3] //number of payload checkpoints
    
    Create hud text "UNLOCKING CHECKPOINT AND PAYLOAD"
    G <= last text id
    Create hud text "FIND TEAM 2 PLAYERS" text visible when no one on team 2
    H <= last text id
    
    Set match time(0)
    Wait 1
    Set match time(0)
    Wait 1
    
    Disable built-in completion
    
    State <= 15 // unlocking state
    
**Global state 15 - Freeze players**
Rule type: Ongoing - Each Player

    Cond: Has status(Even player, frozen) == false
    
    Set status(Event player, frozen, 999)
    
    Global state 15 and control points remain - unlock control points
    Rule type: Ongoing - Each Player //NOTE THE ATYPICAL RULE TYPE
    Team 2 players
    Cond: objective index < E
    Cond: On objective(Event player) == false
    
    Teleport(Event player, Objective position(objective index)
    Wait .25
    Loop if true
    
**Global state 15 and payload checkpoints remain - advance payload**
Rule type: Ongoing - Each Player

    Team 2 players
    Cond: objective index >= E
    Cond: objective index < E + F
    Cond: On objective(Event player) == false
    
    Teleport(Event player, Payload position)
    Wait .25
    Loop if true
    
**Global state 15 and no payload points remain - complete map unlock**

    cond: objective index == E + F
    
    Destroy text(G)
    Destroy text(H)
    Clear status(Frozen, all players)
    
    State <= 19
    
**Global state 19 - complete game set up**

    
    L <= [] //empty leaderboard
    W <= 999 //starting best time
    R <= Q //default debug hud toggle to debug state
    Pause match time
    Set match time 11:11 (671 seconds)
    Set objective description ("go fast!")
    
    State <= 20
    
**Global state 20 - Leaderboard set up part 1**

    
    E <= C[4] // Leaderboard start position vector
    I <= C[5] //scale
    F <= {header to first entry gap} * I
    G <= {entry gap} * I
    H <= {entry size} * I
    
    Create in-World text(
    	Visible to: All
    	Text: "Leaders"
    	Position: E
    	Scale: {Leaders text size} * I
    	Clipping: Yes
    	Reevaluation: Visible to and String
    )
    E += (down * F)
    Create in-World text(
    	Visible to: All
    	Text: "-------------"
    	Position: E
    	Scale: {dash text size} * I
    	Clipping: Yes
    	Reevaluation: Visible to and String
    )
    E += (down * F)
    
    State <= 21
    
**Global state 21 - Leaderboard set up part 2**

    
    (repeat x5)
    Create in-world text(
    	Visible to: filtered array ( All Players, L[{index}]:W != 0)
    	Text: "{L[{index}]} - {L{index}]:W} Sec"
    	Position: E
    	Scale: H
    	Clipping: Yes
    	Reevaluation: Visible to and string
    )	
    E <= E + (down * G)
    
    State <= 30
    
**Global state 30 - Create finish checkpoint and icon**

    
    [create goal effect]
    [create goal icon]
    
    State <= 40
    
**Global state 40 - Create white checkpoint 2-4**

    
    [Create effects]
    State <= 41
    
**Global state 41 - Create white checkpoint 5-9**

    
    [Create effects]
    State <= 42
    
**Global state 42 - Create white checkpoint 10-14**

    
    [Create effects]
    State <= 43
    
**Global state 43 - Create white checkpoint 15-19**

    
    [Create effects]
    State <= 90
    
**Global state 90 - complete global init**

    Cond: is game in progress == true
    
    Y <= true //init complete
    State <= 999
    
    Global state 999 - Wait state (no actual rule)

## Player State Machine

All below rules are of rule type "Ongoing - Each Player" unless otherwise noted.

All below rules have an implied condition of `"ep:Z == {state in rule name}"`

**Player state 0 and global initialization complete - Player initialization**

    Cond: gY == true //global init complete
    
    Disable built in mode respawning(event player) //this doesnft even work
    Set respawn max time(ep, 99) //long respawn
    Start forcing throttle(ep, stop) //stop player during init
    Disallow button(ep, ultimate) //disallow ultimate
    
    E <= 1 //start edit adjust at max size
    
    P <= 999
    
    Q <= 0 //init attempts
    R <= [] //empty best times array
    S <= [] // empty last times array
    T <= 0 //init total play time timer
    Chase player variable at rate(T, 5999, 1/s) //chase to max of 99:59
    U <= 0 // init completions
    W <= 999 //initial best time
    
    State <= 1
    
**Player state 1 - hud creation part 1 - Top text**

    
    Create reset instructions on top - sort 9
    Create second display on top - sort 10
    
    Create start instructions on top - sort 11
    Visible to: Filtered array(Event player, current:state == 20 )
    
**Player state 1 - hud creation extra - moderator hud**

Rule type: Ongoing - Each player, team 2, slot 1

    Create hud text (
        Text: "Mode: {A}" //If I can make it something like "command" or something better do
        sort: 12
        visible to: filtered array(even player, cu:state == 70)
        Location: top
    )
    
**Player state 2 - hud creation part 2 - Statistics**

    
    Create records header on left - sort -10
    Create attempts text - sort -9
    Create completions text - sort -8
    Create play time text - sort -7
    
    State <= 3 
    
**Player state 3 - hud creation part 3 - Fastest times**

    
    Create "-------" text - sort -2
    Create fastest times text - sort -1
    
    (repeat x5)
    Create 1st fastest time text - sort 0-4
    
    State <= 4
    
**Player state 4 - hud creation part 4 - Latest times**

    
    Create "-------" text - sort 8
    Create "times" text - sort 9
    (repeat x5)
    Create 1st most recent completion time - sort 10-14
    
    State <= 5
    
**Player state 5 - hud creation part 5 - Checkpoint display**

    
    Create effect (
    Visible to: filtered array (ep, P < gN)
    Type: Sphere
    Color: Purple
    Position: gA[P]
    Radius: gB[P]
    Reevaluation ( visible to, position, and scale)
    )
    Create icon (
    Visible to: filtered array (ep, P < gN)
    Type: diamond
    Color: white
    Position: gA[P]
    Reevaluation ( visible to, position, and scale)
    )
    
    State <= 9
    
**Player state 9 - Debug hud creation**


    [Make the debug hud with the following visable do condition]
    Filtered array (event player, gR == true)
    
**Player state 10 - Spawn player and prep waiting state**

    
    Respawn(ep)
    P <= 999
    X <= 0
    Set status(ep,invincible,9999)
    Stop forcing throttle(ep)
    
    State <= 11
    
**Player state 11 - Wait for player to be alive**

    Cond: is alive(ep) == true
    Cond: is in spawn room(ep) == true
    
    Skip if ( team of(ep) != gD[1] ) {
    	//ep in teleport group
    	State <= 15
    	Abort
    }
    State <= 20
    
**Player state 15 - Teleport to gather room**

    
    A <= gC[2]
    B <= random real(-A,A)
    C <= random real(-A,A)
    D <= gC[1] + (B,0,C)
    teleport(ep,D)
    Set facing(ep,gC[3])
    
    State <= 20
    
**Player state 20 and interact pressed - Initialize race**

    Cond: button held(event playser, interact) == true
    
    throttle(event player, stop)
    apply status(ep, stun, 9999)
    wait .25
    State <= 21 //start spawn in process
    
**Player state 20 and left spawn - Return player to gather point**

    Cond: Team of(event player) != D[1] //player not in teleport group and spawn doors open
    Cond: In spawn(event player) == false
    Cond: gQ == false // don't force it in debug
    
    State <= 15
    
**Player state 20 and in moderator spot - Start moderator mode**

Rule type: Ongoing - each player, team 2, slot 1

    Cond: distance between( position of(ep), gC[8]) < 1.5 
    
    A <= 1
    State <= 70
    
**Player state 21 - Wait for player to stop**

    Cond: speed of(ep) < 0.2
    
    State <= 22
    
**Player state 22 - spawn into race**

    
    Teleport(ep,gA[0]) //race start pos
    Set facing(ep, gC[0]) //race start facing
    P <= 1
    A <= 0.7 //adjust for feel
    Big message(3)
    Wait(A)
    Big message(2)
    Wait(A)
    Big message(1)
    Wait(A)
    
    Clear status(ep, stun)
    Clear status(ep, invincible)
    
    Stop throttle(event player)
    Big message ("Go!")
    
    X <= 0
    Chase player variable(ep, X, 999, 1) // start race timer
  
    State <= 30
    
**Player State 30 - Reach checkpoint**

    Cond: Distance between( pos of(ep), gA[P]) < gB[P] + D[0] )
    
    P += 1
    Abort if (P < N)
    State <= 40
    
**Player State 40 - reach goal**

    Cond: Distance between( pos of(Event player), gA[gN]) < gB[gN] + D[0] )
    
    P <= 999
    State <= 50
    
**Player state 50 - Start finish race**

    
    Stop chasing player variable(ep, X) // stop race timer
    throttle(ep,stop)
    U += 1 //update completion count
    
    Skip if ( X >= W) { // skip if not new best
    	State <= 51
    	Abort
    }
    Small message("Time - {X} Sec")
    State <= 60
    
**Player state 51- Update records**

    
    W <= X //update personal best
    Skip if ( W >= gW ) { //skip if not new server record
    	gW <= W
    	Create big message ("New High Score: {event player} - {W} Sec")
    	State <= 60
    	Abort
    }
    Create small message "g{event player} new record - {W} Sec")
    State <= 60
    
**Player state 60 - Update stats and end race**

    
    U += 1 //increment finishes
    A <= S[1:4] //Drop oldest time
    S <= append(S, A) //Add newest time
    A <= append(R, x) // put best 5 times and newest time into intermediate
    B <= sorted array(A) //Sort all 6
    R <= B[0:4] //Save the best 5
    
    Wait 2 //extra wait on completion, adjust for feel
    
    State <= 90
    
**Player state 70 and primary fire - increase moderator action index**

    Cond: A < {total number of moderator actions} //currently 3 total
    Cond: is pressed(ep,primary fire) == true
    
    A += 1
    
**Player state 70 and secondary fire - decrease moderator action index**

    Cond: A > 1
    Cond: is pressed(ep,secondary fire) == true
    
    A -= 1
    
**Player state 70 and interact - run moderator action**

    Cond: is pressed(ep,interact) == true
    
    State <= 100 + (A-1)
    
**Player state 70 and leave moderator spot - go back to wait mode**

    Cond: distance between( position of(ep), gC[8]) > 1.5
    
    State <= 20
    
    
**Player state 90 - Start reset sequence**

    Stop chasing player variable(ep, X)
    Throttle(Event player, stop)
    Q += 1 // increase attempt counter
    
    Wait 1 //wait during reset, adjust for feel
    
    State <= 10
    
    
**Player state 100 - test moderator action 1**

    small message("something useful or interesting or funny")
    wait 1
    state <= 20

**Player state 101 - test moderator action 2**

    small message("Something useful or interesting or funny 2")
    wait 1
    state <= 20
    
**Player state 101 - test moderator action 2**

    gR <= not( gR ) //toggle debug text
    wait 1
    state <= 20

