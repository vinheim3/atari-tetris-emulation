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

### Quick specs
These are for reference, and will be explained in further detail later in this page.
* Frequency divider tells us how many APU ticks until the square wave pulse changes.
* Channel control byte is structured `nnnovvvv`, where `n` is the noise sampling params, `o` is if the channel is volume-only, and `vvvv` is the channels' volume
* Audio control byte has bits with the following meaning when set
    * bit 7 - use 9-bit poly counter instead of 17-bit
    * bit 6 - channel 1 is clocked by 1.79MHz instead of 64KHz
    * bit 5 - channel 3 is clocked by 1.79MHz instead of 64KHz
    * bit 4 - channel 2 is clocked by channel 1 (the 2 are combined to give a 16-bit frequency divider, not multiplicative, ie chn1 with a freq divider of 0x17, and chn2 with a freq divider of 0xc0, gives a total freq divider of 0xc017)
    * bit 3 - channel 4 is clocked by channel 3
    * bit 2 - high pass filter in channel 1, clocked by channel 3
    * bit 1 - high pass filter in channel 2, clocked by channel 4
    * bit 0 - clock channels by 15KHz instead of 64KHz

Tetris does not use high pass filters, or volume-only channels.
1.79MHz is the speed of the APU.
64KHz's precise value means that we tick frequency counters every 28 APU ticks, rather than every tick.
15KHz's precise value means that we tick every 112 APU ticks.

### The basics

This guide assumes you're a complete beginner with sound. There are resources like: https://stikeys.co/tutorials/understanding-sound-waves-and-waveforms/ which explores sound from that beginner perspective.

Putting all this info into some practical application, we can start with producing a square wave with a frequency of 440Hz, commonly referred to as a pitch of A in octave 4: https://mixbutton.com/mixing-articles/music-note-to-frequency-chart/. After producing it, we can compare with playing an A pitch on an instrument or online.

Playing a sound typically requires us to send samples based on your sound programming library of choice. Common sample rates include 44100Hz or 48000Hz, which is the number of samples we need to send per second.

440Hz means we need to plot our square wave (imagine the sine wave but with flat tops and bottoms like the diagrams in atariarchives), 440 times a second laid out horizontally. We then need to split that graph evenly, horizontally 44100 times. The Y values of the wave at each of the points represents 1 of the 44100 sample values we want to send to our library of choice. The audio device will put these samples together, and cause the speaker to move based on those values, giving us the 440Hz A sound.

In reality, we will be producing these samples in real time: we will have some small buffer that will contain generated samples and send them to the audio device in chunks. We will be generating a sample every 1/44100 seconds based on our emulated APU's speed. Tetris' CPU and APU have a clock speed of 1,789,772Hz. Divide that by 44100, for example, and we need to add a sample onto our buffer approx every 40.6 CPU/APU cycles. Division will be used to generate these after the right integer amounts of CPU cycles.

Here are some sample phases, step-by-step, to get us to emulating the sound chip as needed for Tetris.

### Phase 1 - generating sounds outside the emulator
* Generate + send 735 samples, 60 times a second, based on where you believe the 440Hz square wave to be at
* Use the max value the sound lib requires for the square wave tops, and the min value for the square wave bottoms
* Send the 735 samples at the end of each frame

### Phase 2 - realtime updates in emulator
* After every CPU instruction, detect if we've past the next 1/44100 seconds, then generate the next of the 735 samples for the frame
* This will be the last phase for the manual A pitch

### Phase 3 - use correct clocks and frequency dividers
The Atari docs say that the frequency is approximately `clock/ 2(divider+1)`. The way we can interpret this, is that when a frequency divider is set, we add 1 to it, and it becomes a base counter that tells us when a square wave pulse alternates between high and low. Every 2 times that counter reaches 0, we have a complete square wav, so the frequency of that square wave is produced `2(divider+1)` times a second
* When Pokey registers 0-8 are written to, store their values appropriately
* If a channel's clock speed is 1.79MHz, tick their frequency counter every CPU/APU cycle. Change to 28 cycles per freq counter tick if 64KHz, and 112 cycles for 15KHz
* Keep track of a high/low phase for the square wave, and alternate each time the frequency counter goes to 0.
* Reset the counter to its base counter whenever 0 is reached

### Phase 4 - implement 16-bit channels
At this point, we should have different sounds playing. Due to lack of noise sampling, and combining channels, the pitches will be very high
* Whenever a channel is clocked by another, do not tick it down by itself. Instead tick the channels down as if a 16-bit value. This means the low byte channel (eg channel 1) does not need to care about its frequency divider
* Once both values are 0, alternate the square wave level, and reset both channel counters to their initial base counter values, then repeat ticking the 16-bit value

### Phase 5 - volume
* Volume is easy. Instead of sending a max sample value during the square wave's tops, send a `<max value> / 15 * <channel volume>` as the volume values range from 0 to 15

### Phase 6 - noise sampling
We will need to keep track of 4 poly counters, that tick every APU cycle, regardless of how the channels are clocked.
* 4-bit
    * Initially 0
    * Every APU tick, the complement of bit 0 is xor'd with bit 1 to get the new bit 3 as the whole 4-bit value is shifted right once
* 5-bit
    * Initially 0
    * Every APU tick, the complement of bit 0 is xor'd with bit 2 to get the new bit 4 as the whole 5-bit value is shifted right once
* 9-bit
    * Initially 0x1ff
    * Every APU tick, bit 0 is xor'd with bit 5 to get the new bit 8 as the whole 9-bit value is shifted right once
* 17-bit
    * Initially 0x1ffff
    * Every APU tick, bit 8 is xor'd with bit 13 to get the next bit 7. And bit 0 becomes the next bit 16 as the whole 17-bit value is shifted right once
As shown in atariarchives, if the appropriate poly counters do not have bit 0 set, when a square wave is about to pulse to its non-min value, then a single square wave up/down is not carried out
