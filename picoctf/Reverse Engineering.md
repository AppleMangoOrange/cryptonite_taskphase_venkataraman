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

