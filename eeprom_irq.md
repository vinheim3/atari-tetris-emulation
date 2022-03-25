# EEPROM

The EEPROM is a read/write save data area of 0x200 bytes, from 0x2400-0x25ff. It starts off in a 'locked' state, and should be initialized to all 0xff, otherwise the game will not boot due to some code being stuck in an infinite loop waiting to read an 0xff

Reads can only happen while locked

Writes can only happen while unlocked, and once written to, the EEPROM is locked

The EEPROM can be unlocked by writing to 0x3400

# IRQs

At scanlines `16 + 32*v` for `v = 0 to 7`, the system will hold the CPU's IRQ line active. This will cause the CPU to be interrupted and jump to the IRQ vector if interrupts are enabled for it

Considering that servicing the interrupt disables interrupts in the process, and that the IRQ vector's `reti` will re-enable it, the game will need to find some way to make the CPU's IRQ line inactive, otherwise when returning from the IRQ vector, the CPU will go back and service the IRQ vector again straight away

Writing to 0x3800 will acknowledge the IRQ and cause the system to hold the CPU's IRQ line inactive
