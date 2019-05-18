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
    E -
    F -
    G -
    H -
    I
    J
    K
    L
    M - Is moderator?
    N
    O - Option array [
        0: skip countdown
        1: mute sounds
    ]
    P - Checkpoint index
    Q - Attempts
    R - Best time array
    S - Last times array
    T - Play time
    U - finishes
    V - 
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
        visble to: filtered array(all players, current:M)
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
        visible to: filtered array(all players, current:M)
        Text: Moderate
        position: C[7] + (up * 3)
        scale: 2
        clipping: clip against surfaces
    )       
    
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
        header: "Records"
        Position: left
        Sort: -10
    )
    Create hud text(
        text: "-------"
        Position: left
        sort: -1
    )
    Create hud text(
        text: "fastest times"
        Postion: left
        sort -2
    )
    create hud text( //reset instructions
        visible to: Filtered array(all players, cu:state >= 30 && cu:state <= 40 )
        text: "use ultimate ability = try again"
        location: top
        sort order: 9
    }
    
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
    
    Wait( random real(0.25, 2) ) //wait random length to avoid server load/crashes
    
    State <= 1
    
**Player state 1 - hud creation part 1 - Top text**

    create hud text( //run timer
        Visible to: event player
        header: "{ep:X} sec"
        location: top
        sort order: 10
    }    

    Create hud text (
        visible to: filtered array(ep, cu:state == 70)
        header: "Moderate -> {A}"
        sort: 12
        Location: top
    )

    Create hud text ( //option select, false
        visible to: filtered array(ep, state == 80) && O[A]
        header: "#{A+1}: off"
        sort: 12
        Location: top
    )
    
    Create hud text ( //option select, true
        visible to: filtered array(ep, state == 80) && O[A]
        header: "#{A+1}: on"
        sort: 12
        Location: top
    )
    
    Wait( random real(0.25, .50) ) //wait random length to avoid server load/crashes
    
    state <= 2
    
**Player state 2 - hud creation part 2 - Statistics**
    
    {Create attempts text - sort -9}
    {Create completions text - sort -8}
    {Create "total game time" text - sort -6}
    
    Wait( random real(0.25, .50) ) //wait random length to avoid server load/crashes
    
    State <= 3 
    
**Player state 3 - hud creation part 3 - Fastest times**
    
    (repeat 0 to 4)
    {Create {index}+1 fastest time text - sort {index}}
    
    Wait( random real(0.25, .50) ) //wait random length to avoid server load/crashes
    
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

    {Make the debug hud with the following visable do condition}
    Filtered array (event player, gR)
    
**Player state 10 - Spawn player and prep waiting state**

    P <= 999
    X <= 0
    
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
    
**Player state 20 and interact pressed - Initialize race**

    Cond: button held(event playser, interact) == true
    
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
    
    wait(10, abort if false)
    small message("unlocking moderator")
    M <= true
    
**player state 20 and in options spot - start options mode**

    Cond: distance between( position of(ep), gC[6]) < 1.5
    
    wait 1 abort
    A <= 0
    State <= 80
    
**player state 20 and in moderator spot - start moderator mode**

    Cond: M == true
    Cond: distance between( position of(ep), gC[7]) < 1.5
    
    wait 1 abort
    A <= 1
    State <= 70

**Player state 20 - teach how to start race**

    Cond: Q <= 3 //stop displaying message once the player has figured out how to start a couple times
    Cond: gQ == false //don't show it in debug mode to avoide cluttering event window
    
    wait 5 abort
    small message("Use ultimate ability = start race")
    loop if true
    
**Player state 21 - Wait for player to stop**

    Cond: speed of(ep) < 0.2
    
    State <= 22
    
**Player state 22 - spawn into race**

    
    Teleport(ep,gA[0]) //race start pos
    Set facing(ep, gC[0]) //race start facing
    P <= 1
    A <= 0.8 //adjust for feel
    skip if( O[0] ) {
        Small message(3)
        Wait(A)
        Small message(2)
        Wait(A)
        Small message(1)
    }
    Wait(A)
    
    Clear status(ep, root)
    Clear status(ep, invincible)
    
    Stop throttle(event player)
    Big message ("Go!")
    skip if( O[1] ) {
        Play effect(ep, buff explosion sound, white, position of(ep), 999)
    }
    
    X <= 0
    Chase player variable(ep, X, 999, 1) // start race timer
  
    State <= 30
    
**Player State 30 - Reach checkpoint**

    Cond: Distance between( pos of(ep), gA[P]) < gB[P] + D[0] )
    
    A <= gA[P]
    B <= gB[P]
    skip if ( O[1] ) {
        Play effect(ep, ring explosion, white, A,  B * 2.5)
        Play effect(ep, explosion sound, white, position of(ep), 80)
    }

    P += 1
    Abort if (P < N)
    State <= 40
    
**Player State 40 - reach goal**

    Cond: Distance between( pos of(Event player), gA[gN]) < gB[gN] + D[0] )
    
    C <= gA[P]
    skip if ( O[1] ) {
        Play effect(ep, ring explosion sound, white, C, 999)
    }
    
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
    
    Wait 1.5 //extra wait on completion, adjust for feel
    
    State <= 90
    
**Player state 70 and primary fire - increase operator action index**

    Cond: A < {total number of moderator actions} //currently 2
    Cond: is pressed(ep,primary fire) == true
    
    A += 1
    
**Player state 70 and secondary fire - decrease operator action index**

    Cond: A > 1
    Cond: is pressed(ep,secondary fire) == true
    
    A -= 1
    
**Player state 70 and interact - run opterator action for index**

    Cond: is pressed(ep,interact) == true
    
    State <= 100 + A
    
**Player state 70 and leave operator spot - go back to wait mode**

    Cond: distance between( position of(ep), gC[7]) > 3
    
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
    
    A += facing angle(ep)*2
    wait(0.016)
    loop if condition is true
    
**player state 200 - move camera backward**

    cond: is button held(ep, secondary fire) == true
    
    A += facing angle(ep)*-2
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
