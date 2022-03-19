## The architecture of the system

A system, at least from the perspective of emulating old games/consoles, can be thought of as being composed of 3 important components:
- CPU (Central processing unit)
- PPU (Picture processing unit)
- APU (Audio processing unit)

These components can be physically composed of various hardware. The game system ticks these units along, each tick telling them to perform a new operation, eg for the CPU, a tick will get it to run a new instruction, for the PPU, a tick might get it to draw a pixel. It does get more complex than this, and certain opertations can take more than 1 tick

In Atari Tetris' case:
- **CPU**: The 6502 chip is in use here, popular for home systems (consoles like NES and PCs like Commodore 64), and less so for arcade games
- **PPU**: No special hardware here. We'll dive into how the PPU hardware renders to the screen in its section
- **APU**: Atari includes their own sound chip, Pokey, to handle sound. This chip is capable of much more, eg keyboard handling and interrupting the CPU. For Tetris, we are only concerned with how it handles sound, and how it reads values from external hardware such as the joystick

<image>
