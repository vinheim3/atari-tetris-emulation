## Emulating Atari's Tetris

You may have just come from emulating Space Invaders, or even Chip 8/Bytepusher, and want the next challenge. The next usual steps up (Gameboy/NES) are not completey insurmountable, however I present an option for a more gradual difficulty increase.

At the end, you will
- have a reusable implementation of the same cpu core that systems such as the NES and Commodore 64 use
- understand how the common tilemap rendering system works
- understand how to go about implementing a sound chip
- understand some other common miscellaneous concepts like rom banking, dealing with memory maps, irqs, vertical blank, etc

This guide assumes some familiarity with emulating a simpler system

Credit mostly goes to the MAME source for detailed information on the system

### Table of Contents

1. [The architecture of the system](architecture.md)
2. [Memory map](memorymap.md)
3. [Emulating the cpu (6502)](6502.md)
4. [Tilemap rendering](rendering.md)
5. [Atari's Slapstic 101](slapstic101.md)
6. [EEPROM + IRQ + Port handling](eeprom_irq_port_handling.md)
7. [The sound chip (Atari's Pokey)](pokey.md)
