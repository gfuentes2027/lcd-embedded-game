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

![STM32 LCD game wiring schematic](docs/349_project_hardware.svg)

# Software Architecture

  ```mermaid
  ---
  config:
    htmlLabels: false
    flowchart:
      wrappingWidth: 180
      padding: 15
  ---
  flowchart TB
      CUBE["`STM32CubeMX / CMSIS
      Startup and device support`"]

      RESET["Reset Handler"]

      subgraph APP["Application - 349_project_code.s"]
          MAIN["`Assembly entry point
          __main`"]

          INIT["`System Initialization
          LCD, GPIO, RNG, and game state`"]

          LOOP["Main Game Loop"]
          COLLISION["Collision Detection"]

          OBSTACLES["`Obstacle Manager
          Spawn, animate, and move`"]

          INPUT["`Button Handler
          Edge detection and debounce`"]

          PLAYER["`Player Manager
          Toggle row and redraw`"]

          GAMEOVER["`Game-Over State
          Display score and stop gameplay`"]

          STATE[("`Game State
          Score, seed, delays, flags,
          positions, and animation frames`")]
      end

      subgraph SERVICES["Drivers and Utilities"]
          RNG["`Random Generator
          Linear congruential generator`"]

          LCDDRIVER["`LCD Driver
          Init, command, and data`"]

          DELAY["Busy-Wait Timing"]
          GPIO["Memory-Mapped GPIO"]
      end

      subgraph HARDWARE["Hardware"]
          BUTTON["`Active-Low Pushbutton
          PA0`"]

          LCD["`16x2 Character LCD
          PA5-PA7 and PC0-PC7`"]

          MCU["`STM32F401RE
          Arm Cortex-M4`"]
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

# Register-Level Design
The game uses direct memory-mapped register access rather than high-level GPIO functions:

- `RCC_AHB1ENR` enables the GPIOA and GPIOC peripheral clocks.
- `GPIOA_MODER`, `GPIOA_IDR`, and `GPIOA_ODR` configure and control the
pushbutton and LCD control signals.
- `GPIOC_MODER` and `GPIOC_ODR` drive the LCD's 8-bit data bus.
- ARM registers R4-R12 retain obstacle positions, animation frames,
button state, and player position during gameplay.

For a complete CPU and peripheral register map, see the [registers table](docs/register_map.md).

