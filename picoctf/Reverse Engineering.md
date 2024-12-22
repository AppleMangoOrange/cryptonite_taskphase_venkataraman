# GDB baby step 1

**Flag:** `picoCTF{549698}`

## Approach

- The `debugger0_a` file downloaded does not have debug info (which can be seen using `file`). The required number is the value of the `eax` register at the end of main.
- To find this, we can make use of the GNU  debugger. `gdb ./debugger0_a`. We can enter our command at the end of `(gdb) `. Running the program with `r` returns an error code.
- We can add a breakpoint to the code using `b main`. Now, when running with `r`, the program stops just before main. With `n` (or `s`), we can move to the end of the main function (since we do not have the source code).
- Now, we can view the value in the `eax` register using `info registers eax`. The third value is the value stored in the register.

## New concepts

1. Debugging using GDB
2. CPU registers

## References

- https://ctf101.org/reverse-engineering/what-is-gdb/
- https://ctf101.org/reverse-engineering/what-is-assembly-machine-code/
- https://www.geeksforgeeks.org/gdb-step-by-step-introduction/

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# GDB baby step 2

**Flag:** `picoCTF{307019}`

## Approach

- Follow the same steps as in [GDB baby step 1](#gdb-baby-step-1)

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# GDB baby step 4

**Flag:** `picoCTF{12905}`

## Approach

- Use Cutter/Rizin to decompile the given binary
- In the `sym.func1` function, we find the line `return (int32_t)arg1 * 0x3269;`
- `3269` in hex is `12095` in decimal

## New concepts

1. De-compiling

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ARMssembly 0

**Flag:** `picoCTF{F5053F31}`

## Approach

- The only file we are given is an armv8-a or AARCH64 assembly code plain text file. (as seen in the first line of the text file)
- Going through the code, we see two funcions; `func1` and `main`. Let's first look at `func1`.
- This seems to be a function with two paramemters. Let's call them `a` and `b`. The code can be understood with the following C-like translation:
```
func1 (a, b):
    sp -= 16b           // Reserve 16b of space on the stack
    *(sp + 12) = w0     // save 'a' at (sp+12)
    *(sp + 8) = w1      // save 'b' at (sp+8)
    w1 = *(sp + 12)     // w1 = a
    w0 = *(sp + 8)      // w0 = b
    if w1 <= w0:        // if (a <= b)
        w0 = *(sp + 8)  // return b
    else:
        w0 = *(sp + 12) // return a
    sp += 16b
```
- Let's do the same for `main`:
```
main (argc, argv):
    sp -= 48b; *sp = x29; *(sp + 8b) = x30      // store values of the SP and LR in the stack
    x29 = sp            // stores new sp value at x29
    *(sp + 16) = x19    // save x19 (to be restored later)
    *(x29 + 44) = w0    // *(x29 + 44) = argc
    *(x29 + 32) = x1    // *(x29 + 32) = argv // argv = array of strings {"./program", "a", "b"}
    x0 = *(x29 + 32)    // x0 = argv = &argv[0] // *x0 = argv[0]
    x0 += 8             // x0 = argv+8 = &argv[1] // size of pointer = 8 (size of memory address)
    x0 = *x0            // x0 = argv[1] = "a"
    x0 = int(x0)        // x0 is now an int // x0 = a
    w19 = w0            // w19 = a
    x0 = *(x29 + 32)    // x0 = argv
    x0 += 16            // x0 = argv+16 = &argv[2]
    x0 = *x0            // x0 = argv[2] = "b"
    x0 = int(x0)        // x0 = b
    w1 = w0             // w1 = b
    w0 = w19            // w0 = a
    call func1 > w0     // w0 = func1(a, b)
    w1 = w0             // w1 = w0 = func1(a, b)
    x0 = &LC0[..12]
    x0 += &LC0[12..]    // Add pointer to LC0 in x0
    call printf()       // Print output
    return 0
    x19 = *(sp + 16)    // Restore x19
    x29 = *sp; x30 = *(sp + 8b); sp += 48b      // Restore x29, x30 and sp
    ret                 // Return from main
```
- Now, we can decompile this assembly code:
```
func1 (a, b):
    if a <= b:
        return b
    else:
        return a

main (argc, argv):
    printf("Result: %ld\n", func1(argv[1], argv[2]))
    return 0
```
- This means the program just returns the bigger of the 2 input program paramenters.

## New concepts

1. Working of assembly code
2. Working of commands in AARCH64 assembly
3. `argc` and `argv`

## References

- https://www.geeksforgeeks.org/command-line-arguments-in-c-cpp/ (`argc` and `argv`)
- https://stackoverflow.com/questions/64638627/explain-arm64-instruction-stp
- https://developer.arm.com/documentation/102374/0102/Loads-and-stores---load-pair-and-store-pair
- https://stackoverflow.com/questions/78880407/struggling-with-using-the-ldr-instruction-in-arm-assembly
- https://stackoverflow.com/questions/66107147/instruction-writeback-in-arm
- https://developer.arm.com/documentation/dui0801/h/A64-General-Instructions/ADRP
- https://stackoverflow.com/questions/8236959/what-are-sp-stack-and-lr-in-arm
- https://pages.cs.wisc.edu/~markhill/restricted/arm_isa_quick_reference.pdf
- https://godbolt.org/ (instructions quick reference)
- https://learn.arm.com/learning-paths/servers-and-cloud-computing/exploiting-stack-buffer-overflow-aarch64/
- https://developer.arm.com/documentation/107829/0200/
- https://developer.arm.com/documentation/ddi0602/2022-09/Base-Instructions?lang=en (AARCH64 instructions)
- https://valsamaras.medium.com/arm-64-assembly-series-load-and-store-6bfe9c1d1896
- https://www.tutorialspoint.com/c_standard_library/c_function_atoi.htm (`atoi()`)

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ARMssembly 1

**Flag:** `picoCTF{000000e8}`

## Approach

- Here is the entire assmebly code translated and with comments:
```
func (a):
    sp -= 32b
    [sp+12] = w0        // = a
    w0 = 87
    [sp+16] = w0        // = 87
    w0 = 3
    [sp+20] = w0        // = 3
    w0 = 3
    [sp+24] = w0        // = 3
    w0 = [sp+20]        // w0 = 3
    w1 = [sp+16]        // w1 = 87
    w0 = w1 << w0       // 87 = 1010111, w0 = 1010111000 = 696
    [sp+28] = w0        // = 696
    w1 = [sp+28]        // w1 = 696
    w0 = [sp+24]        // w0 = 3
    w0 = w1 // w0       // w0 = 696 // 3 = 232
    [sp+28] = w0        // = 232
    w1 = [sp+28]        // w1 = 232
    w0 = [sp+12]        // w0 = a
    w0 = w1-w0          // w0 = 232 - a
    [sp+28] = w0        // = 232 - a
    w0 = [sp+28]
    sp += 32b

main (argc, argv = {./program, a}):
    sp -= 48b; [sp] = x29; [sp + 8b] = x30
    x29 = sp+0
    [x29+28] = w0       // = argc
    [x29+16] = x1       // = argv
    x0 = [x29+16]       // x0 = argv = &argv[0]
    x0 += 8             // x0 = &argv[1]
    x0 = int(x0)        // x0 = a
    [x29+44] = w0       // = a
    w0 = [x29+44]       // w0 = a
    func(a) > w0        // w0 = 232 - a
    if (w0 != 0):
        x0 = &LC1[..12]
        x0 += &LC1[12..]    // Add pointer to LC1 in x0
        call puts()         // Print lose condition
    else:
        x0 = &LC0[..12]
        x0 += &LC0[12..]    // Add pointer to LC0 in x0
        call puts()         // Print win condition
    x29 = *sp; x30 = *(sp + 8b); sp += 48b
```

## New concepts

1. `lsl` bitwise shift instruction

## References

- https://developer.arm.com/documentation/ddi0602/2022-09/Base-Instructions?lang=en
- https://courses.cs.washington.edu/courses/cse469/19wi/arm64.pdf
- https://multidec.web-lab.at/base-converter.php

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ARMssembly 2

**Flag:** `picoCTF{9c174346}`

## Approach

- Convert the file to a link file using `aarch64-linux-gnu-as -o <output> <input>`
- Open the `.o` file in Rizin/Cutter. The Ghidra decompiler is the easier option here:
```
int32_t func1(int64_t arg1)
{
    int64_t var_14h;
    int64_t var_8h;
    
    // [01] -r-x section size 140 named .text
    var_8h._0_4_ = 0;
    for (var_8h._4_4_ = 0; var_8h._4_4_ < (uint32_t)arg1; var_8h._4_4_ = var_8h._4_4_ + 1) {
        var_8h._0_4_ = (int32_t)var_8h + 3;
    }
    return (int32_t)var_8h;
}

void main(int argc, char **argv)
{
    undefined4 uVar1;
    int64_t arg1;
    
    arg1 = atoi(argv[1]);
    uVar1 = func1(arg1);
    printf("Result: %ld\n", uVar1);
    return;
}
```
- As we can see, the function runs a loop to add 3 to 0 `a` times, where a is the CLI paramemter we give. This means it just outputs `a*3`. `a` here is `3736234946`, so `a*3` is `11208704838`.
- Converting this to binary, we get `29c174346`. Since we should only take 32 bits, the number we want is `9c174346`.

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ARMssembly 3

**Flag:** `picoCTF{00000036}`

## Approach

- - Convert the file to a link file using `aarch64-linux-gnu-as -o <output> <input>`
- Open the `.o` file in Rizin/Cutter. The Ghidra decompiler is the easier option here. `main` passes the CLI parameter (say `a`) to `func1`. `func2` just adds 3 to its input. Let's look at `func1`:
```
uint32_t func1(int64_t arg1)
{
    int64_t var_30h;
    int64_t var_28h;
    int64_t var_14h;
    uint32_t uStack_4;
    
    uStack_4 = 0;
    // [01] -r-x section size 176 named .text
    for (var_14h._0_4_ = (uint32_t)arg1; (uint32_t)var_14h != 0; var_14h._0_4_ = (uint32_t)var_14h >> 1) {
        if (((uint32_t)var_14h & 1) != 0) {
            uStack_4 = func2((uint64_t)uStack_4);
        }
    }
    return uStack_4;
}
```
- This can be further simplified as follows:
```
    int var_14h;
    unsigned int uStack_4;
    for (var_14h = 597130609; var_14h != 0; var_14h = var_14h >> 1) {
        if ((var_14h & 1) != 0) {
            uStack_4 += 3;
        }
    }
```
- This program can be run in C to get the output of `54`, which is `36` in hexadecimal.

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# ARMssembly 4

**Flag:** `picoCTF{c1cc042c}`

## Approach

- Decompile the code using Rizin/Cutter as before, but only concentrate on the functions that would actually be called:
```
main (64_a): // 3251372985
    printf("Result: %ld\n", func1(u32_a));

func1 (64_a):
    if (u32_a <= 100):
        func3(32_a)
    else:
        func2(u64_(u32_a + 100)) // 3251373085

func2 (64_a):
    if (32_a < 500):
        # func4()
    else:
        func5(u64_(u32_a + 13)) // 3251373098

func5 (64_a):
    return func8(32_a)          // 3251373098

func8 (64_a):
    return (32_a + 2)           // 3251373100

```
- Convert `3251373100` to hexadecimal

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# vault-door-1

**Flag:** `picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_f6daf4}`

## Approach

- Simply write down the characters corresponding to the index position as given in the code.

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# vault-door-3

**Flag:** `picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}`

## Approach

- Write a temporary input string that represents each character (like `0123456789abcdefghijklmnopqrstuv`). Modify the code so that you can see the changes that it makes:
```
...
            buffer[i] = password.charAt(i);
        }
        System.out.println(buffer); // 
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm18g947_u_4_m9r54f");
...
```
- Now input the test string into the program:
```
$ java V*
Enter vault password: picoCTF{0123456789abcdefghijklmnopqrstuv}
01234567fedcba98uhsjqlonmpkritgv
Access denied!
```
- Now that we know how the program jumbles the string, we can easily decipher the flag using the string `s` in the program.

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# vault-door-4

**Flag:** `picoCTF{jU5t_4_bUnCh_0f_bYt3s_8f4a6cbf3b}`

## Approach

- The program simply stores each character of the flag in various bases. We could convert these character individually to get the flag, but there is an easier method.
- To avoid doing manual labour, we can modify the code to do our work for us:
```
        for (int i=0; i<32; i++) {
            // if (passBytes[i] != myBytes[i]) {
            //     return false;
            // }
            System.out.print(myBytes[i]+" ");
        }
```
- We can then just put this output through an ASCII decoder like https://www.dcode.fr/ascii-code.

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# vault-door-5

**Flag:** `picoCTF{c0nv3rt1ng_fr0m_ba5e_64_0b957c4f}`

## Approach

- We can see an `expected` string in the end of the `checkPassword()` function.
- The string is encoded using URL, then base64. So we can take the expected string and put it through a [base64 decoder](https://www.dcode.fr/base-64-encoding) and a [URL decoder](https://www.dcode.fr/url-decoder).

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Classic Crackme 0x100

**Flag:** `picoCTF{s0lv3_angry_symb0ls_4699696e}`

## Approach

- Disassemble the given program:
`dbg.main`
```
undefined8 dbg.main(void)
{
    uint32_t uVar1;
    int32_t iVar2;
    char input [51];
    char output [51];
    int64_t var_28h;
    int secret3;
    int secret2;
    int secret1;
    int len;
    int32_t iStack_10;
    int i;
    
    output = "qhcpgbpuwbaggepulhstxbwowawfgrkzjstccbnbshekpgllze"
    setvbuf(_stdout, 0, 2, 0);
    printf("Enter the secret password: ");
    __isoc99_scanf("%50s", input);
    i = 0;
    len = strlen(output);
    secret1 = 0x55;
    secret2 = 0x33;
    secret3 = 0xf;
    var_28h._7_1_ = 'a';
    for (; i < 3; i = i + 1) {
        for (iStack_10 = 0; iStack_10 < len; iStack_10 = iStack_10 + 1) {
            uVar1 = (iStack_10 % 0xff >> 1 & secret1) + (iStack_10 % 0xff & secret1);
            uVar1 = ((int32_t)uVar1 >> 2 & secret2) + (secret2 & uVar1);
            iVar2 = ((int32_t)uVar1 >> 4 & secret3) +
                    ((int32_t)input[iStack_10] - (int32_t)var_28h._7_1_) + (secret3 & uVar1);
            input[iStack_10] = var_28h._7_1_ + (char)iVar2 + (char)(iVar2 / 0x1a) * -0x1a;
        }
    }
    iVar2 = memcmp(input, output, SEXT48(len));
    if (iVar2 == 0) {
        printf("SUCCESS! Here is your flag: %s\n", "picoCTF{sample_flag}");
    } else {
        puts("FAILED!");
    }
    return 0;
}
```
- Convert the code to Python:
```
output = "qhcpgbpuwbaggepulhstxbwowawfgrkzjstccbnbshekpgllze"
inp = input("Enter the secret password: ")[:50] # input

length = len(output) # len
def crackme(inp: str):
    inp = list(inp)
    secret1 = 0x55
    secret2 = 0x33
    secret3 = 0xf
    a = ord('a')

    for i in range(0, 3):
        for j in range(0, length):
            # print("testing:", j, inp[j])
            uVar1 = (j % 0xff >> 1 & secret1) + (j & 0xff & secret1)
            uVar1 = (uVar1 >> 2 & secret2) + (secret2 & uVar1)
            iVar2 = (uVar1 >> 4 & secret3) + (ord(inp[j]) - a) + (secret3 & uVar1)
            inp[j] = chr(a + iVar2 + (iVar2 // 0x1a) * -0x1a)
    
    return "".join(inp)

print(crackme("aiousdbviubauwkeyfbuiashfduihusidrhgiuvsdlfiuhbsui"))
# Output: alravjhelahjaftqblhdojbtlmduqgexgxnpodeejuoudtnhar
```
- Simplify the code. It is a cyclic cipher.
```
inp = input("Enter the secret password: ")[:50] # input

def crackme(inp: str):
    def cyclic_cipher(offset: int, char: str):
        offset_map = [0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4, 1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5, 1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5, 2, 3]
        a = ord('a')
        term = offset_map[offset] + (ord(char) - a)
        return chr(a + term % 26)

    inp = list(inp)
    a = ord('a')

    for i in range(0, 3):
        for j in range(0, 50):
            # print((uVar1 >> 4 & secret3) + (ord(inp[j]) - a)) # To generate offset map
            inp[j] = crack(j, inp[j])
    return "".join(inp)

print(crackme("aiousdbviubauwkeyfbuiashfduihusidrhgiuvsdlfiuhbsui"))
# Output: alravjhelahjaftqblhdojbtlmduqgexgxnpodeejuoudtnhar
```
- Reverse the cyclic cipher
```
out = "qhcpgbpuwbaggepulhstxbwowawfgrkzjstccbnbshekpgllze"

def crack(enc: str):
    def cyclic_inverse(offset: int, char: str):
        offset_map = [0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4, 1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5, 1, 2, 2, 3, 2, 3, 3, 4, 2, 3, 3, 4, 3, 4, 4, 5, 2, 3]
        a = ord('a')
        term = (ord(char) - a) - offset_map[offset]
        return chr(a + term % 26)

    enc = list(enc)

    for i in range(0, 3):
        for j in range(0, 50):
            enc[j] = cyclic_inverse(j, enc[j])
    
    return "".join(enc)

print(crack(out))
# Output: qezjdvjltvuxavgiibmkrsncqrntxfykgmntwsepmyvyguzwtv
```
- Enter the output into the program to get the flag

## Incorrect methods tried

- Attempting to reverse the individual terms

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# unpackme.py

**Flag:** `picoCTF{175_chr157m45_cd82f94c}`

## Approach

- The payload contains the code that checks the password being input by the user. Printing it before its execution reveals the flag.

Input:
```
import base64
from cryptography.fernet import Fernet



payload = b'...'

key_str = 'correctstaplecorrectstaplecorrec'
key_base64 = base64.b64encode(key_str.encode())
f = Fernet(key_base64)
plain = f.decrypt(payload)
print(f"plain: {plain.decode()}")
# exec(plain.decode())
```
Output:
```
plain: 
pw = input('What\'s the password? ')

if pw == 'batteryhorse':
  print('picoCTF{175_chr157m45_cd82f94c}')
else:
  print('That password is incorrect.')

```

## New concepts

1. `cryptography.fernet.Fernet` encrypting

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Picker I

**Flag:** `picoCTF{4_d14m0nd_1n_7h3_r0ugh_ce4b5d5b}`

## Approach

- Enter `win` in the input of the program
- [Decode the flag](https://gchq.github.io/CyberChef/#recipe=Split('%200x','')From_Hex('0x')&input=MHg3MCAweDY5IDB4NjMgMHg2ZiAweDQzIDB4NTQgMHg0NiAweDdiIDB4MzQgMHg1ZiAweDY0IDB4MzEgMHgzNCAweDZkIDB4MzAgMHg2ZSAweDY0IDB4NWYgMHgzMSAweDZlIDB4NWYgMHgzNyAweDY4IDB4MzMgMHg1ZiAweDcyIDB4MzAgMHg3NSAweDY3IDB4NjggMHg1ZiAweDYzIDB4NjUgMHgzNCAweDYyIDB4MzUgMHg2NCAweDM1IDB4NjIgMHg3ZA) from hex


## New concepts

1. `exec()` and `eval()` functions in Python

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Picker II

**Flag:** `picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_0b5f1131}`

## Approach

- The program checks if the user input contains `win`, so get around it using multiple `b'...'.decode()` and `+` (string concatenation).
- Use `printf` in the terminal to enter the input:
```
$ python picker-II.py < <(printf "exec(b'\x77'.decode() + b'\x69\x6e'.decode() + '()')\n#")
TEST_FLAG
```
- [Decode the flag from hex](https://gchq.github.io/CyberChef/#recipe=Split('%200x','')From_Hex('0x')&input=MHg3MCAweDY5IDB4NjMgMHg2ZiAweDQzIDB4NTQgMHg0NiAweDdiIDB4NjYgMHgzMSAweDZjIDB4MzcgMHgzMyAweDcyIDB4MzUgMHg1ZiAweDY2IDB4MzQgMHgzMSAweDZjIDB4NWYgMHg2MyAweDMwIDB4NjQgMHgzMyAweDVmIDB4NzIgMHgzMyAweDY2IDB4MzQgMHg2MyAweDM3IDB4MzAgMHg3MiAweDVmIDB4NmQgMHgzMSAweDY3IDB4NjggMHgzNyAweDVmIDB4MzUgMHg3NSAweDYzIDB4NjMgMHgzMyAweDMzIDB4NjQgMHg1ZiAweDMwIDB4NjIgMHgzNSAweDY2IDB4MzEgMHgzMSAweDMzIDB4MzEgMHg3ZA)

## New concepts

1. `echo` vs `printf` in terminal

## Incorrect methods tried

- Using `echo` to input the string
- Trying to use multiple statements in `eval()`

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Picker III

**Flag:** `picoCTF{7h15_15_wh47_w3_g37_w17h_u53r5_1n_ch4rg3_c20f5222}`

## Approach

- The program checks user input for brackets but not for the `win` keyword
- Overwrite `getRandomNumber` with `win`:
```
$ nc saturn.picoctf.net 55249
==> 1
1: print_table
2: read_variable
3: write_variable
4: getRandomNumber
==> 3
Please enter variable name to write: getRandomNumber
Please enter new value of variable: win
==> 4
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x37 0x68 0x31 0x35 0x5f 0x31 0x35 0x5f 0x77 0x68 0x34 0x37 0x5f 0x77 0x33 0x5f 0x67 0x33 0x37 0x5f 0x77 0x31 0x37 0x68 0x5f 0x75 0x35 0x33 0x72 0x35 0x5f 0x31 0x6e 0x5f 0x63 0x68 0x34 0x72 0x67 0x33 0x5f 0x63 0x32 0x30 0x66 0x35 0x32 0x32 0x32 0x7d 
==>
```
- [Decode the flag from hex](https://gchq.github.io/CyberChef/#recipe=Split('%200x','')From_Hex('0x')&input=MHg3MCAweDY5IDB4NjMgMHg2ZiAweDQzIDB4NTQgMHg0NiAweDdiIDB4MzcgMHg2OCAweDMxIDB4MzUgMHg1ZiAweDMxIDB4MzUgMHg1ZiAweDc3IDB4NjggMHgzNCAweDM3IDB4NWYgMHg3NyAweDMzIDB4NWYgMHg2NyAweDMzIDB4MzcgMHg1ZiAweDc3IDB4MzEgMHgzNyAweDY4IDB4NWYgMHg3NSAweDM1IDB4MzMgMHg3MiAweDM1IDB4NWYgMHgzMSAweDZlIDB4NWYgMHg2MyAweDY4IDB4MzQgMHg3MiAweDY3IDB4MzMgMHg1ZiAweDYzIDB4MzIgMHgzMCAweDY2IDB4MzUgMHgzMiAweDMyIDB4MzIgMHg3ZA)

## New concepts

1. Overwriting Python objects' values

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Bit-O-Asm-1

**Flag:** `picoCTF{48}`

## Approach

- `mov    eax,0x30` means the `eax` register has the number 48 (30 in hexadecimal)

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Bit-O-Asm-2

**Flag:** `picoCTF{654874}`

## Approach

```
mov    DWORD PTR [rbp-0x4],0x9fe1a
mov    eax,DWORD PTR [rbp-0x4]
```
- The first line `0x9fe1a` (654874 in base 16) into `rbp-0x4`
- The second line stores that value in the `eax` register

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

# Bit-O-Asm-3

**Flag:** `picoCTF{2619997}`

## Approach

```
mov    DWORD PTR [rbp-0x4],0x9fe1a
mov    eax,DWORD PTR [rbp-0x4]
```
- `mov    eax,DWORD PTR [rbp-0xc]`: `eax` is now 654874
- `imul   eax,DWORD PTR [rbp-0x8]`: `eax` is now 654874 * 4 = 2619496
- `add    eax,0x1f5`: eax is now 2619496 + 501 = 2619997
- The next two lines store the value of eax in the stack and retrieve it, which does not change its value

&nbsp;

&nbsp;

<hr style="border:2px solid gray; background-color: gray">
&nbsp;

&nbsp;

