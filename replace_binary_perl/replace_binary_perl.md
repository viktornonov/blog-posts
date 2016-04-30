# Binary replacement

Text replacement is a very common thing to do when you write code, but have you ever needed to replace a binary sequence in a file? (yeah, me neither)
Anyways, you can use this one liner to do it:

```sh
perl -pi -e "s/\x20\x69/\x69\x20/g"
```

Breakdown of the command:

* **-p** adds loop around the -e code

```perl
while (<>) {
  s/\x20\x69/\x69\x20/g
} continue {
  print or die "-p destination: %!\n";
}

```

* **-i** the replacement will be performed in-place (perl will read the file, make the substitution and the write the result in a new file with same name, which basically will override it)
* **-e** indicates that perl code follows
* **s/\x20\x69/\x69\x20/g** replace 0x20 0x69 binary sequence with the 0x69 0x20 sequence

