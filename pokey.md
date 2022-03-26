# The Sound Chip

There are some good resources here:
* https://usermanual.wiki/Document/pokeyC012294.3349751284 - Datasheet
* https://www.atariarchives.org/dere/chapt07.php - A user-friendly form of the above

The Pokey sound chip has other non-sound features. For completeness, they are (the values on the left are the address offset for the register):

* 0x8 (read) - Where port values are read from
* 0xa (read) - Gets a random byte value
* 0xb (write) - Gets up-to-date port values when reading from 0x8, for 228 scanlines
* 0xf (write) - I don't know much about the bits set here, it's not necessary to implement

For sound, the Pokey has 4 channels that produce square waves. The offsets that the game writes to are:

* 0, 2, 4, 6 - frequency divider for each channel
* 1, 3, 5, 7 - control byte for each channel
* 8 - audio control, ie non-channel specific control

### The basics

TODO:
