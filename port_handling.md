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
* 
