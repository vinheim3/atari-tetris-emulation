# Atari's Slapstic

This is a device that, when written to, changes the swappable ROM bank region at memory map address 0x4000-0x7fff

The Slapstic is a security device by Atari that made it difficult to just copy its ROM and run on a different machine

It would hold an internal rom bank value between 0-3, initialized to 3. When that value is 0 or 2, PRG ROM at 0x0000-0x3fff could be read from. When that value is 1 or 3, PRG ROM at 0x4000-0x7fff could be read from

The Slapstic has an internal state machine, starting at `IDLE`. During the `BIT_SET` state, it also has an `odd/even` mode

In the following sections, Slapstic offsets means an offset from its memory map address at 0x6000. 

## Changing banks

The device, and its successors, had 3 methods of changing banks: basic, alternative and bitwise. You only need basic and bitwise for Tetris.

To attempt any of these modes, you needed to move from `IDLE` to `ACTIVE` by writing to offset 0x0000

### Basic

By simply writing to offsets 0x80, 0x90, 0xa0, 0xb0, (ie 0x6080-0x60b0) you would change the internal rom bank value respectively. The state would then go back to `IDLE`

You can write to 0x6080 and 0x6090 and that would be sufficient from a programmer's perspective for developing Tetris, but I guess Atari would have required use of the more difficult options to take advantage of the security device

Basic banking mode is only used during Service mode, when checking if the ROM is valid

### Bitwise

In this mode, very specific addresses are written to in sequence, so keeping track of the state is more important. At any point, writing to offset 0x0000 goes back to the `ACTIVE` state

* Writing to 0x1540-0x154f changes state to `BIT_LOAD`
* In `BIT_LOAD` mode, writing to the offsets 0x80, 0x90, 0xa0 or 0xb0:
    * Starts keeping track of the current bank, for manipulation
    * Puts us in `odd` mode
    * changes to state `BIT_SET`
* In `BIT_SET` mode, we can manipulate the bits of the current bank (it won't change the bank yet). After each manip, we swap between `odd` mode and `even` mode. The 1st 4 address are repeated 4 times in the 0x1540-0x154f region
    * Writing to 0x1540 clears bit 0 in odd mode and sets bit 1 in even mode
    * Writing to 0x1541 sets bit 0 in odd mode and clears bit 1 in even mode
    * Writing to 0x1542 clears bit 1 in odd mode and sets bit 0 in even mode
    * Writing to 0x1543 sets bit 1 in odd mode and clears bit 0 in even mode
    * Writing to 0x1550-0x1557 will commit the above changes and go to the `IDLE` state

