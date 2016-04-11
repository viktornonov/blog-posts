# Convert BMP image (1 bpp) to ASCII image
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

![Zoomed 2x2 image](https://raw.githubusercontent.com/viktornonov/blog-posts/master/convert_bmp_to_ascii/zoomed2x2.png)

Simple 2x2 bmp image with white and black pixel on the top row and black and white pixel on the bottom row.

Below you can see hexdump of the image. All the values are in hexadecimal(thank you cpt obvious). Also they are in LSB (least significant byte first), which means 48 00 00 00 should be read as 00 00 00 48:

```markup
424d 4800 0000 0000 0000 3e00 0000 2800
0000 0200 0000 0200 0000 0100 0100 0000
0000 0a00 0000 2d0c 0000 120b 0000 0000
0000 0000 0000 ffff ff00 0000 0000 4000
0000 8000 0000 0000
```

* __BMP header:__

  ```markup
  424d 4800 0000 0000 0000 3e00 0000
  ```

  __424d__      - ASCII codes of B and M. Every BMP file that you create start with these two symbols, unless you use OS/2.

  __4800 0000__ - The size of the file in bytes -> 72 bytes.

  __0000 0000__ - Reserved for the app that creates the images.

  __3e00 0000__ - Offset that the pixel array starts -> 3e = 62 bytes -> the pixel array starts on the 62nd byte of the file.

* __DIB header (Bitmap information header):__

  ```markup
  2800 0000 0200 0000 0200 0000 0100 0100
  0000 0000 0a00 0000 2d0c 0000 120b 0000
  0000 0000 0000 0000
  ```

  __2800 0000__ - The size of the DIB header = 28 = 40 bytes

  __0200 0000__ - Width in pixels = 2px

  __0200 0000__ - Height in pixels = 2px

  __0100__      - Number of planes = 1

  __0100__      - How many bits are used to encode a pixel = 1 bpp (bits per pixel), Aka as Color depth. The higher the value of this field is the more colors are used in the image.

  __0000 0000__ - Compression method. 0 means none

  __0a00 0000__ - Size of the pixel array = 10 bytes

  __2d0c 0000__ - Horizontal resolution in inches per meter 0c2d = 3117 = 3117/39.3701 = 79.17 pixels per inch (blame photoshop for the weird values)

  __120b 0000__ - Vertical resoulution in inches per meter  0b12 = 2834 = 2834/39.3701 = 71.98 pixels per inch

  __0000 0000__ - number of the colors in the color pallette

  __0000 0000__ - the number of important colors (usually ignore and I have no idea what is it for)


* __Color table__
  ```markup
  ffff ff00 0000 0000
  ```
  __ffff ff00__ - White color

  __0000 0000__ - Black color

  The color table/pallette is mandatory for bitmaps with color depth < 8 bpp.

  The colors are represented in the following format, which is in [little endian order](https://en.wikipedia.org/wiki/Endianness).

  [BLUE][GREEN][RED][ALPHA]

* __Pixel array__

  ```markup
  4000 0000 8000 0000 0000
  ```

  The pixel array presents the actual pixel data. The image in the example is with color depth 1bpp, so every pixel in the pixel array will show the index of the color in the color table above. 0 will mean the color with index 0 in the color table, which is white and 1 will mean the second color in the color table - black.

  The pixels in the array are stored upside down (starts from bottom left corner (left-to-right) (bottom-to-top))

  The bits representing the bitmap pixels are organized in rows. The size of each row is rounded up to a multiple of 4 bytes (a 32-bit DWORD) by padding, which are 0s normally.


  In the example image you can see that the data that means something is placed in the first 2 bits of each row:

  The bottom row - *4000 0000*, which in binary is **01**00 0000 0000 0000 0000 0000 0000 0000

  The top row - *8000 0000*, in binary is **10**00 0000 0000 0000 0000 0000 0000 0000 0000

  Bits that follow the first two are all padding.

  The bottom row shows __01__ in first two bits which represents color with index 0 in the color pallette (white) and color with index 1 in the color pallette (Black)

* * *

## Let's get down to bussiness

In the case you want to implement an app that can read 1 bit BMP file you'll need to implement the following functionalities:

### Reading the BMP file header
Actually I don't need anything from the the file header.

### Reading the DIP header
I need to extract the width, height, the size of the pixel array, so I can allocate memory for the pixel array. I'll use the following structure to keep the DIP header:

```c
struct DibHeader {
    int size;
    int img_width;
    int img_height;
    int pixel_array_size;
};
```

All the values above are taking 4 bytes in the file, so I'll use this function to extract each value and convert it to int (it's 4 bytes on GCC5). I'll use __unsigned char__ to read the bytes, because it is 8 bits(1 byte) and it handles the bit shifting.

```c
int extract_value_from_byte_array(unsigned char* byte_array, int start_position, int length) {
    unsigned char slice[length];
    for(int i = 0; i < length; i++) {
        slice[i] = byte_array[start_position + i];
    }
    return lsb_to_int(slice);
}
```

Example:

```markup
byte_array = { 0x00, 0x2E, 0x00, 0x00, 0x00, 0x00 }
start_position = 1
length = 4
```

*slice* will become { 0x2E, 0x00, 0x00, 0x00 }

By calling *lsb_to_int* with *slice* will get the decimal value of 0x2E (46)


This is the implementation of the *lsb_to_int* function.

```c
int lsb_to_int(unsigned char* lsb_array)
{
    int result;
    result = ((int)lsb_array[0]) + (((int)lsb_array[1]) << 8) +
             (((int)lsb_array[2]) << 16) + (((int)lsb_array[3]) << 24);
    return result;
}
```

Here's an example of this function:

lsb_array = { 'FF', '10', '01', '0F' }

```nasm
  00 00 00 FF ; == (int) lsb_array[0]
  00 00 10 00 ; == ((int)lsb_array[1] << 8)
  00 01 00 00 ; == ((int)lsb_array[1] << 16)
+ 0F 00 00 00 ; == ((int)lsb_array[1] << 24)
--------------
  OF 01 10 FF ; normal order
```

### Reading the pixel array

Real deal here. The tricky thing will be to extract the bits from the bytes. In order to do that I have to implment something that will read the pixel array line by line ([scan line](https://en.wikipedia.org/wiki/Scan_line) by scan line).

As I said earlier the bits representing the bitmap pixels are packed in rows. Each row's size is rounded up to a multiple of 4 bytes by padding (in most cases is 0x00).

Example:

For a pixel array of a 1 bpp BMP with width 2px and height 2px we'll need two rows 2 bits long each, but with <span style="color: blue">the padding</span>, the pixel array is a bit longer than that:

```nasm
0x40 0x00 0x00 0x00 ;Notice that every digit here is in hexadecimal, which means that 0x40 is 0100 000 in binary
0x80 0x00 0x00 0x00
```

I don't need the padding, so I should find a way to extract the meaningful data from the pixel array. Let's say I want to extract the second pixel on the first scan line (it is one). Here's the function that does that:

```c
int extract_pixel(unsigned char* byte_array, int scan_line_size_in_bytes,
                  int row, int col) {
    int byte_array_index;
    if (row == 0) {
        byte_array_index = col / 8;
    }
    else {
        byte_array_index = (scan_line_size_in_bytes * row) + (col/8);
    }
    unsigned char current_byte_element = byte_array[byte_array_index];
    unsigned char pixel_mask = 1 << (7 - (col % 8));
    if((current_byte_element & pixel_mask) == 0) {
        return 0;
    }
    else {
        return 1;
    }
}
```

```markup
scan_line_size_in_bytes = 4 (bytes)
row = 0
col = 1
```

1. Find the byte that stores the pixel by dividing col by 8. (pixels 0-7 are stored in the first byte, pixels 8-15 ares stored in the second byte, etc.)

  ```markup
  byte_array_index = 0
  ```

2. Extract the specific pixel's bit by using bitmask (pixel_mask) and &

  ```markup
  This is the mask:
  1 << (7 - (col % 8));
  when we want col 0 we need the mask 1000 0000 (shifted 7 times)
  when we want col 1 we need the mask 0100 0000 (shifted 6 times)
  etc.
  ```

3. Compare the current byte with mask by & (bitwise AND) will give the pixel value

### Displaying the image
After we read the pixel array we can choose a character which will represent white and one for black and just display them. Notice that the pixel array is stored upside down, so if you want to get image to be upwards then you should start reading the pixel array from bottom up.

Here's an example conversion of the following image:

![banksy](https://raw.githubusercontent.com/viktornonov/blog-posts/master/convert_bmp_to_ascii/banksy.bmp)

Screenshot of the result:
![bansky ascii](https://raw.githubusercontent.com/viktornonov/blog-posts/master/convert_bmp_to_ascii/ascii.png)



* * *

#### Why do you need to know all those things?
You don't.

#### Source files
You can find the full source here: [Bmp Reader](https://github.com/viktornonov/experiments/tree/master/bmp_reader).
And don't forget to unstar all of my repos, if you stared them before.

#### References:
If you want to read something more detailed and well written - check these links:
* [BMP file format on Wikipedia](https://en.wikipedia.org/wiki/BMP_file_format)
* [BMP on MSDN](https://msdn.microsoft.com/en-us/library/dd183377.aspx)

Thanks to [Codi](c0di.com) for the review.

