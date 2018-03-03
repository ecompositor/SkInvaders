This is a Space Invaders clone made using [SkookumScript](http://www.skookumscript.com/) and Unreal Engine 4. The goal of this project is to provide a fully working sample project that illustrates how to write a basic game in UE4 using the SkookumScript programming language. 

This project doesn't have much for visuals, if you're here to learn how to make things look Unreal then you are in the wrong place. However, if you've come to learn SkookumScript then welcome friend!

This project is MIT licensed, if you make a cool clone I'd like to see it!

# Requirements
* UE4 4.18
* SkookumScript Plugin from the UE4 Marketplace

# Architecture
![Component Overview](https://raw.githubusercontent.com/error454/SkInvaders/master/Doc/class%20chart.png)

## Components
### GunComponent
Implements ability to fire classes of `BP_Projectile`. Defines:

* Max # of projectiles allowed on screen at a time
* Delay between shots
* Audio cues for firing sound and gun empty sound
* Spread firing pattern

All of the gun parameters are defined in the struct `S_GunParams` with the idea that the current gun can be overridden by collecting a `BP_Powerup`. The direction and location that the gun fires from is determined by the X direction of the gun component. This is the only component that is a SceneComponent (has a transform).

### HealthComponent
* Tracks current health/max health
* Automatically registers damage handling routines
* Spawns damage and death particles 
* Plays death audio cue.
* Provides death callback to broadcast when its owner has expired. 

Also squirreled away here is the number of points that something is worth when destroyed.

### InvaderMovement
* Provides the typical space invaders movement scheme (right, down, left, down)
* Exposes parameters for speed and distance

Can also be used for things that travel in a straight line like `BP_Saucer` and `BP_Powerup`. Typical use requires knowledge of the game playfield and screen boundaries which are provided by `BP_World`.

### PatternSpawner
* Provides ability to spawn a group of actors from an ASCII defined pattern of 0's and 1's
* Can spawn individual actors like in the case of `BP_Shield` which spawns `BP_ShieldBlock` to create the destroyable shields. Or `BP_Invasion` which spawns groups of `BP_Invader`.
* Can add instanced static mesh instances to create a pattern as you can see in `BP_Invader` and `BP_PlayerPawn`

Patterns are enumerated in `E_Pattern` and defined in the SkookumScript method `Pattern.pattern`. Spawned patterns can be centered about an arbitrary index, for instance in the powerup pattern below, it would be ideal to center it on index 7 so that the pattern will be geometrically centered at the location it is spawned at:

    1 1 1
    1 0 1
    1 1 1
    1 0 0
    1 0 0

## Actors

### BP_Invasion
The coordinator for a group of invaders. The invasion spawns the invaders, groups them together and then moves them down the playfield. It also coordinates:
* Firing amongst the remaining invaders 
* Difficulty adjustment based on how many invaders are remaining and what level you're on
* Spawning saucers
* Determining if invaders have landed

The main entry point here is `_invasion`. 

The actual invasion start/stop is controlled by `GM_Invaders`.

### BP_Invader
The bad guys. They pretty much just stand around and fire once in awhile. They also take damage and die, they are basically puppets controlled by `BP_Invasion`. There are also sub-classes `BP_InvaderType1-3` which are just different spawn patterns.

### BP_Saucer
* Fly's to the right until it reaches its despawn destination
* Defines an array of `BP_Powerup` and chooses one at random to spawn when it is destroyed by the player

### BP_Powerup
* Defines a gun parameter `S_GunParams`
* Overlaps `BP_PlayerPawn` who overrides the `S_GunParams` in her `GunComponent`.

### BP_Shield
* Spawns `BP_ShieldBlock`
* Parameterizes shield class, pattern and scale

### BP_ShieldBlock
One of the simplest classes in the game. Spawned in a pattern to form a shield that blocks projectiles.
* Takes damage thanks to HealthComponent
* Also takes damage when overlapped by `BP_Invader`

### BP_Projectile
Fired by `GunComponent`, damages `HealthComponent`.

The single class in the game that can apply damage to anything it overlaps. This is also the only class that uses physics to move, I had thought of extending this later on to make guided projectiles.

Projectiles have a maximum lifetime after which they self-destruct. They also have a variety of other parameters to tune speed, look and feel.

There is 1 subclass `BP_EnemyProjectile` whose 2 key differences are:
* A different particle system
* A different collision profile on the sphere component

### BP_PlayerPawn
The player pawn. 
* Handles overlaps with `BP_Powerup`
* Handles input and logic for moving and shooting

## Game Logic & Helpers
### BP_WorldSetup
Does all of the heavy lifting for calculating game coordinates:
* Movement boundaries
* Player start
* Screen corners
* Saucer spawn/despawn
* Game Over line
* Invasion spawn point
* Shield spawn points 

Shield, invasion class along with shield count and shield start location are all parameterized. Run the `TEST_WorldBounds` map and eject to see bounds and helpers drawn out. You can probably guess what most of these are.

![World Setup](https://raw.githubusercontent.com/error454/SkInvaders/master/Doc/world%20setup.jpg)

### GM_Invaders
The GameMode that defines the player controller `PC_Invaders` and the default pawn `BP_PlayerPawn`. This is also where the logic for starting, running and restarting the game is found (see `_invaders_game`).

This class is where the majority of the stage direction takes place.

### PC_Invaders
The player controller. 
* Is the primary entry point for everything UI related
* Handles input for the start UI
* Tracks number of player lives
* Tracks player score and high score

## UI
### BP_StartMenu
A simple UI that invites the player to press start. This screen is implemented in `PC_Invaders` `_start_menu` and is called by `GM_Invaders` `_start_game`.

### BP_GameOverMenu
Simply the words Game Over in the center of the screen. This is handled in `PC_Invaders` `show_game_over` and is called by `GM_Invaders` `_handle_win_lose` and `_restart_game`.

### BP_GameUI
The primary game screen which displays:
* Number of player lives
* Score
* Highscore

This UI mostly uses UMG bindings that in-turn call SkookumScript code to perform updates. For instance, the `update_score` method which smoothly increments the player score from previous to current. However, you can also see an example of text being set directly in `PC_Invaders` `_spawn_pawns` where we directly set number of player lives when a player is killed.

# Attributions
* Erbos Draco Monospaced NBP font — Created in 2012 by total FontGeek DTF, Ltd