# RSA
RSA is an asymettric cryptographic algorithm by Ronal Rivest, Adi Shami and Leonard Adleman. RSA is an implementation based on Diffie & Hellman public key cipher ideas. The implementation allows each user to have their own keyset and allows simpler distribution of those keys between members. Each member has a public and private key. The public key is what is shared to others, and is used to encrypt the messages using RSA. The private key is kept secret by the key-pair owner and is used to decrypt the message.

The idea behind RSA is the use of prime numbers. Prime numbers are numbers that can only be divided by themselves or 1. Generating large prime numbers is incredibly time consuming and is hard to find. This fact alone is what RSA relys on for its security. RSA in this case is "computationaly secure", as the information being encrypted will be obsolete by the time the user can crack it. One of the largest prime numbers: 2^(657,885,161)-1 for example is 17,425,170 digits long, fills a 17MB file and takes 4-6 days to prove it is prime.

Another factor behind RSA is the use of coprime numbers. Coprime numbers are two numbers that's greatest common divisor is 1 ( GCD(A,B)==1 ). Note that A and B do not have to be prime numbers to be coprime numbers.

## Algorithm
The RSA algorithm is quite simple. First you must generate a public and private key. This is done by answering a number of equations as follows:

1. Choose 2 prime numbers P and Q
2. Calculate N by multiplying P and Q ( N = P*Q )
3. Calculate the totient T by multiplying 1 less of P with 1 less then Q ( T = (P-1)*(Q-1) )
4. Choose a 3rd prime number E which is greater then prime P or prime Q but less then T. E when compared with T must also have a GCD of 1 ( E where E > (P || Q ) && E < T && GCD(E,T)==1 )
5. Calculate D by stabilizing the following congruency: E * D = 1(mod T). This can be calculated through manual trial and error or using the Extended Euclidean Algorithm
6. After this, test that E and D are coprime

After completing this you will have all the components needed for creating a public and private key. In the most simplistic form, E and N make up the public key. D and N make up the private key. This information alone is all that is needed to carry out the RSA algorithm, however for transferring this information a standard ASN.1 format is used most commonly when circulating private and public keys.

## Encryption
Encryption is simply done by running each segment of data through an equation using the public key components created above, and retrieving the ciphertext segments.

The general equation is as follows:
```
C = M^E(mod N)
```
C is the ciphertext. M is the plaintext. E and N come from the public key generated in the above section. Note that size of M matters when encrypting with RSA. The value of M as an integer cannot be larger then the value of N as it will otherwise be lost (a consequence of modular arithmetic). Thus when encryption plaintext, some earlier calculations must be done to ensure the M value is small enough.

Given ASCII characters map to 8-bit this means that N must always be larger then 2^8 = 256. This is not always possible if you are using smaller prime numbers such as P = 5 and Q = 11, N = 55. In this case the ASCII letter need to be cut up into smaller segments. The segment size can be calculated with: Math.floor(Log(N)/Log2). This will give the number of bits that the message must be broken into and the maximum number of plaintext bits that can be used in M.

Using our example of P = 5 and Q = 11, we know N = 55. Using the above mentioned equation we know that Math.floor(Log(55)/Log(2)) = 5. That means whatever plaintext value M is, cannot be larger then the maximum value that can be represented by 5 bits or 2^5 = 32. We can also prove this limitation: if we used 6 bits the maxiumum value would be 2^6 = 64 which is larger then N = 55, thus 5 is the maximum we can use.

After the encryption, you will need to do an extra calculation to determine how large the encrypted values should be. This is important as some of the encrypted values may need padding, so that they can be properly parsed at decrytpion. To determine how many bits should be in the encrypted value use the equation Math.ceil(Log(N)/Log(2)). Using our example Math.ceil(Log(55)/Log(2)) = 6. So with our encrypted values in binary, ensure that they are left padded to 6-bits before doing any appending actions of the ciphertext bits.

## Decryption
Decryption is done in the exact same way as Encryption except with the inverse equation using the private key components.

```
M = C^D(mod N)
```
C is the ciphertext. M is the plaintext. D and N come from the private key generated from earlier. Like encryption the size of N matters when decrypting with RSA. It differs slightly from encryption though as the encrypted values can represent inclusively up to the value N. Thus our equation for determining how to parse apart the ciphertext segments differs slightly. To determine the the number of bits to represent the ciphertext we use the equation Math.ceil(Log(N)/Log(2)). Using N = 55, we know that the ciphertext must be broken up into Math.ceil(Log(55)/Log(2)) = 6 bit segments.

Like encryption, after decryption you can use Math.floor(Log(N)/Log(2)) to calculate how many bits should be in the plaintext value. Left-pad the value until it is the length calculated, and then append the values together. If the message received is ASCII characters, you can now process the binary in 8-bit segments. Note that to work with N during encryption there may be extra padding on the end. Read in the ASCII data from left to right, parsing it into 8-bit segments. If you reach remaining data, which should all be zeros, simply drop it, before converting the binary to characters.

## Example
Lets take the simple example of the message "YO" and encrypt it. First we need to generate all of our key components. Lets use P = 5 and Q = 11. Therefor:
```
N = P * Q = 5 * 11 = 55
T = (P-1)*(Q-1) = 4 * 10 = 40
E = 13 (note E could be other numbers that meet the requirements but we chose this one to keep it simple)
```
Next we need to calculate D. This is done using the Extended Euclidean Algorithm. See the bottom of this page for additional resources on the Extended Euclidean Algorithm. It looks like this:
```
A = 13 B = 40
1) 40 = 3*13 + 1
2) 13 = 13*1 + 0
----------------
1 = 40 - 3*13
  = (1)40 + (-3)13

(-3) + 40 = 37
D = 37
```
We can test this is correct by checking the congruency of the equation:
```
E * D = 1(mod T)

13 * 37 = 1(mod 40)

(13*37) - 1 = 480 which is a factor of 40
```
So D = 37

We now have all of the components. Are public key is comprised of E and N (13, 55) and our private key is D and N (37, 55)

Now to encrypt our message:
### Encrypt
To encrypt we use the encryption algorithm above, but first we must convert this message to binary. In binary the message "YO" looks as follows:

```
010110001 01010001
   Y         O
```
Normal ASCII uses 8-bits which is much larger then our N value. Thus we must calculate the maximum that can be used for our N value. By calculating Math.floor(Log(55)/Log(2)) we get 5. So our M value must be 5 bits long. Recutting our message into 5-bit segments looks like this
```
01011 00101 01000 10000
```
Note that we have appended 4 zero's onto the end of the last segment. This is to make it even out into 5 bits. for RSA, POST appending is the most effective way to even out the segments. When decryption occurs we will be able to determine what to remove as the message will be regrouped into 8-bit segments as before. When this happens we will have leftovers and know to drop it.

Now we take the integer values those binary numbers represent and run them through the equation. After encrypting, we left pad the binary until it contains Math.ceil(Log(55)/Log(2)) = 6 bits. It looks like this:
```
01011 00101 01000 10000
  11    5     8      16

11^13(mod 55) = 11 = 001011
5^13(mod 55) = 15 = 001111
8^13(mod 55) = 28 = 011100
16^13(mode 55) = 26 = 011010
```

We now have our encrypted message:
```
001011001111011100011010
```

### Decrypt
To decrypt the above encrypted message we use the D and the N value in another equation. Again using N we can calculated how to split up the binary. Using Match.ceil(Log(55)/Log(2)) = 6 bits. We cutup the binary into 6-bit segments. This looks as follows:
```
001011 001111 011100 011010
```
Using then the decryption equation we can convert these segments back to their plaintext values. Afterwards we calculate Math.floor(Log(55)/Log(2)) = 5 bits to determine how much to pad the plaintext values.
```
001011 = 11 = 11^37(mod 55) = 11 = 01011
001111 = 15 = 15^37(mod 55) = 5 = 00101
011111 = 28 = 28^37(mod 55) = 8 = 01000
011010 = 26 = 26^37(mod 55) = 16 = 1000
```
Now to convert back we append all of these together and parse them into 8-bit ASCII
```
0101100101010001000

01011001 01010001   000
   Y        O     [extra]
```
Note the extra we had talked about that would be appended. We know we can get rid of this because it does not equate to 8-bits of a valid ASCII character and it is also all zeros.

## Python Code Example
Below is a python walkthrough example of an encryption and then decryption using python. Note that this implementation is not the most efficient implementation as it converts the binary representation of the numbers into string, thus allowing each binary value to be accessed at an index. Conversion between binary and binary strings in python is quite trivial, and it does make the below example a bit more legible if you are unfamiliar with bit manipulation actions, but note this is at the expense of performance.

The code sample below includes print statements, copy the the code snippet below and execute with Python3 to get output through each stage of the algorithm

```python
import math
import sys

unencryptedMessage = "Hello World!"

p1 = 5  # prime number 1
p2 = 11  # prime number 2

t = 40  # totient. AKA (p1-1)*(p2-1)
n = 55  # n. AKA p*q - part of public key
e = 13  # e. prime number - part of public key
d = 37  # d. private key


plaintext_message_seg_length = int(math.floor(math.log(float(n), 2)))
encrypted_message_seg_length = int(math.ceil(math.log(float(n), 2)))

print("Plaintext: " + str(plaintext_message_seg_length))
print("Encrypted: " + str(encrypted_message_seg_length))
print("Original Message: " + unencryptedMessage)

# convert the message to all binary bits - padd out to make sure they all are 8 bits long for the character
binaryUnencryptedMessage = ''.join(format(ord(x), '08b') for x in unencryptedMessage)
print(binaryUnencryptedMessage)

# post pad the string to get an even number
while len(binaryUnencryptedMessage) % plaintext_message_seg_length != 0:
    binaryUnencryptedMessage += '0'

print(binaryUnencryptedMessage)

# split it up into segments of plaintext_message_seg_length
unencryptedMessageSegments = list()
for i in range(0, len(binaryUnencryptedMessage), plaintext_message_seg_length):
    unencryptedMessageSegments.append(binaryUnencryptedMessage[i: i + plaintext_message_seg_length])

print(unencryptedMessageSegments)

#encrypt each segment using RSA
encryptedMessageSegments = list()
for i in unencryptedMessageSegments:
    print("------------------")
    print(i)
    segmentInt = int(i, 2)  # converts string to int, interpreting it as in base 2
    print(str(segmentInt) + " - " + bin(segmentInt))
    encryptedSegmentInt = (segmentInt ** e) % n
    print(str(encryptedSegmentInt) + " - " + bin(encryptedSegmentInt))
    encryptedSegmentBinary = format(encryptedSegmentInt, '0' + str(encrypted_message_seg_length) + 'b')
    print(encryptedSegmentBinary)
    encryptedMessageSegments.append(encryptedSegmentBinary)


print("***********************")
print(encryptedMessageSegments)
encryptedMessageBinaryString = ''.join(encryptedMessageSegments)
print(encryptedMessageBinaryString)

encryptedMessageInt = int(encryptedMessageBinaryString, 2)
print(encryptedMessageInt)
print(bin(encryptedMessageInt))


encryptedMessage = encryptedMessageInt.to_bytes(byteorder=sys.byteorder,
                                            length=math.ceil(len(encryptedMessageBinaryString) / 8 ))
print(encryptedMessage)

# -- AT THIS POINT THE MESSAGE IS ENCRYPTED AS A BYTE ARRAY--

number = int.from_bytes(encryptedMessage, byteorder=sys.byteorder, signed=False)
print(number)

print (" ** BEGINNING DECRYPTION **")

binaryEncryptedMessage = str(bin(number))[2:]
print(binaryEncryptedMessage)

# pre pad encrypted until is appropriate length to be cut up
while len(binaryEncryptedMessage) % encrypted_message_seg_length != 0:
    binaryEncryptedMessage = '0' + binaryEncryptedMessage

# cut into decryptable segments
encryptedMessageSegments = list()
for i in range(0, len(binaryEncryptedMessage), encrypted_message_seg_length):
    encryptedMessageSegments.append(binaryEncryptedMessage[i: i + encrypted_message_seg_length])

print(encryptedMessageSegments)

unencryptedSegments = list()
for i in encryptedMessageSegments:
    print("------------")
    segmentInt = int(i, 2)  # converts string to int, interpreting it as in base 2
    print(i)
    print(str(segmentInt) + " - " + bin(segmentInt))
    unencryptedSegmentInt = int((segmentInt ** d) % n)
    print(unencryptedSegmentInt)

    # left pad with 0 to return segment to decrypted segment length
    unencryptedSegmentBinary = format(unencryptedSegmentInt, '0' + str(plaintext_message_seg_length) + 'b')
    print(unencryptedSegmentBinary)
    unencryptedSegments.append(unencryptedSegmentBinary)

print(unencryptedSegments)
joinedSegments = ''.join(unencryptedSegments)
print(joinedSegments)


letters = list()
for i in range(0, len(joinedSegments), 8):
    letters.append(joinedSegments[i: i + 8])

print(letters)

plainMessage = ""
for letter in letters:
    letterInt = int(letter, 2)
    character = chr(letterInt)
    plainMessage += character

print(plainMessage)
```

## Extended Euclidean Algorithm
Additional Resources:

* https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm
* https://en.wikibooks.org/wiki/Algorithm_Implementation/Mathematics/Extended_Euclidean_algorithm
* http://www.mast.queensu.ca/~math418/m418oh/m418oh04.pdf
