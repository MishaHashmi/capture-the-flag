# Rock Scissors Paper

Running the binary of this challenge simply presents us with a menu to play a game of rock scissors paper with the computer. Or rather a thousand games. It informs us that the key will simply be handed to us if we can win a thousand times straight.

![Inline](../images/rps_1.png?raw=true)

Observing the source file, we can see something strange. The **srand** function is being initialised with _2019_! 

![Inline](../images/rps_2.png?raw=true)

This is an unsafe practice. Seed of the random function determines the pattern of random numbers that will be generated. It is common to use date and time as the seed to ensure different numbers generated each time.

So now we have a vulnerability to exploit. We can generate the same numbers computer is using. It is easy to defeat someone in rock scissors paper if we already know their next move.



## PAYLOAD

Now all that’s left is automating the game through **pwn**. We send a winning letter each time and receive the key.

```python
from pwn import *

# Our target
target = './rock-scissor-paper'
e = context.binary = ELF(target,checksec=True)
context.log_level='debug'

io = process(target)

file = open("patt.txt", 'r')
pattern = file.read()
file.close()

for counter in range(1000):
    print("counter: ", counter)
    io.recv()
    io.send(pattern[counter])

io.recvuntil('-')
io.recv()
```
