# callme
The challenge is available [here](https://ropemporium.com/challenge/callme.html).

## Black-Box Test
Let's start as usual with an initial check of the program's behavior.

```
./callme32
```
![](./0.png)

In this challenge as well, the return address is located at the same index.

```
perl -e 'print "A"x44 . "\xef\xbe\xad\xde"' | ./callme32
```
![](./1.png)
```
sudo dmesg -k | tail -2
```
![](./2.png)

Before we start exploring in more depth, let's check what the maximum input size we can enter is.

```
ltrace ./callme32
```
![](./3.png)

By entering simple input (in this case, `ABCD`) while running the program with `ltrace`, we discovered that the maximum input size we can enter is `512` bytes.

## In-depth research
