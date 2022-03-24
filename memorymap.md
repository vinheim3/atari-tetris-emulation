# The memory map

When emulating a given system, an overly simplified, and partial in many cases, view would be that you are emulating a CPU and how it reads from an address, how it processes that data and internal data, and how it can write out to addresses.

Whichever components these read/writes happen to communicate with, determines the side effects of the system, or the data that gets returned back to you, eg writing to 0x2802 affects some sound frequency internal to the 1st Pokey sound chip.

![Memory map](memorymap.png)

### RAM

Work RAM, Video RAM and Palette RAM have no special considerations. They are addresses you can read from/write to.

### EEPROM

The 0x200 region is like RAM, except it does have a 'locked' state. Reads can only happen while locked. Writes can only happen while unlocked. Writes will also auto-lock the EEPROM, so you can only do 1 write at a time before unlocking it. Unlocking happens via writing to the 'EEPROM Write Enable' at 0x3400

### Pokey

Each Pokey device has 16 registers you can read/write from. Reads and writes have very different effects on the same address, eg reading from offset 8 (0x2808/0x2818) reads info from joypads/inputs/dipswitches/etc, whereas writing to offset 8 sets the Pokey's 'Audio control'

### Misc

This refers to the registers from 0x3000 to 0x3c00
* Watchdog - I have not yet implemented this, but based on some searching: https://wiki.neogeodev.org/index.php?title=Watchdog, implementing this should reset the system if not regularly written to
* EEPROM Write Enable - discussed in the EEPROM section above
* IRQ Acknowledge - we will understand this when going through IRQs, but this will prevent the CPU from continuously being interrupted away from main game code
* Coin counters - I have not yet implemented this, and from searching, it seems similar to Watchdog in that it may have been used for hardware-related issues

### PRG ROM

The PRG ROM file is 0x10000 in size. On a 6502 this would take up the entire 16-bit address space. In order to cater to ROM size issues in general, a concept of ROM banking had already been around. Essentially the CPU would not directly read from ROM, but some intermediate device that could decide which window of the ROM to read from. A developer would have control over that window, via writes to an address that the intermediate device cares about reading from. The Gameboy, for example, could handle 8MB games despite having a 32KB ROM region

A ROM bank refers to a window of code that could be swapped to read from a different section of ROM, or it could be fixed to look at the same window at all times. Atari's Tetris has:
* a banked ROM region, 0x4000 in size. It reads from the ROM at offsets 0x0000-0x3fff, OR 0x4000-0x7fff
* a fixed ROM region, 0x8000 in size. It reads from the ROM at offsets 0x8000-0xffff, ie the memory map addresses do not have to be adjusted to read from the right offset in ROM

The the intermediate device is Atari's Slapstic. Writing to 0x6000-0x7fff in very specific ways, will change which of the 2 ROM regions the CPU reads from, when reading between 0x4000-0x7fff
