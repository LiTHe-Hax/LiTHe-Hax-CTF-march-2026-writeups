The one time code is generated with python's random module, which is not cryptographically secure.

The random module uses the "Mersenne Twister", which doesn't create truly random numbers.

By collecting 624 32-bit samples (the one time code happens to be random 32-bit numbers),
one can recreate the current state of the Mersenne Twister and predict the next numbers.

Now you can send a one time code to the admin account and predict what code was sent.

There are recources online which help crack the the module, in my solution script I used [this one](https://github.com/kmyk/mersenne-twister-predictor)

Solution script:
```
import string
import time

from mt19937predictor import MT19937Predictor
from pwn import *


HOST = "127.0.0.1"
PORT = 80
HAS_SSL = False
r = remote(HOST, PORT, ssl=HAS_SSL)


def create_user():
    r.recvuntil(b":")
    r.sendline(b"2")
    r.recvuntil(b":")
    r.sendline()
    r.recvuntil(b":")
    r.sendline()


def get_sample():
    r.recvuntil(b":")
    r.sendline(b"1")
    r.recvuntil(b":")

    r.sendline()
    r.recvuntil(b":")

    r.sendline(b"?")
    r.recvuntil(b":")

    r.sendline(b"1")
    r.recvuntil(b":")
    
    r.sendline(b"q")
    r.recvuntil(b":")

    r.sendline()
    r.recvuntil(b":")

    r.sendline()
    r.recvuntil(b":")
    
    r.sendline(b"1")
    r.recvuntil(b":")
    text = r.recvuntil(b":").decode().splitlines()[1].strip()
    otc = int(text)

    r.sendline(b"q")
    return otc


def stringify(number):
    as_str = str(number)
    return "0" * (10 - len(as_str)) + as_str


def login_admin(otc):
    r.recvuntil(b":")
    r.sendline(b"1")
    r.recvuntil(b":")
    
    r.sendline(b"admin")
    r.recvuntil(b":")

    r.sendline(b"?")
    r.recvuntil(b":")

    r.sendline(b"1")
    
    r.sendline(otc.encode())
    return r.recvall()


if __name__ == "__main__":
    predictor = MT19937Predictor()
    create_user()
    for i in range(624):
        sample = get_sample()
        print(f"{str(i+1)+":":<5}", sample)
        predictor.setrandbits(sample, 32)
    admin_otc = stringify(predictor.getrandbits(32))
    flag_str = login_admin(admin_otc)
    flag = flag_str.decode().splitlines()[-1]
    print()
    print("flag:")
    print(flag)

```
