Write-ups for challenges completed in niteCTF

# import rAnDoM.md

## Flag: `nite{br0_y4pp1ng_s33d_sl1pp1ng}`

## Solution
- The challenge makes it obvious that the `random` Python module is not cryptographically secure, and opts to use the mofied flag as the seed for `random`.
- `chall.py` uses the module `rAnDoM.py` to generate 6 pseudo-random 30 bit integers using 4-byte chunks of the flag as the seed. Specifically, the numbers at the index values **0, 1, 2, 227, 228, 229** are used, which seems specific. Looking up how to reverse the rando m module with keywords `227 228 229` returns [this blog-post](https://stackered.com/blog/python-random-prediction/#seed-recovery-from-few-outputs), which is the solution to the challenge. The entire solution and the code is given in [this part](https://stackered.com/blog/python-random-prediction/#seed-recovery-from-few-outputs).
- Put the required functions into `randrev.py` and make a script `rAnDrev.py` with the following code:
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
- This code will return the flag when run in the terminal.
