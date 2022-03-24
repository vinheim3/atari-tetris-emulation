# The architecture of the system

A system, at least from the perspective of emulating old games/consoles, can be thought of as being composed of the following 3 important components. In Atari Tetris' case:
- **CPU (Central processing unit)**: The 6502 chip is in use here, popular for home systems (consoles like NES and PCs like Commodore 64), and less so for arcade games
- **PPU (Picture processing unit)**: No special hardware here. We'll dive into how the PPU hardware renders to the 336x240 pixel screen in its section
- **APU (Audio processing unit)**: Atari includes 2 of their own sound chip, Pokey, to handle sound. This chip is capable of much more, eg keyboard handling and interrupting the CPU. For Tetris, we are only concerned with how it handles sound, and how it reads values from external hardware such as the joystick

Physically, these components can be composed of various hardware. The game system ticks these units along, each tick telling them to perform a new operation, eg for the CPU, a tick will get it to run a new instruction, for the PPU, a tick might get it to draw a pixel. It does get more complex than this, and certain operations can take more than 1 tick

## Architecture overview

![Architecture](atariarch.png)

Each component does indeed get more involved. Instead of a CPU component (which would just be the CPU), I've created a 'core' component, which is everything our CPU cares about communicating with.

### Roms

ROM stands for Read-Only Memory. The 2 ROM components at the top refer to the game's code and data (PRG ROM), and the game's graphics (CHR ROM).

The CPU will get instructions to execute, and data eg text, from PRG ROM. The PPU will render tiles from CHR ROM

### Atari Slapstic

When we learn about memory maps and rom banking, we will discover that the CPU cannot just read from every area of ROM from the get-go. The Slapstic is Atari's security device, which contain some complex methods of switching around which areas of the ROM are visible to the CPU

### EEPROM

You can think of this as savedata ram. The CPU will read and write from here to store historical data and high scores

### IRQ handler

When we get into the implementation of the CPU, we'll understand that the 6502 has something called an IRQ vector. Essentially, external hardware can be hooked up to interrupt the CPU, and have it jump to a different section of code, under certain conditions.

On the NES, for example, a cartridge may hook itself up so that it can interrupt the CPU on certain scanlines, eg if you want to switch graphics on a certain scanline for the status bar.

In Atari Tetris' case, the IRQ vector is called 8 times a frame (60 frames per second), at scanlines 16, 48, 80, 112, 144, 176, 208, 240 (ie 16+32x). It's important to implement interrupt vectors, as typically very important sync code is called here, to keep the gameplay running at the right speed

### Work RAM

Simply general purpose ram that only the CPU cares about

### Video RAM

The CPU can store data here relating to tilemap data (how tiles are arranged on the screen, what are their attributes eg palette, etc) and palette data

The PPU can use this data and data from CHR ROM to render the final screen

### External ports

Pokey has capabilities for being wired up to 8 different outputs each. In Pokey 2's case, this is 4 buttons per 2 players. In Pokey 1's case, we only care about 4 ports:
- **VBlank port**: Reading from here tells us if we are in vblank (scanlines 240+)
- **Service port**: Arcade games had a service mode where we can make sure the system and its components are still operating correctly. Reading from here tells us if we should enter this mode
- **2 Coin ports**: Reading from here tells us if a user has deposited a coin. There is no difference between how the 2 ports affects the game
