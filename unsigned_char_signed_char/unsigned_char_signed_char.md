# Difference between char(signed char) and unsigned char

When you manipulate binary data in C the **unsigned char** is preferable type to use, but how **char** can be unsigned or signed?

**char** in C is just integer type with length 8 bits or 1 byte. It stores the ASCII code of the character and when we **printf** with **%c** we get the character.

```c
char character = 'a';
printf("%d", character); //=> 97 (ASCII code of 'a')
printf("%c", character); //=> a

```

So if **char** is just undercover integer, then we can explain how it can be **unsigned** or **signed**.

When we define **unsigned char** we will treat the most significant bit as normal bit and we'll be able to represent 2^8 values from 0 to 255:

```c
unsigned char byte = 0;
byte |= 128; //128 in binary is 1000 0000
printf("%d", byte); //=> 128
byte = 0;
byte |= 256; //256 in binary is 1 0000 0000
printf("%d", byte); //=> 0 (the unsinged char is just 8 bits so no bits were touched)
```

When we define **char** (implicitly **signed char**  \*), we can represent values from -128 to +127, because we treat the most significant bit as a sign bit:

```c
char byte = 0;
byte |= 128; //128 in binary is 1000 0000
printf("%d", byte); //=> -128 (because the most significant bit(sign bit) is 1)
```

\* This is the case with Apple LLVM version 8.1.0 (clang-802.0.42) and probably gcc 6.3.0. Some other compilers may implement char as unsigned char. You can check if your char is signed or unsigned in the **limits** header file

