# The Sound Chip

There are some good resources here:
* https://usermanual.wiki/Document/pokeyC012294.3349751284 - Datasheet
* https://www.atariarchives.org/dere/chapt07.php - A user-friendly form of the above

The Pokey sound chip has other non-sound features. For completeness, the ones used by Atari's Tetris are (the values on the left are the address offset for the register):

* 0x8 (read) - Where port values are read from
* 0xa (read) - Gets a random byte value
* 0xb (write) - Gets up-to-date port values when reading from 0x8, for 228 scanlines
* 0xf (write) - I don't know much about the bits set here, it's not necessary to implement

For sound, the Pokey has 4 channels that produce square waves. The offsets that the game writes to are:

* 0, 2, 4, 6 - frequency divider for each channel
* 1, 3, 5, 7 - control byte for each channel
* 8 - audio control, ie non-channel specific control

### The basics

This guide assumes you're a complete beginner with sound. There are resources like: https://stikeys.co/tutorials/understanding-sound-waves-and-waveforms/ which explores sound from that beginner perspective.

Putting all this info into some practical application, we can start with producing a square wave with a frequency of 440Hz, commonly referred to as a pitch of A in octave 4: https://mixbutton.com/mixing-articles/music-note-to-frequency-chart/. After producing it, we can compare with playing an A pitch on an instrument or online.

Playing a sound typically requires us to send samples based on your sound programming library of choice. Common sample rates include 44100Hz or 48000Hz, which is the number of samples we need to send per second.

440Hz means we need to plot our square wave (imagine the sine wave but with flat tops and bottoms like the diagrams in atariarchives), 440 times a second laid out horizontally. We then need to split that graph evenly, horizontally 44100 times. The Y values of the wave at each of the points represents 1 of the 44100 sample values we want to send to our library of choice. The audio device will put these samples together, and cause the speaker to move based on those values, giving us the 440Hz A sound.

In reality, we will be producing these samples in real time: we will have some small buffer that will contain generated samples and send them to the audio device in chunks. We will be generating a sample every 1/44100 seconds based on our emulated APU's speed. Tetris has a master clock of 14,318,181Hz, the CPU and APU have a clock speed of 1/8th that, which means they tick at 1,789,772Hz. Divide that by 44100, for example, and we need to add a sample onto our buffer approx every 40.6 CPU/APU cycles. Division will be used to generate these after the right integer amounts of CPU cycles.

We can experiment outside of the emulator, without need for doing realtime audio updates, just to verify our understanding. Assuming 60 frames/second, generate 735 samples, based on where you believe the 440Hz square wave to be at, using the max value the sound lib requires for the square wave tops, and the min value for the square wave bottoms

TODO:
