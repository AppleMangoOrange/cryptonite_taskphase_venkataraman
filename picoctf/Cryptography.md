# C3

**Flag:** `picoCTF{adlibs}`

## Approach

- Download the encoder and ciphertext. The first few lines are used to take input from the user through the terminal. Next, we have 2 strings. The name suggests they are used as lookup tables, which matches the description of the challenge:
> This is the Custom Cyclical Cipher!

- Running the program gives an idea of how it works. `test` returns `nfOB` and `abababab` returns `OBtBtBtB`. The output has the same length as the input, and looking at the code we can see that each character corresponds to a character in the same position (index) of the input string.

- Let's look at the encoder again. The `for` loop moves through each character of the input. It stores the index of both the current (`cur`) and previous (`prev`) characters. This means the that any character in the output only depends on two factors, the corresponding input character index and the one before it. This is reflected in the `abababab` test input that we tried. The first charater in that test input was different to the pattern because of the line just before the for loop; `prev = 0`. This basically "assumes" the zeroeth character's lookup index to be 0.

```
prev = 0
for char in chars:
  cur = lookup1.index(char)
  out += lookup2[(cur - prev) % 40]
  prev = cur
```

- Now let's take a look at the loopup tables. They are both a "list" of 40 characters each.
```
lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
lookup2 =   "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"
```

- If the previous character's index were 0 (which is always true for the first character), it would just copy the same index. This is because of a specific line in the `for` loop. In python, if the dividend in a modulo operation is negative, the result subtracted from the divisor. This means `55 % 40`, which would usually result in -15, returns 25 in python since `40 - 15 = 25`. This means we can easily reverse the operation done in this encoder.

`out += lookup2[(cur - prev) % 40]`
- This can easily be understood by modifying the encoder code a bit. The first output character has the same index as `prev = 0`. The next character is calculated using `(18 - 33) % 40 = 25`. This is continued till the last character of the input.
  - Input
    ```
    import sys
    chars = ""
    from fileinput import input
    # for line in input():
    #   chars += line
    chars = "test"
    print(repr(chars)) #
    
    lookup1 = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
    lookup2 =   "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"
    
    out = ""
    
    prev = 0
    for char in chars:
      cur = lookup1.index(char)
      print(f"{repr(char)}: lookup1[{cur}]", end='\t') #
      out += lookup2[(cur - prev) % 40]
      print(f"{lookup2[(cur - prev) % 40]}: lookup2[{(cur - prev) % 40}]") #
      prev = cur
    
    sys.stdout.write(out)
    print() #
    ```
  - Output
    ```
    'test'
    't': lookup1[33]        n: lookup2[33]
    'e': lookup1[18]        f: lookup2[25]
    's': lookup1[32]        O: lookup2[14]
    't': lookup1[33]        B: lookup2[1]
    nfOB
    ```
- We can now write a decoder program. Keep in mind that our input here is encoded and the output is plain text. 
  - For the first character `n`, its index in the encoded loopup table is 33. We also take `prev = 0` in this case. Now, `(0 + 33) % 40` returns 33, which is the index for `t` in the plain lookup table. Now we store 33 in `prev`.
  - For the second character `f` (index = 25), we again calculate `(33 + 25) % 40`, which returns 18, the index of `e`. Now we store 18 in `prev`.
  - This goes on till the last character of the encoded string.
```
encoded = "nfOB"
plain = ""
lookup_plain = "\n \"#()*+/1:=[]abcdefghijklmnopqrstuvwxyz"
lookup_encoded = "ABCDEFGHIJKLMNOPQRSTabcdefghijklmnopqrst"

prev = 0
for char in encoded:
    cur = lookup_encoded.index(char)
    plain += lookup_plain[(prev + cur) % 40]
    prev = (prev + cur) % 40

print(plain)
```
- As expected, the output is `test`. Now we can input the given ciphertext in our decoder and get the output, which is clearly a code written in Python 2.
```
#asciiorder
#fortychars
#selfinput
#pythontwo

chars = ""
from fileinput import input
for line in input():
    chars += line
b = 1 / 1

for i in range(len(chars)):
    if i == b * b * b:
        print chars[i] #prints
        b += 1 / 1


```
- This seems to be a piece of code which only prints the input characters whose index is a cube of a natural number (1, 8, 27...). Looking at `#selfinput`, we know that the program takes itself as its input. Doing that returns this:
```
a
d
l
i
b
s
```

- **The flag is:** `picoCTF{adlibs}`.


## New concepts

1. Working of the `fileinput.input()` function in python


## Incorrect methods tried

- In the second stage, after looking at `#selfinput`, I thought the input might be the same ciphertext. This returns `LgHDPt` (ignoring `\n`). The flag `picoCTF{LgHDPt}` was incorrect, though.

- `#selfinput` could also mean the program takes itself as its input. This gives the output `adlibs` (ignoring `\n`). I put this through the original encoder which returned `ODIrnR`. (an unnecessary extra step)


&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;


# Custom encryption

**Flag:** `picoCTF{custom_d2cr0pt6d_751a22dc}`

## Approach

- The code is clearly a Python module which is meant to be used for encryption. Running the program with a few test inputs shows that the output is a list of numbers as long as the input message.

- The `generator()` function has a simple output of `(g ^ x) mod p`. This has a similar look to `ciphertext = (message ^ exponent) mod (pq)` from RSA excryption.

- The `test()` function seems to have an example usage of the library. Let's analyse it.
  - The `message = sys.argv[1]` means that the message to be encrypted has to be entered as a command line argument. We can modify it temporarily: `message = "<plain text>"`.
  - It has the following inputs:
    - 2 prime numbers p (= 97) and g (= 31)
    - `plain_text`: The text message to be encrypted
    - `text_key`: The text key used to encrypt `plain_text`
  - The function generates 2 random numbers a and b, which are random numbers upto 10 less than p and q respectively.
  - Then, it calculates two values: `(((g**b) mod p)**a) mod p` and `(((g**a) mod p)**b) mod p`. If these two are same, they become the `shared_key`.
  - Now, the `plain_text` is XOR-ed with the `text_key` using `dynamic_xor_encrypt()`, which is further encrypted using the `encrypt()` function.

- The encrypted flag has the same output format as the `test()` function, with the random values `a` and `b` included. This and the included values suggest that the values of `p` and `g` also remain unchanged. Solving for `key` with these values gives **93**.

- The `dynamic_xor_encrypt()` is the step 1 of our cipher. It is a [simple XOR cipher](https://www.dcode.fr/xor-cipher) and repeats the key if it is shorter than the given message. Notably, it reverses the message. Step 2 is `encrypt()`, which does `character * shared_key * 311` for every character of its input. We already know the `shared_key` is probably 93, so let's give it a go with that.

- This is a simple program that reverses the encryption:
```
cipher = [ # As given in the challenge
    260307,
    ..., # Removed numbers in between to maintain readability
    491691
]

# Undo the work done by `encrypt()` using the known values
semi_cipher = [(i // (93 * 311)) for i in cipher]

# Undo the XOR cipher using a guessed key `trudeau`
for i in range(len(semi_cipher)-1, -1, -1):
    print(chr(semi_cipher[i] ^ ord("trudeau"[i % 7])), end='')

print()
```

- The first step just reverses the multiplication done in the `encrypt()` function. The second step reverses the XOR encryption by guessing that the same plain-text key was used. Since XOR is reversible using XOR, this is a trivial matter. We can simply XOR the characters of the encrypted message with those of the plain-text key.

- The output reveals the key along with the `picoCTF{}` wrapper: 
```
picoCTF{custom_d2cr0pt6d_751a22dc}
```

## New concepts
1. Modular arithmetic

## References
- https://ctf101.org/cryptography/what-is-rsa/
- https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic
- https://www.dcode.fr/xor-cipher


&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# miniRSA

**Flag:** `picoCTF{n33d_a_lArg3r_e_ccaa7776}`

## Approach

- The given data has a small public exponent and relatively small ciphertext. This means we can apply the cube root attack (since e = 3) to the ciphertext. This gives:
`13016382529449106065894479374027604750406953699090365388203708028670029596145277`
- Then, we can conver this number to hexadecimal:
`0x7069636f4354467b6e3333645f615f6c41726733725f655f63636161373737367d`
- This can then be separated into groups of 2 digits (1 byte each) and converted to `utf-8` to get the flag.
- All of this can just be done by the RsaCtfTool python script.
```
Decrypted data :
HEX : 0x7069636f4354467b6e3333645f615f6c41726733725f655f63636161373737367d
INT (big endian) : 13016382529449106065894479374027604750406953699090365388203708028670029596145277
INT (little endian) : 14498533606165685119718956852327439022714916090771037807901302412009841646332272
utf-8 : picoCTF{n33d_a_lArg3r_e_ccaa7776}
STR : b'picoCTF{n33d_a_lArg3r_e_ccaa7776}'
```

## References

- https://ctf101.org/cryptography/what-is-rsa/
- https://github.com/RsaCtfTool/RsaCtfTool
- https://www.dcode.fr/unicode-coding
- https://www.mathsisfun.com/calculator-precision.html

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# rotation

**Flag:** `picoCTF{r0tat1on_d3crypt3d_949af1a1}`

## Approach

- Apply brute force decrption of the Affine cipher knowing that the flag starts with `picoCTF`

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ReadMyCert

**Flag:** `picoCTF{read_mycert_41d1c74c}`

## Approach

- Opening the .csr file using the certificate manager shows its name, which is the flag

## New concepts

1. CSR and CSR files

## References

- https://en.wikipedia.org/wiki/Certificate_signing_request

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# HideToSee

**Flag:** `picoCTF{atbash_crack_7142fde9}`

## Approach

- Use a steganographic decoder on the image to get `krxlXGU{zgyzhs_xizxp_7142uwv9}`
- Use [an Atbash Cipher decoder](https://www.dcode.fr/atbash-cipher) to get the flag

## Incorrect methods tried

- Going to the website mentioned in the image

## References

- 

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

