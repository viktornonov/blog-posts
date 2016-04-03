# Convert BMP image (1 bit) to ASCII art
## BMP File format

The BMP file format is a raster graphics image file format. It's pretty straight forward to implement a reader for it. (Although it took me 5 days)
It has 4 main parts:
* __BMP header__ - stores general information about the file like size of the file
* __DIB header__ - bitmap information header. Stores width, height, bits/pixel encoding etc.
* __Color table__ - lists all the colors used in the image
* __Pixel array__ - describes the image's pixels

### Example (2x2 BMP Image (1 bit per pixel encoded))
1 bit per pixel encoded bmp images use 1 bit to represent each pixel in the image, meaning that a pixel can be either 0 or 1. (black or white).
That's the __zoomed__ version of the image that's the example is for:
![Zoomed 2x2 image](https://raw.githubusercontent.com/viktornonov/blog-posts/master/bmp_format/2x2zoomed.png)
Simple 2x2 bmp image with white and black pixel on the top row and black and white pixel on the bottom row.

Below you can see hexdump of the image. All the values are in hexadecimal(thank you cpt obvious). Also they are in LSB (least significant byte first), which means 48 00 00 00 should be read as 00 00 00 48:
```
424d 4800 0000 0000 0000 3e00 0000 2800
0000 0200 0000 0200 0000 0100 0100 0000
0000 0a00 0000 2d0c 0000 120b 0000 0000
0000 0000 0000 ffff ff00 0000 0000 4000
0000 8000 0000 0000
```

* BMP header:
```
424d 4800 0000 0000 0000 3e00 0000
```

424d      - ASCII codes of B and M. Every BMP file that you create start with these two symbols, unless you use OS/2.
4800 0000 - The size of the file in bytes -> 72 bytes.
0000 0000 - Reserved for the app that creates the images.
3e00 0000 - Offset that the pixel array starts -> 3e = 62 bytes -> the pixel array starts on the 62nd byte of the file.

* DIB header (Bitmap information header):
```
2800 0000 0200 0000 0200 0000 0100 0100
0000 0000 0a00 0000 2d0c 0000 120b 0000
0000 0000 0000 0000
```

2800 0000 - The size of the DIB header = 28 = 40 bytes
0200 0000 - Width in pixels = 2px
0200 0000 - Height in pixels = 2px
0100      - Number of planes = 1px
0100      - How many bits are used to encode a pixel = 1 bpp (bits per pixel), Aka as Color depth. The higher the value of this field is the more colors are used in the image.
0000 0000 - Compression method. 0 means none
0a00 0000 - Size of the pixel array = 10 bytes
2d0c 0000 - Horizontal resolution in inches per meter 0c2d = 3117 = 3117/39.3701 = 79.17 pixels per inch (blame photoshop for the weird values)
120b 0000 - Vertical resoulution in inches per meter  0b12 = 2834 = 2834/39.3701 = 71.98 pixels per inch
0000 0000 - number of the colors in the color pallette
0000 0000 - the number of important colors (usually ignore and I have no idea what is it for)


* Color table
```
ffff ff00 0000 0000
```
ffff ff00 - White color
0000 0000 - Black color

The color table/pallette is mandatory for bitmaps with color depth < 8 bpp.
The colors are represented in the following format, which is in [little endian order](https://en.wikipedia.org/wiki/Endianness).
[BLUE][GREEN][RED][ALPHA]

* Pixel array
```
4000 0000 8000 0000 0000
```
The pixel array presents the actual pixel data. The image in the example is with color depth 1bpp, so every pixel in the pixel array will show the index of the color in the color table above. 0 will mean the color with index 0 in the color table, which means white and 1 will mean the second color in the color table - black.
The pixels in the array are stored upside down (starts from bottom left corner (left-to-right) (bottom-to-top))
The bits representing the bitmap pixels are organized in rows. The size of each row is rounded up to a multiple of 4 bytes (a 32-bit DWORD) by padding, which are 0s normally.
In the example image you can see that the data that means something is placed in the first 2 bits of each row:
The bottom row - 4000 0000, which in binary is __01__00 0000 0000 0000 0000 0000 0000 0000
The top row - 8000 0000, in binary is __10__00 0000 0000 0000 0000 0000 0000 0000 0000
Bits that follow the first two are all padding.
The bottom row shows __01__ in first two bits which represents color with index 0 in the color pallette (white) and color with index 1 in the color pallette (Black)

* * *

### Why do you need to know all those things?
You don't.

### References:
If you want to read something more detailed and well written - check these links:
* [BMP file format on Wikipedia](https://en.wikipedia.org/wiki/BMP_file_format)
* [BMP on MSDN](https://msdn.microsoft.com/en-us/library/dd183377.aspx)

