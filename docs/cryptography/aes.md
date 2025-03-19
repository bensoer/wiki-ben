# Advanced Encryption Standard (AES)

The Nationional Institute of Standards and Technology (NIST) put out a notice in early 1999 requesting submissions for a new encryption standards. The requirements were as follows:
* A symmetric block cipher with a variable length key (128, 192 or 256 bit) and a 128-bit block
* More secure then Triple-DES
* Must belong to the public domain - royalty free world wide
* Remain secure for at least 30 years

15 algorithms were submitted from 10 different countries. NIST relied on public participation in algorithm proposals, cryptanalysis and efficiency testing.

From August 20 - April 15, 1999 round 1 submissions were taken. A second conference for AES was held then on February 1, 1999 where the top 5 finalists were announced. The following 6 to 9 months was then spend analysing the finalists. At the thrid AES conference in 2001, the winner was announced. The Rijnhael implementation was selected, and the AES standard was born

The Reijnhael implementation contained the following specs:

* Variable block lenghts of 128, 192 and 256 bit
* Varialbe key lenght of 128, 192 and 256 bit
* Variable number of rounds (iterations) of 10,12 and 14
    * The number of rounds corresponds and depend on the key/block length

## Algorithm Example
For the algorithm example we will be primarily following documentation based on the NIST FIPS 197 Publication of the AES Algorithm. The original documentation can be found here : http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf

We are going to use the message:  

* `AESbeststherest!`

and encrypt it with the key: 

* `2B 7F 15 16 28 AE D2 A6 AB F7 15 88 08 CF 4F 3C`

Note that both of these make perfect 128-bit message and key. This example will use the 128-bit version of AES. This version requires a 128-bit key. For a message that is larger then 128 bits, this process merely needs to be repeated. The last bit then padded to fit into the 128-bit block.

Note that this implementation is also abstracted from either being EBS or CBC as these portions of the algorithm have to do with how each block is treated with the next in the final encryption. This can be easily wrapped over the current example to implement them

## Encryption


### Step 1: Convert Message And Place Into Matrix
Step one is to simply place the message into a matrix, in then convert it to its hexadecimal values, this is done simply with an ASCII table lookup

```
//plaintext
+===+===+===+===+    
| A | e | t | e |
+===+===+===+===+
| E | a | h | s |
+===+===+===+===+
| S | t | e | t |
+===+===+===+===+
| b | s | r | ! |
+===+===+===+===+

//convert to hex
+===+===+===+===+
|41 |65 |74 |65 |
+===+===+===+===+
|45 |61 |68 |73 |
+===+===+===+===+
|53 |74 |65 |74 |
+===+===+===+===+
|62 |73 |72 |21 |
+===+===+===+===+
```

### Step 2: S-box Lookup
Next take the matrix we created in step 1 and using each hex number as a row then column indexes, apply a substitution to each value. 

Taking the top left value in the matrix of 41, this means then to replace this value with the value located at row 4, column 1 in the S-box matrix (83). The S-box matrix is as follows:

![AES S-box](https://captanu.files.wordpress.com/2015/04/aes_sbox.jpg)


After running all values through the S-box, our matrix looks as follows:
```
+===+===+===+===+
|83 |4D |92 |4D |
+===+===+===+===+
|6E |EF |45 |8F |
+===+===+===+===+
|ED |92 |4D |92 |
+===+===+===+===+
|AA |8F |40 |FD |
+===+===+===+===+
```

### Step 3: Row Shift Operation
The next step is to row shift the matrix. We do this by starting with the top row and rotating it 0 boxes to the left, then the 2nd row by 1, 3rd by 2, and 4th by 3. The end result looks as follows:

```
//pre-shift
+===+===+===+===+
|83 |4D |92 |4D |
+===+===+===+===+
|6E |EF |45 |8F |
+===+===+===+===+
|ED |92 |4D |92 |
+===+===+===+===+
|AA |8F |40 |FD |
+===+===+===+===+

//post-shift
+===+===+===+===+
|83 |4D |92 |4D |
+===+===+===+===+
|EF |45 |8F |6E |
+===+===+===+===+
|4D |92 |ED |92 |
+===+===+===+===+
|FD |AA |8F |40 |
+===+===+===+===+
```

The effect here is similar to a left-shift except we wrap the end values around to the other end

## Decryption

## Key Schedule / Key Generation
The Key schedule process shares similar steps to the encryption process. The initial key is used to help generate additional sub-keys, which are then used at each iteration of the AES encryption procedure.

The initial key we will use is:
* `2B 7F 15 16 28 AE D2 A6 AB F7 15 88 08 CF 4F 3C`

From this we will generate the rest of the keys

We start by placing the key into a matrix:
```
+===+===+===+===+
|2B |28 |AB |08 |
+===+===+===+===+
|7F |AE |F7 |CF |
+===+===+===+===+
|15 |D2 |15 |4F |
+===+===+===+===+
|16 |A6 |88 |3C |
+===+===+===+===+
```

Now, when we start, we start with the first column and the column 3 columns ahead of it (column 4 in this case). With these 2 columns we will generate the next column that will be appended onto the matrix. After doing this 4 times in this example, we will have successfully generated another key.

There is one special rule though in the generation process, whenever you are using the first column adn 4th column of a single key, a number of extra steps must be done.

To start, take the first column and the 4th column as such:
```
+===+       +===+
|2B |       |08 |
+===+       +===+
|7F |       |CF |
+===+       +===+
|15 |       |4F |
+===+       +===+
|16 |       |3C |
+===+       +===+
```

Because this is the 1st and 4th column of the key we need to apply a shift to the 4th collumn followed by running the column through the S-box. You can see the S-box in the encryption section. Like the encryption step involving the S-box the procdure is to take the first value as the row and the second number as the column to determine what value is substituted in. Taking the first value 08 means to look in the S-box on row 0, column 8 for the value to replace it with. After shifting and then substituting the results through the s-box you will have the following results:

```
//post-rotate
+===+
|CF |
+===+
|4F |
+===+
|3C |
+===+
|08 |
+===+
//post s-box substitution
+===+
|8A |
+===+
|84 |
+===+
|EB |
+===+
|30 |
+===+
```

From here we then XOR the 1st column with our newly manipulated 4th column. In addition we also XOR this value with an Rcon column value. The Rcon we choose is always the farthest left column, which after it has been used is removed from the Rcon table. This way we always use a different Rcon value every time we have this situation with the 1st and 4th column of the same key

The Rcon table looks as follows:


![Rcon Table](https://hsto.org/getpro/habr/post_images/631/e45/54f/631e4554f8368120368715b029c0f10b.jpg)


Our equation to XOR all together looks as follows:

```
+===+       +===+       +===+
|2B |       |8A |       |01 |
+===+       +===+       +===+
|7F |       |84 |       |00 |
+===+  XOR  +===+  XOR  +===+
|15 |       |EB |       |00 |
+===+       +===+       +===+
|16 |       |30 |       |00 |
+===+       +===+       +===+
```

Once these values have been XORd together, this newly generated column is appended to the end of our original matrix as such:

```
+===+===+===+===++===+
|2B |28 |AB |08 ||A0 |
+===+===+===+===++===+
|7F |AE |F7 |CF ||FB |
+===+===+===+===++===+
|15 |D2 |15 |4F ||FE |
+===+===+===+===++===+
|16 |A6 |88 |3C ||26 |
+===+===+===+===++===+
```

Now we take the 2nd column and the now just added new column. Note that we are choosing this because we are simply incrementing along the columns from the last cycle ?(last cycle was colums 1 and 4, now we are using 2 and 5)

Since this iteration does not use 2 columns from the same key (column 5 is part of the partialy generated 2nd key) We do not need to do the intermediate rotate and s-box substitution steps. Note the next time you will need to do the extra rotate and s-box substitution steps in this example would be when we have iterated to using column 5 and column 8.

In this simpler mode we simply take column 2 and column 5 as such:

```
    +===+       ++===+
    |28 |       ||A0 |
    +===+       ++===+
    |AE |       ||FB |
    +===+       ++===+
    |D2 |       ||FE |
    +===+       ++===+
    |A6 |       ||26 |
    +===+       ++===+
```

And then XOR them together

```
    +===+       ++===+
    |28 |       ||A0 |
    +===+       ++===+
    |AE |       ||FB |
    +===+  XOR ++===+
    |D2 |       ||FE |
    +===+       ++===+
    |A6 |       ||26 |
    +===+       ++===+
```

The resulting new column is then appended back to our original matrix as such:

```
+===+===+===+===++===+===+
|2B |28 |AB |08 ||A0 |88 |
+===+===+===+===++===+===+
|7F |AE |F7 |CF ||FB |55 |
+===+===+===+===++===+===+
|15 |D2 |15 |4F ||FE |2C |
+===+===+===+===++===+===+
|16 |A6 |88 |3C ||26 |80 |
+===+===+===+===++===+===+
```

This loop then continues until all required keys have been generated
