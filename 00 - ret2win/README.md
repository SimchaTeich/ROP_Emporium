# ret2win

You can find this challange [here](https://ropemporium.com/challenge/ret2win.html).

## Black-Box Test
Let's run the program `ret2win32` and observe the output

```
./ret2win32
```
![](./0.png)

Let's input normal data, such as `ABCD`, and observe the program's behavior.

![](./1.png)

After some trial and error, I found the exact index where we can overwrite the return address.

```
perl -e 'print "A"x44 . "\xfe\xbe\xad\xde"' | ./ret2win32
```
![](./4.png)
```
sudo dmesg -k | tail -2
```
![](./5.png)





## Finding important details

If we examine the stack permissions, we'll find that NX protection is enabled on it, so we won't be able to execute code that is written directly on it.

```
rabin2 -I ret2win32
```
![](./2.png)

Therefore, we will look for functions that can help us.

```
rabin2 -qs ret2win32 | grep -ve 'imp' -e ' 0 ' -e '_'
```
![](./3.png)

The function `ret2win` looks very suspicious and matches the challenge name, so it's likely the function we're looking for. To be absolutely sure, let's take a closer look at the function itself.

