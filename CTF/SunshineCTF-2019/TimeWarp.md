## TimeWarp (Scripting, 50pts)

#### Challenge Description

Oh no! A t3mp0ral anoma1y has di5rup7ed the timeline! Y0u'll have to 4nswer the qu3stion5 *before* we ask them!

```
nc tw.sunshinectf.org 4101
```

Author: Mesaj2000

#### Overview

When we connect to the given hostname and port, we're greeted with the following message:

```I'm going to give you some numbers between 0 and 999.
Repeat them back to me in 30 seconds or less!
```

If we type any random number (let's say, `1`), we'll get a printout of the number that was the correct one before getting a message about our guess. 

```
1
39
0h de4r, not aga1n!
```

The message we get seems to be from a list of messages for both a correct and incorrect guess. The important thing is that if we connect again and enter `39`, we get it correct.

```
39
39
G3tting cl0ser!
1
61
G3tting c0lder!
```

This means the numbers we have to guess are consistent every time we connect, so we need to build a list of correct numbers to send to get the flag.

A good way to go about doing this is to write a script that will try a number, and if it fails, store the printout of the correct answer, then reconnect and use that number. We'll then do this recursively until we get the flag. In a way, you could consider this a script that "auto learns" from it's previous failures.

You could use one of two ways to determine if your guess was correct or not. The first is to build a table of the responses for correct and incorrect answers and match the second message against that. The second is to compare your guess to the number it prints out after you send it. This is the way I chose as it was easier.

#### Flag Script

I used pwntools for handling the networking side of things since I already had it setup for pwn challenges.

```python
from pwn import *
context(arch='i386', os='linux')

# Setup a list to hold correct answers
resetConnection = True
correctAnswers  = []
i = 0
r = None

while i < 300:
	if resetConnection:
		r = remote('tw.sunshinectf.org', 4101)

		# Receive trash
		r.recvline()
		r.recvline()

		resetConnection = False

	guess = 1

	if len(correctAnswers) > i:
		guess = correctAnswers[i]

	r.send(str(guess) + "\x0A")

	resp = r.recvline()
		
	# If our guess was incorrect, store it in the correctAnswers list and start over
	if int(resp) != guess:
		correctAnswers.append(int(resp))
		i = 0

		# Need to reset the connection
		resetConnection = True
	else:
		print("[+] Question #" + str(i) + " answer: " + str(guess))

		# If our guess was correct, receive the next line and continue
		potentialFlag = r.recvline()
		i += 1

		print("[+] Response: " + potentialFlag)

print(r.recvline())
print(r.recvline())
```

As it turns out, you need to do this 300 times before you'll get the flag. It can take a little while and there may be a more elegant solution, but this is the solution I came up with. After running it for long enough, we'll get the following output:

```
[+] Question #299 answer: 370
[+] Response: Wow! You did it!

As a reward for fixing the timestream, here's the flag:

sun{derotser_enilemit_1001130519}
```

The flag:

```
sun{derotser_enilemit_1001130519}
```

I had quite a bit of fun with this challenge, though the script took a little bit to run due to how many rounds there were.