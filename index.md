## Emulating Atari's Tetris

You may have just come from emulating Space Invaders, or even Chip 8/Bytepusher, and want the next challenge. The next usual steps up (Gameboy/NES) are not completey insurmountable, however I present an option for a more gradual difficulty increase.

At the end, you will
- have a reusable implementation of the same cpu core that systems such as the NES and Commodore 64 use
- understand how the common tilemap rendering system works
- understand how to go about implementing a sound chip
- understand some other common miscellaneous concepts like rom banking, dealing with memory maps, irqs, vertical blank, etc

### Table of Contents

1. [The architecture of the system](architecture.md)
2. [Memory map](memorymap.md]
3. Emulating the cpu (6502)
4. Tilemap rendering
5. Atari's Slapstic 101
6. EEPROM
7. IRQ + port handling
8. The sound chip (Atari's Pokey)
