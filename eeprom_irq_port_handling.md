# EEPROM

The EEPROM is a read/write save data area of 0x200 bytes, from 0x2400-0x25ff. It starts off in a 'locked' state, and should be initialized to all 0xff, otherwise the game will not boot due to some code being stuck in an infinite loop waiting to read an 0xff

Reads can only happen while locked

Writes can only happen while unlocked, and once written to, the EEPROM is locked

The EEPROM can be unlocked by writing to 0x3400

# IRQs

At scanlines `16 + 32*v` for `v = 0 to 7`, the system will hold the CPU's IRQ line active. This will cause the CPU to be interrupted and jump to the IRQ vector once interrupts are enabled for the CPU.

Considering that servicing the interrupt disables interrupts in the process, and that the IRQ vector's `reti` will re-enable it, the game will need to find some way to make the CPU's IRQ line inactive, otherwise when returning from the IRQ vector, the CPU will go back and service the IRQ vector again straight away

Writing to 0x3800 will acknowledge the IRQ and cause the system to hold the CPU's IRQ line inactive

# Port handling

With arcade games, the CPU will need some visibility of either the status of the game's inputs, or the status of other settings, eg dipswitches

The Pokey sound chips have various features other than sound, one of them being reading the values from up to 8 different ports. This is how Tetris has visibility of these values

After writing to a Pokey's register 0xb, for 228 scanline renders, you can read from a Pokey's register 8 to retrieve these values

Pokey 1 (0x2800-0x280f) will yield a value with these bits set:
* Bit 7 - If in service mode. This is a dipswitch setting, ie not user controlled, and it will determine if Tetris plays the main game, or goes into admin-like settings
* Bit 6 - If in vblank. This is when the scanline to render is 240+ (the last IRQ interrupt happens at 240, and it will check this port to execute different functionality
* Bit 1 - If a coin was entered into coin slot 1
* Bit 0 - If a coin was entered into coin slot 2. Same functionality as above

Pokey 2 (0x2810-0x281f) will yield a value with these bits set:
* Bit 7 - Player 2 left
* Bit 6 - Player 2 right
* Bit 5 - Player 2 down
* Bit 4 - Player 2's button
* Bit 3 - Player 1 left
* Bit 2 - Player 1 right
* Bit 1 - Player 1 down
* Bit 0 - Player 1's button
