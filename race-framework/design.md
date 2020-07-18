# Overwatch Race Framework

## Global Variable Definitions


    A - checkpoint positions
    B - checkpoint sizes. B[0] is spawn facing vector instead of size
    
    C - gather room settings [
        0: gather point center
        1: gather point width
        2: gather point length
        3: gather point angle //degrees in world space
        4: leaderboard position
        5: leaderboard scale
        6: options spot
        7: operator spot
    ]
    D - misc settings [
        0: Checkpoint distance adjustment
        1: Team const - Which team does gather room belong to?
        2: Number of capture points
        3: Number of payload checkpoints
    ]
    
    E-H - intermediate values
    
    I
    J
    K
    L - Leaderboard
    M - is top score disconnected?
    N - finish line checkpoint index
    O - 
    P -
    Q - Debug
    R - Enable/disable debug hud
    S - split index array (max size 4)
    T
    U
    V
    W - Server fastest time
    X - Server fastest time splits
    Y - Initialization completed
    Z - Global state machine state

## Player Variable Definitions

    A-D - Main loop intermediate values
    E -
    F -
    G -
    H - 
    I - intermediates and messages [
        0: stats text state (1 for fastests times, 2 for speedrun splits)
    ]
    J - split deltas
    K - current splits
    L - comparison splits
    M - Is moderator?
    N - top text
    O - Option array [
        0: skip countdown
        1: enable split display
        2: user server best splits
    ]
    P - Checkpoint index
    Q - Attempts
    R - Best time array
    S - Best time splits
    T - Play time
    U - finishes
    V - Best time timestamp
    W - Best time
    X - Run time
    Y - records text
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
    append L <= sorted array(
        Array: E
        Sort value: current:W
    )
    M <= false
    skip if (true for any( all players, cu:W != 0 && cu:W <= W) ) {
        M <= true //if best time doesn't belong to any player, set the flag
        E <= L
        L <= append(null, E) //add a blank first entry for disconnected player
    }
    wait 5
    Loop

**Make players phase**

Rule type: Ongoing - Each Player

    Cond: Has status(ep, phased out) == false
    Cond: Q == false //don't run during debug to keep inspector clean
    
    Apply status( Event playser, phased out, 999)
    
**Loading message**

Rule type: Ongoing - Each Player

    Cond: state < 10
    
    small message("wait, loading...")
    wait 2.5
    loop if condition true
    
**Build latest time display**

Rule type: Ongoing - Each Player

    Cond: state >= 10
    Cond: O[1] == false
    
    destroy text(Y[0])
    destroy text(Y[1])
    destroy text(Y[2])
    destroy text(Y[3])
    destroy text(Y[4])
    
    Y <= []
    wait 0.1
    
    (repeat 0 to 4) //there is a wait added between index 3 and 4 to fix misterious issues
    Create hud text(
        visible to: filtered array(ep, R[{index}] != 0)
        Text: "{index+1}: {R[{index}]} sec"
        sort order: {index}
    )
    append Y <= last created text id
    
    
**Build stats display text**

Rule type: Ongoing - Each Player

    Cond: state >= 10
    Cond: (not(O[1]) && I[0] != 1) || (O[1] && I[0] != 2) == true
    
    destroy text(Y[0])
    destroy text(Y[1])
    destroy text(Y[2])
    destroy text(Y[3])
    destroy text(Y[4])
    
    Y <= []
    wait 0.1
    
    skip if ( O[1] ) {
        I[0] <= 1
        (repeat 0 to 4) //there is a wait added between index 3 and 4 to fix misterious issues
        Create hud text(
            visible to: filtered array(ep, R[{index}] != 0)
            Text: "{index+1}: {R[{index}]} sec"
            sort order: {index}
        )
        append Y <= last created text id
        wait 0.1
        loop if condition is true
        abort
    }
    
    I[0] <= 2
    (repeat 0 to 3)
    Create hud text(
        visible to: filtered array(ep, gS[{index}] != 0)
        Text: "{index+1}: {L[index]} /
            {K[index] /
            {filtered array("+ {J[index]}", J[index] > 0)} {filtered array({J[index]}, J[index] <= 0)}"
        sort order: {index}
    )
    append Y <= last created text id
    
    wait 0.1 //wait to fix mysterious issues
    Create hud text(
        visible to: ep
        Text: "{index+1}: {L[4]} /
            {K[4] /
            {filtered array("+ {J[4]}", J[4] > 0)} {filtered array({J[4]}, J[4] <= 0)}"
        sort order: 4
    )
    append Y <= last created text id
    
    wait 0.1
    loop if condition is true
    
## Global State Machine:

All below rules are of rule type "Ongoing - Global" unless otherwise noted.

All below rules have an implied condition of `"gZ == {state in rule name}"`

**Global state 0 - checkpoint position creation**

    
    A <= [{checkpoint positions}]
    State <=1
    
**Global state 1 - checkpoint size creation**

    
    B <= [{checkpoint sizes}]
    State <=2
    
**Global state 2 - gather room config**

    
    C <= [{gather settings}]
    State <= 3
    
**Global state 3 - misc config**

    //If the gather room is in a team's spawn room, that team must be forced to that index. Otherwise, any index will work
    start forcing spawn room(Team 1, {room})
    start forcing spawn room(Team 2, {room})
    
    Q <= {true/false} //debug state
    
    D <= [{config settings}]
    S <= [{split locations}]
    
    State <= 10
    
**Global State 10 - Unlock map**

    Disable built-in game mode completion
    Disable built-in game mode anounce
    

    skip if( D[2] != 1 || D[3] > 0 ) {
        //if there is one capture point and no payload, it is a control map
        state <= 16
        abort
    }
    
    Skip if (D[2] == 0 AND D[3] == 0) {
        //if not control but has objectives, unlock. For Assault, escort, and hybrid.
    	state <= 11
    	Abort
    }
    
    State <= 19
    
**Global state 11 - get in game and prep unlocking**

    
    E <= D[2] //number of capture points
    F <= D[3] //number of payload checkpoints
    
    Set match time(0)
    Wait 1

    Set match time(0)
    Wait 1
        
    State <= 15 // unlocking state
    
**Global state 15 - Freeze players**

Rule type: Ongoing - Each Player

    Cond: Has status(Even player, frozen) == false
    
    Set status(Event player, frozen, 999)
    
**Global state 15 and control points remain - unlock control points**

Rule type: Ongoing - Each Player, Team 2 players

    Cond: objective index < E
    Cond: On objective(Event player) == false
    
    Teleport(Event player, Objective position(objective index)
    Wait .25
    Loop if true
    
**Global state 15 and payload checkpoints remain - advance payload**

Rule type: Ongoing - Each Player, Team 2 players

    Team 2 players
    Cond: objective index >= E
    Cond: objective index < E + F
    Cond: On objective(Event player) == false
    
    Teleport(Event player, Payload position)
    Wait .25
    Loop if true
    
**Global state 15 and no payload points remain - complete map unlock**

    cond: objective index == E + F
    
    Clear status(Frozen, all players)
    
    State <= 19
    
**Global state 16 - unlock control map**

    Set match time(0)
    Wait 1

    Set match time(0)
    Wait 1
        
    State <= 19
    
**Global state 19 - complete game set up**

    
    L <= [] //empty leaderboard
    W <= 999 //starting best time
    R <= Q //default debug hud toggle to debug state
    Pause match time
    Set match time (671) //display "11:11" because it looks nice
    skip if ( D[2] != 1 && D[3] != 0 ) {
        set match time(1111) //control maps display in seconds
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
    	Text: "High Scores"
    	Position: E
    	Scale: {high scores text size} * I
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

    
    //create two versions of the first entry. One normal, and one if the top score has left the game
    Create in-world text( //normal version
        Visible to: filtered array ( All Players, !M && L[0]:W != 0)
        Text: "{L[{index}]} - {L{index}]:W} Sec"
        Position: E
        Scale: H
        Clipping: Yes
        Reevaluation: Visible to and string
    )
    Create in-world text( //disconcted best score version
        Visible to: filtered array ( All Players, M)
        Text: "Disconnected player - {W} Sec"
        Position: E
        Scale: H
        Clipping: Yes
        Reevaluation: Visible to and string
    )
    E <= E + (down * G)
    (repeat 1 to 4)
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

    Create effect(
        visible to: All players
        type: sphere
        color: yellow
        position: A[N]
        radius: B[N]
    )
        
    Create icon(
        visible to: filtered array(all players, cu:P == N)
        position: A[N]
        icon: exclamation mark
    )
    
    State <= 31
    
**Global state 31 - Create options and moderator zones**

    Create effect(
        visble to: All players
        type: ring
        color: green
        position: C[6]
        size: 1.5
    )
    Create effect(
        visble to: filtered array(host player, ce:M)
        type: ring
        color: red
        position: C[7]
        size: 1.5
    )
    Create in-world text(
        visible to: All players
        Text: Optimize
        position: C[6] + (up * 3)
        scale: 2
        clipping: clip against surfaces
    )
    Create in-world text(
        visble to: filtered array(host player, ce:M)
        Text: Moderate
        position: C[7] + (up * 3)
        scale: 2
        clipping: clip against surfaces
    )
    
    state <= 32
    
**Global state 32 - Create split checkpoints**

    (repeat 0 to 3)
    create effect (
        visible to: filtered array( all players, cu:O[1] == true && cu:P == S[index] )
        type: sphere
        color: blue
        position: A[S[index]]
        radius: B[S[index]]
    )
    State <= 40
    
**Global state 40 - Create white checkpoint 1-4**

    (repeat 1 to 4)
    create effect (
        visible to: filtered array( all players, cu:P < {index} && {index} < N )
        type: sphere
        color: white
        position: A[{index}]
        radius: B[{index}]
    )
    State <= 41
    
**Global state 41 - Create white checkpoint 5-9**

    
    (repeat 5 to 9)
    create effect (
        visible to: filtered array( all players, cu:P < {index} && {index} < N )
        type: sphere
        color: white
        position: A[{index}]
        radius: B[{index}]
    )
    State <= 42
    
**Global state 42 - Create white checkpoint 10-14**

    
    (repeat 10 to 14)
    create effect (
        visible to: filtered array( all players, cu:P < {index} && {index} < N )
        type: sphere
        color: white
        position: A[{index}]
        radius: B[{index}]
    )
    State <= 43
    
**Global state 43 - Create white checkpoint 15-19**
    
    (repeat 15 to 19)
    create effect (
        visible to: filtered array( all players, cu:P < {index} && {index} < N )
        type: sphere
        color: white
        position: A[{index}]
        radius: B[{index}]
    )
    
    State <= 50
    
**Global state 50 - Create static hud text**

    Create hud text(
        header: "STATS"
        Position: left
        Sort: -10
    )
    Create hud text(
        text: "-------"
        Position: left
        sort: -1
    )
    Create hud text(
        text: "Server uptime: {Total Time Elapsed}"
        Position: left
        sort: -1
    )
    Create hud text(
        visible to: filtered array(all players, cu:O[1] == false)
        text: "Best 5 times:"
        Postion: left
        sort -2
    )
    Create hud text(
        visible to: filtered array(all players, cu:O[1] == true)
        text: "Best Split / Current Split / Difference
        Postion: left
        sort -2
    )
    create hud text( //reset instructions
        visible to: All Players
        text: "Press ultimate(Q) to restart"
        location: top
        sort order: 9
    }
    Create hud text(
        visible to: filtered array(host player, gR == true)
        text: Load: {server load} Avg: {server avg load} Max: {Sever max load}
        location:right
        sort order: 10
    )
    Create hud text(
        visible to: filtered array(all players, ce:state >= 80 && ce:state <=89)
        text: "left/right click to select option, press interact(F) to toggle"
        location: top
        sort order:11
    )
    
    State <= 90
    
**Global state 90 - complete global init**

    Cond: is game in progress == true
    
    Y <= true //init complete
    State <= 999
    
**Global state 999 - Wait state (no actual rule)**

## Player State Machine

All below rules are of rule type "Ongoing - Each Player" unless otherwise noted.

All below rules have an implied condition of `"ep:Z == {state in rule name}"`

**Player state 0 and global initialization complete - Player initialization**

    Cond: gY == true //global init complete
    
    Disable built in mode respawning(event player)
    Set respawn max time(ep, 99) //long respawn
    Disallow button(ep, ultimate) //disallow ultimate
        
    P <= 999
    
    O <= [false, false]
    Q <= 0 //init attempts
    R <= [] //empty best times array
    S <= [] // empty last times array
    T <= 0 //init total play time timer
    Chase player variable at rate(T, 10000, 1/s) //play time counter
    U <= 0 // init completions
    W <= 999 //initial best time
    H <= [1,0,null,null]
    I <= [0]
    
    Wait( random real(0.25, 2) ) //wait random length to avoid server load/crashes
    
    State <= 1
    
**Player state 1 - Create player hud**

    create hud text( //run timer
        Visible to: event player
        header: "{ep:X} sec"
        location: top
        sort order: 10
    }
    
    {Create attempts text - sort -9}
    {Create completions text - sort -8}
    {Create "total game time" text - sort -6}
    
    wait 0.1 //needed to avoid glitches with stats creation
    
    state <= 2
    
**Player state 2 - Create effects**

    Create effect (
        Visible to: filtered array (ep, P < gN && not(O[1] && array contains(gS,P)) )
        Type: Sphere
        Color: blue
        Position: gA[P]
        Radius: gB[P]
        Reevaluation ( visible to, position, and scale)
    )
    Create icon (
        Visible to: filtered array (ep, P < gN)
        Type: asteric
        Color: white
        Position: gA[P]
        Reevaluation ( visible to, position, and scale)
    )
    
    state <= 9

    
**Player state 9 - finished loading**

    Big Message(ep, "Finished Loading")
    
    State <= 10
    
**Player state 10 - Spawn player and prep waiting state**

    P <= 999
    
    H[0] <= 1 + (O[1] && array contains(gS,P))
    
    stop forcing throttle(ep)
    Set status(ep,invincible,9999)
    
    State <= 11
    
**Player state 11 - Wait for player to be alive**

    Cond: is alive(ep) == true
    Cond: is in spawn room(ep) == true
    
    State <= 15
    
**Player state 15 - Teleport to gather room**

    //Generate a position in world space inside the rotated rectangle
    //defined in the gather room settings array.
    A <= gC[1]
    C <= random real(-A,A)
    A <= gC[2]
    D <= random real(-A,A)
    A <= Direction from angles(gC[3] + 90, 0) * C
    B <= Direction from angles(gC[3], 0) * D
    D <= gC[0] + A + B
    
    teleport(ep,D) //teleport to spot
    
    Set facing ( //look towards leaderboard along level plane
        player: ep
        direction:
            direction from angles(
                horizontal angle: horizontal angle from direction ( direction towards(D, C[4]) )
                vertical angle: 0
            )
    )
    
    State <= 20
    
**Player state 20 and ultimate pressed - Initialize race**

    Cond: button held(event playser, ultimate) == true
    
    set status(ep, root, 9999)
    wait .25
    State <= 21 //start spawn in process
    
**Player state 20 and left spawn - Return player to gather point**

    Cond: Team of(event player) == D[1] //Gather room belongs to player, so the doors are open and they count as in spawn
    Cond: In spawn(event player) == false
    Cond: gQ == false // don't force it in debug
    
    State <= 15

**player state 20 and button combo - enable moderator**

Rule type: ongoing - each player, team 2, slot 0

    cond: is button held(ep,interact) == true
    cond: is button held(ep,crouch) == true
    cond: event player == host player
    
    wait(10, abort if false)
    small message("unlocking moderator")
    M <= true
    
**player state 20 and in options spot - start options mode**

    Cond: distance between( position of(ep), gC[6]) < 1.5
    
    wait 1 abort
    A <= 0
    Create hud text ( //option select, false
        visible to: ep
        header: "optomize {A+1}: O[A]"
        sort: 12
        Location: top
    )
    N <= last created text id
    State <= 80
    
**player state 20 and in moderator spot - start moderator mode**

    Cond: Event Player == Host Player
    Cond: distance between( position of(ep), gC[7]) < 1.5
    
    wait 1 abort
    A <= 1
    Create hud text ( //moderator text
        visible to: event player
        header: "Moderate -> {A}"
        sort: 12
        Location: top
    )
    N <= last created text id
    State <= 70

**Player state 20 - teach how to start race**

    Cond: Q < 3 //stop displaying message once the player has figured out how to start a couple times
    Cond: gQ == false //don't show it in debug mode to avoide cluttering event window
    
    wait 5 abort
    small message("Press Ultimate(Q) to start!")
    loop if true
    
**Player state 21 - Wait for player to stop**

    Cond: speed of(ep) < 0.2
    
    State <= 22
    
**Player state 22 - spawn into race**
    
    L <= S // set to comparison splits to personal best
    skip if( not( O[2] ) ) {
        L <= gX //set comparison to server best if option is set
    }
    
    X <= 0
    I <= [W,0,0,0]
    K <= []
    J <= []
    P <= 1
    
    Teleport(ep,gA[0]) //race start pos
    Set facing(ep, gC[0]) //race start facing
    
    A <= 0.8 //adjust for feel
    skip if( O[0] ) {
        Small message(3)
        play effect(ep, buff explostion sound, white, position of(ep), 35)
        Wait(A)
        Small message(2)
        play effect(ep, buff explostion sound, white, position of(ep), 35)
        Wait(A)
        Small message(1)
        play effect(ep, buff explostion sound, white, position of(ep), 35)
    }
    Wait(A)
    
    Clear status(ep, root)
    Clear status(ep, invincible)
    
    Stop throttle(event player)
    Big message ("Go!")
    Play effect(ep, buff explosion sound, white, position of(ep), 999)
    
    Chase player variable(ep, X, 999, 1) // start race timer
  
    State <= 30
    
**Player State 30 - Reach checkpoint**

    Cond: Distance between( pos of(ep), gA[P]) < gB[P] + D[0] )
    
    A <= gA[P]
    B <= gB[P]
    Play effect(ep, ring explosion, white, A,  B * 2.5)
    Play effect(ep, explosion sound, white, position of(ep), 80)

    skip if( not( is true for any( gS, P == cu) ) ) {
        append K <= X
        append J last_of(K) - L[ index of value(gS,P) ]
    }
    
    P += 1
    H[0] <= 1 + (O[1] && array contains(gS,P))
    
    Abort if (P < gN)
    State <= 40
    
**Player State 40 - reach goal**

    Cond: Distance between( pos of(Event player), gA[gN]) < gB[gN] + D[0] )
    
    C <= gA[P]
    Play effect(ep, ring explosion sound, white, C, 999)
    
    P <= 999
    State <= 50
    
**Player state 50 - Start finish race**

    Stop chasing player variable(ep, X) // stop race timer
    
    K[4] <= X
    J[4] <= X - L[4]
    
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
    S <= K //update splits
    Skip if ( W >= gW ) { //skip if not new server record
    	gW <= W
        gX <= S
    	Create big message ("New High Score: {event player} - {W} Sec")
    	State <= 60
    	Abort
    }
    Create small message "g{event player} new record - {W} Sec")
    State <= 60
    
**Player state 60 - Update stats and end race**

    U += 1 //increment finishes
    A <= append(R, x) // put best 5 times and newest time into intermediate
    B <= sorted array(A) //Sort all 6
    R <= B[0:4] //Save the best 5
    
    Wait 1.5 //extra wait on completion, adjust for feel
    
    State <= 90
    
**Player state 70 and primary fire - increase operator action index**

    Cond: A < {total number of moderator actions} //currently 3
    Cond: is pressed(ep,primary fire) == true
    
    A += 1
    
**Player state 70 and secondary fire - decrease operator action index**

    Cond: A > 1
    Cond: is pressed(ep,secondary fire) == true
    
    A -= 1
    
**Player state 70 and interact - run opterator action for index**

    Cond: is pressed(ep,interact) == true
    
    destory text(N)
    State <= 100 + A
    
**Player state 70 and leave operator spot - go back to wait mode**

    Cond: distance between( position of(ep), gC[7]) > 3
    
    destory text(N)
    State <= 20

**Player state 80 and primary fire - increase option index**

    Cond: A < count of(O) - 1
    Cond: is pressed(ep,primary fire) == true
    
    A += 1
    
**Player state 80 and secondary fire - decrease option index**

    Cond: A > 0
    Cond: is pressed(ep,secondary fire) == true
    
    A -= 1
    
**Player state 80 and interact - toggle option**

    Cond: is pressed(ep,interact) == true
    
    B <= not( O[A] )
    O[A] <= B
    
**Player state 80 and leave options spot - go back to wait mode**

    Cond: distance between( position of(ep), gC[6]) > 3

    destory text(N)
    State <= 20
    
**Player state 90 - Start reset sequence**

    Stop chasing player variable(ep, X)
    Start forcing throttle(Event player, stop)
    Q += 1 // increase attempt counter
    
    Wait 1.5 //wait during reset, adjust for feel
    
    State <= 10
    
    
**player state 101 - operator action 1 - greet server**

    Cond: is button held(ep, interact) == false
    
    small message("Hello players! Thanks 4 joining!")
    wait 1
    state <= 20

    
**player state 102 - operator action 2 - toggle debug hud**

    Cond: is button held(ep, interact) == false
    
    //rule currently non-functional due to removed debug text
    gR <= not( gR ) //toggle debug text
    state <= 20
    
**player state 103 - operator action 3 - start camera**

    Cond: is button held(ep, interact) == false

    A <= gA[0]
    start camera(A, A + facing angle(ep),80)
    start forcing throttle(ep, stop)
    
    P <= 0
    
    state <= 200
    
**player state 200 - move camera forward**

    cond: is button held(ep, primary fire) == true
    
    A += facing angle(ep) * 0.5
    wait(0.016)
    loop if condition is true
    
**player state 200 - move camera backward**

    cond: is button held(ep, secondary fire) == true
    
    A += facing angle(ep) * -0.5
    wait(0.016)
    loop if condition is true
    
**player state 200 - advance checkpoint display**

    cond: is button held(ep, crouch) == true
    
    P = (P + 1) % gN
    
**player state 200 - end camera mode**

    cond: is button held(ep, interact) == true
    
    stop camear(ep)
    stop forcing throttle(ep)
    P <= 999
    
    state <= 20
