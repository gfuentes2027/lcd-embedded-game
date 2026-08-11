# Project Overview

 This is an embedded obstacle dodging game for the STM32F401RE Arm Cortex-M4 microcontroller. The goal of the game is for the player to last as long as possible while dodging objects that are flying towards them. The player moves between the top and bottom rows of the LCD by pressing a pushbutton as the obstacles move in a random order from left to right. 

**Watch the video demo below**

[![Watch the STM32 game demo](https://img.youtube.com/vi/L1NryUr7NEQ/hqdefault.jpg)](https://youtu.be/L1NryUr7NEQ)

 # Hardware

| Component | STM32 Connection |
|-----------|------------------|
|LCD data bus| PC0-PC7, 8-bit parallel|
|LCD RS | PA5|
|LCD RW | PA6|
|LCD Enable | PA7|
|Pushbutton| PA0, active-low|
|Target| STM32F401RETx, Cortex-M4|
Clock| Default 16 MHz HSI|

![STM32 LCD game wiring schematic](349_project_hardware.svg)

# Software Architecture

```mermaid
flowchart TB
    CUBE["STM32CubeMX / CMSIS<br/>startup and device support"]
    RESET["Reset Handler"]

    subgraph APP["Application — 349_project_code.s"]
        MAIN["__main<br/>Application entry"]
        INIT["System Initialization<br/>LCD • GPIO • custom characters • game state"]
        LOOP["Main Game Loop"]

        COLLISION["Collision Detection"]
        OBSTACLES["Obstacle Manager<br/>spawn • animate • move"]
        INPUT["Button Handler<br/>edge detection • debounce"]
        PLAYER["Player Manager<br/>toggle row • redraw"]
        SCORE["Score Manager"]
        GAMEOVER["Game-Over State<br/>display score • stop gameplay"]

        STATE[("Game State<br/>score • random seed • delays<br/>active flags • positions • frames")]
    end

    subgraph SERVICES["Drivers and Utilities"]
        RNG["Random Generator<br/>linear congruential generator"]
        LCDDRIVER["LCD Driver<br/>LCDInit • LCDCommand • LCDData"]
        DELAY["Busy-Wait Timing"]
        GPIO["Memory-Mapped GPIO Access"]
    end

    subgraph HARDWARE["Hardware"]
        BUTTON["Active-Low Pushbutton<br/>PA0"]
        LCD["16×2 Character LCD<br/>PA5–PA7 and PC0–PC7"]
        MCU["STM32F401RE<br/>Arm Cortex-M4"]
    end

    CUBE --> RESET --> MAIN --> INIT --> LOOP

    LOOP --> COLLISION
    COLLISION -->|No collision| OBSTACLES
    COLLISION -->|Collision| GAMEOVER
    OBSTACLES --> INPUT --> PLAYER --> SCORE --> DELAY --> LOOP

    COLLISION <--> STATE
    OBSTACLES <--> STATE
    INPUT <--> STATE
    PLAYER <--> STATE
    SCORE <--> STATE

    OBSTACLES --> RNG
    INIT --> LCDDRIVER
    OBSTACLES --> LCDDRIVER
    PLAYER --> LCDDRIVER
    GAMEOVER --> LCDDRIVER

    BUTTON --> GPIO
    INPUT --> GPIO
    LCDDRIVER --> GPIO
    GPIO --> MCU
    GPIO --> LCD
```

# How The Game Works
1. Check whether the player and an active obstacle collide at column 15.
2. Decrement the top and bottom obstacle spawn timers.
3. Draw or move active obstacles.
4. Read PA0 and toggle the player’s row on a new button press.
5. Advance each obstacle’s animation frame and position.
6. Clear and redraw the player.
7. Increment the score.
8. Wait using a busy-loop delay and repeat.

When a collision occurs, the program clears the LCD and prints "GAME OVER!" along with a three-character score.

