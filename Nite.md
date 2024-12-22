Write-ups for challenges completed in niteCTF

# import rAnDoM.md
This challenge was completed by me during the event.

## Flag: `nite{br0_y4pp1ng_s33d_sl1pp1ng}`

## Solution
1. The challenge makes it obvious that the `random` Python module is not cryptographically secure, and opts to use the mofied flag as the seed for `random`.
2. `chall.py` uses the module `rAnDoM.py` to generate 6 pseudo-random 30 bit integers using 4-byte chunks of the flag as the seed. Specifically, the numbers at the index values **0, 1, 2, 227, 228, 229** are used, which seems specific. Looking up how to reverse the rando m module with keywords `227 228 229` returns [this blog-post](https://stackered.com/blog/python-random-prediction/#seed-recovery-from-few-outputs), which is the solution to the challenge. The entire solution and the code is given in [this part](https://stackered.com/blog/python-random-prediction/#seed-recovery-from-few-outputs).
3. Put the required functions into `randrev.py` and make a script `rAnDrev.py` with the following code:
```
import randrev


def seed_to_string(seed1, seed2):
    def toString(h_num1, h_num2):
        try:
            return bytearray.fromhex(h_num1).decode()
        except UnicodeDecodeError:
            return bytearray.fromhex(h_num2).decode()

    return toString(hex(seed1)[2:], hex(seed2)[2:])

def get_seed(enc_bunch: list[6]):
    S = {[0, 1, 2, 227, 228, 229][k]: randrev.untemper(enc_bunch[k]) for k in range(6)}

    I_227_, I_228 = randrev.invertStep(S[0], S[227])
    I_228_, I_229 = randrev.invertStep(S[1], S[228])
    I_229_, I_230 = randrev.invertStep(S[2], S[229])

    I_228 += I_228_
    I_229 += I_229_

    # two possibilities for I_230
    seed1 = randrev.recover_Kj_from_Ii(I_230, I_229, I_228, 230)
    seed2 = randrev.recover_Kj_from_Ii(I_230+0x80000000, I_229, I_228, 230)

    return seed_to_string(seed1, seed2)


if __name__ == '__main__':
    enc = [int(i) for i in list(open("enc line-by-line").read().split())]
    chunks = [enc[i:i+6] for i in range(0, len(enc), 6)]
    # print(chunks[1])
    flag = [get_seed(chunks[i]) for i in range(len(chunks))]
    [print(i, end='') for i in flag]
    print()
```
4. This code will return the flag when run in the terminal.

> Rest of the challenges were solved after the event

# RSAabc

## Flag: `nite{quICklY_grab_the_codE5_sgOqkA}`

## Solution
1. The challenge encrypts the flag using alphabet reversing, greek alphabet mapping and RSA. The RSA encryption is weak because one of the prime numbers lies in the range of (2^5, 2^25), which is too small.
2. The encryption works for every character in the flag. The characters with composite ASCII values are only reversed. The characters with prime ASCII values are encrypted using RSA, then transformed to greek alphabets. This means the characters which remain in english are only reversed, while the ones in greek have been encrypted with RSA.
3. The program masks the ciphertext in the output by editing one bit before writing, but this change is completely reversible.
4. The following code decrypts the encoded flag:
```
import sympy as sp
import ciphermod # Importing the given program

def int_to_string(n):
    return bytes.fromhex(hex(n)[2:]).decode()

def rsa_decrypt(ciphertext: int, private_key: (int, int)) -> str:
    n, d = private_key
    message_as_int = pow(ciphertext, d, n)
    message = int_to_string(message_as_int)
    return message

flag = ""

enc = "mrgπeτfΟΔςoΝeηiδyegsλexlwVαehιΠπμZe"
ct = [<copy the list ct from out.txt>]
nlist = [<copy the list n from out.txt>]

english =   ['A', 'a', 'B', 'b', 'C', 'c', 'D', 'd', 'E', 'e', 'F', 'f', 'G', 'g', 'H', 'h', 'I', 'i', 'J', 'j', 'K', 'k', 'L', 'l', 'M', 'm', 'N', 'n', 'O', 'o', 'P', 'p', 'Q', 'q', 'R', 'r', 'S', 's', 'T', 't', 'U', 'u', 'V', 'v', 'W', 'w', 'X', 'x', 'Y', 'y', 'Z', 'z']
language =  ['Α', 'α', 'Β', 'β', 'Σ', 'σ', 'Δ', 'δ', 'Ε', 'ε', 'Φ', 'φ', 'Γ', 'γ', 'Η', 'η', 'Ι', 'ι', 'Ξ', 'ξ', 'Κ', 'κ', 'Λ', 'λ', 'Μ', 'μ', 'Ν', 'ν', 'Ο', 'ο', 'Π', 'π', 'Θ', 'θ', 'Ρ', 'ρ', 'Σ', 'ς', 'Τ', 'τ', 'Υ', 'υ', 'Ω', 'ω', 'Ψ', 'ψ', 'Χ', 'χ', 'Υ', 'υ', 'Ζ', 'ζ']

for i, ch in enumerate(enc):
    print(f"Now computing letter {i+1}/{len(enc)}")
    if (ch in english):
        flag += ciphermod.reverse_alphabet(ch)
    else:
        # Reverse ciphertext masking
        lan = ch
        eng = english[language.index(lan)]
        ciphertext = ciphermod.googly(ct[i], ct[i].bit_length() - ord(eng))

        # Calculate the private exponent d
        n = nlist[i]
        e = 65537
        for divr in range(2**5 + 1, 2**25, 2):
            if (n % divr == 0):
                p = divr
                q = n // divr
                break
        phi_n = (p - 1) * (q - 1)
        d = sp.mod_inverse(e, phi_n)
        privkey = (n, d)

        # Decrypt the character
        message = rsa_decrypt(ciphertext, privkey)
        flag += message


print(f"FLAG FOUND: {flag}")
# OUTPUT: nitevquICklYvgrabvthevcodE5vsgOqkAv
```
4. The function `reverse_alphabet()` outputs `e` if the input is `_`, `{` or `}`. The reverse of `e` is `v`. This means that the `v`s in the output of the code above are one of these special characters, which can easily be substituted:
`nite{quICklY_grab_the_codE5_sgOqkA}`

# La Casa de Papel

## Flag: `nite{El_Pr0f3_0f_Prec1s10n_Pl4ns}`

## Solution
1. Use to `Practice Convo` to encrypt any messagelike `REPLACE`. Decode the base64 encoding of the md5 hash.
2. Use length extension attacks to append `Bob` to the md5 hash
```
secret + b'REPLACE' = cdb9d8aa74b9b1298f5f8e2712b2519b
secret + b'Bob' = b4e0a802428cb35f69c0e952d96172d6
```
3. Use this to get the password from Alice with the name `Bob`: `G0t_Th3_G0ld_B3rl1nale`
4. `Crack the Vault` with the password: `nite{El_Pr0f3_0f_Prec1s10n_Pl4ns}`
