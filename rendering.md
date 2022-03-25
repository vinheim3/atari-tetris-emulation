# Tilemap rendering

### General info - tiles

Atari's Tetris has a 0x10000-byte ROM for its graphics. These graphics are stored such that every 0x20 bytes represents a 4bpp 8x8 tile. This gives us 0x800 tiles

Vram is 0x1000 bytes, from 0x1000-0x1fff. Each 8x8 pixel tile takes up 2 bytes, the 1st being a tile idx that references a 0x20-byte tile in CHR ROM, and the 2nd being tile attributes

As a byte can only refer to 0x100 bytes, 3 bits of the tile attribute allow us to refer to up to tile 0x7ff

If you lay out the 0x1000/2 tiles as being 0x40 tiles wide and 0x20 tiles high, this would measure 512 pixels wide by 256 pixels high

The screen is 336 pixels wide and 240 pixels high, so some tiles are not displayed

### General info - palettes

Palettes use up 0x100 bytes, from 0x2000-0x20ff. This consists of 16 palettes with 16 colours each, as 4bpp allows you to reference 16 colours

Each colour, taking up a byte each, has the following bit format: `rrrgggbb` from most-significant bit to least

Multiplying red and green values by 36 gets you a range of 0 to 252, and multiplying blue by 85, gets you a range of 0 to 255

### 4bpp format

Each pixel row of an 8x8 tile uses 4 consecutive bytes each

Naming the bytes b0, b1, b2, b3, and shorthanding the high nybble of a byte as H, and low nybble as L, the 8 row pixels are setup like so:

b0H b0L b1H b1L b2H b2L b3H b3L

So each pixel takes up a value between 0 to 15. For initial experimentation, plotting pixels such that each of R, G and B components are `pixel val * 17` will display the grayscale version of the graphics

### Attributes

We know how tiles are plotted, and how to draw each tile in grayscale, but this is a colour game. The tiles have a separate attribute byte setup like so

`ppppxttt`, p - palette idx, t - high byte of tile idx

* Palette idx - simply put, this is 1 of 16 values that references the 16 palettes
* High byte of tile idx - as previously mentioned these bits help refer to any tile in CHR ROM
