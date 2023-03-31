---
title: Flippin bank crypto challenge Writesup - (Padding Oracle Attack) -
date: "2023-03-30T12:00:00.000Z"
template: "post"
draft: false
slug: "flippin-bank-writes-up"
category: "Security"
tags:
  - "Security"
  - "HackTheBox"
  - "Cryptography"
description: "Writeups of Flippin bank crypto challenge in HackTheBox. This challenge requires a padding oracle attack."
socialImage: "/media/flippin-bank/encryption.jpg"
---

![encryption](/media/flippin-bank/encryption.jpg)

This is a writes up of [Flippin Bank](https://app.hackthebox.com/challenges/Flippin-Bank) crypo challenge in HackTheBox.

1. [Foothold](#foothold)
2. [Padding oracle attack](#padding-oracle-attack)
3. [Exploit (Encryption Attack)](#exploit-encryption-attack)
4. [Extra (Decryption attack)](#extra-decryption-attack)

## Foothold

When I connect to the instance using the `nc` command, I am prompted to enter my username and password. After entering them, a leaked ciphertext is displayed, and I am asked to input another ciphertext. I copy the leaked ciphertext and paste it there, but I receive an error message saying `Please try again`. It appears that the ciphertext I entered was incorrect.

![first_request](/media/flippin-bank/first_request.png)

Let's take a look at `app.py` file that I downloaded from the challenge page.
Within the file, it decrypts the cipher text and checks if the plain text includes `admin&password=g0ld3n_b0y`.
If this string is included, the app displays the flag.

```python
try:
	check = decrypt_data(enc_msg)
except Exception as e:
	send_msg(s, str(e) + '\n')
	s.close()

if check:
	send_msg(s, 'Logged in successfully!\nYour flag is: '+ FLAG)
	s.close()
else:
	send_msg(s, 'Please try again.')
	s.close()

def decrypt_data(encryptedParams):
	cipher = AES.new(key, AES.MODE_CBC,iv)
	paddedParams = cipher.decrypt( unhexlify(encryptedParams))
	if b'admin&password=g0ld3n_b0y' in unpad(paddedParams,16,style='pkcs7'):
		return 1
	else:
		return 0
```

The leaked cipher text is a encrypted text of `msg` string where input username and password are combined like this below.

```python
msg = 'logged_username=' + user +'&password=' + passwd
```

That means, if I leak a cipher text including `admin&password=g0ld3n_b0y`, I can input it and gets the flag.

Let's input `admin` as the username and `g0ld3n_b0y` as the password.
However, The app exits before it leak the cipher text.

![admin](/media/flippin-bank/admin.png)

The reason is below. Before the app encrypts `msg` string and leaks it, the app checks if `msg` includes `admin&password=g0ld3n_b0y`.
If this string is included, the app exits.

Now I understand how the app works. The challenge is that I have to somehow create a cipher text including `admin&password=g0ld3n_b0y`.

```python
try:
	assert('admin&password=g0ld3n_b0y' not in msg)
except AssertionError:
	send_msg(s, 'You cannot login as an admin from an external IP.\nYour activity has been logged. Goodbye!\n')
	raise
```

Let's look at how the app encrypts and decrypts a plain text. The app uses `AES-CBC` with `PKCS7` padding. The initial vector and secret key are initialized when the app starts running.


## Padding oracle attack

After I google a bit, I found one famous attack against `AES-CBC` with `PKCS7` padding encryption, which is called [Padding oracle attack](https://en.wikipedia.org/wiki/Padding_oracle_attack).
> In cryptography, a padding oracle attack is an attack which uses the padding validation of a cryptographic message to decrypt the ciphertext. 

Padding oracle attacks can be used when the following conditions are met:

1. __Symmetric encryption with a block cipher__: The target system must be using a symmetric encryption algorithm in block cipher mode, such as AES-CBC or DES-CBC. 
=> the app uses `AES-CBC` encryption.

2. __Vulnerable padding scheme__: The target system must be using a padding scheme, such as PKCS7, that is susceptible to padding oracle attacks. 
=> the app uses `PKCS7` padding.

3. __Leaky padding validation__: The target system must provide an "oracle" that leaks information about whether the padding of a manipulated ciphertext is valid or not.

4. __Ability to send ciphertext multiple times__: The attacker must be able to send ciphertext multiple times to the target system.
=> I can send cipher text multiple times.

So, I don't know yet if the app satisfies the condition 3.
In order to check it, I replace the last byte of the leaked cipher with `0x00` and send it.
The app says `Padding is incorrect`, so the condition 3 is also satisfied, which means I can use padding oracle attack.

![padding oracle](/media/flippin-bank/padding_oracle.png)

I will not explain how padding oracle attack works in this article because there are many useful articles in Internet.

I mainly read these articles below.
1. [Dotnet’s default AES mode is vulnerable to padding oracle attacks](https://pulsesecurity.co.nz/articles/dotnet-padding-oracles)
2. [Padding Oracle AttackによるCBC modeの暗号文解読と改ざん (Japanese)](https://rintaro.hateblo.jp/entry/2017/12/31/174327)
3. [Padding Oracle Attack 分かりやすく解説したい (Japanese)](https://partender810.hatenablog.com/entry/2021/06/08/225105)

There are 2 type of attacks in padding oracle attack.
1. Decryption attack: An attacker exploits the padding oracle to decrypt a given ciphertext without knowing the encryption key by iteratively modifying and sending crafted ciphertexts to the oracle and observing its responses to reveal the original plaintext.
2. Encryption attack: An attacker exploits the padding oracle to create a new ciphertext for a chosen plaintext without knowing the encryption key by leveraging their ability to decrypt any ciphertext and adjusting the plaintext accordingly, effectively encrypting a new message.

I use the encryption attack for this challenge.

## Exploit (Encryption Attack)
It's time to exploit. The below python code is the exploit code.
I refered to the implementation of [the article](https://rintaro.hateblo.jp/entry/2017/12/31/174327).

```python
from pwn import *
from tqdm import tqdm
from Crypto.Util.Padding import pad
from typing import List

context.log_level = 'error'

HOST = "46.101.81.60"
PORT = 31775
BLOCK_SIZE = 16

def is_valid_padding(c_target: str, dec_ci: bytearray, m_prime: int, c_prev_prime: int) -> bool:
    """
    Check if the padding is valid through the app by chainging the ciphertext

    :param c_target: The target ciphertext block as a hex string
    :param dec_ci: Decrypted block (intermediate state) as a bytearray
    :param m_prime: The byte position (1-indexed) being attacked in the current block
    :param c_prev_prime: The modified byte value for the previous block's corresponding byte position
    :return: boolean if the padding is valid
    """
    r = remote(HOST, PORT)
    # here is not important because the leaked cipher text is not our concerns.
    r.sendafter("username: ", "foo")
    r.sendafter("password: ", "bar")

    # ex. 00000000000000000046
    attempt_byte = b"\x00" * (BLOCK_SIZE-m_prime) + p8(c_prev_prime)
    adjusted_bytes = b""
    for c in dec_ci:
        adjusted_bytes += p8(c ^ m_prime)
    
    r.sendafter("enter ciphertext: ", attempt_byte.hex() + adjusted_bytes.hex() + c_target)
    res = r.recvall()
    return "incorrect" not in res.decode()

def send_cipher(cipher_text: str):
    r = remote(HOST, PORT)
    r.sendafter("username: ", "admin")
    r.sendafter("password: ", "g0ld3n_b0x")
    r.sendafter("enter ciphertext: ", cipher_text)
    res = r.recvall()
    print(res.decode())

def create_cipher(plain_block: List[str], dec_block: List[str]) -> str:
    """
    Creates a ciphertext by XORing corresponding plain and decrypted blocks, and appends a block of zeros.

    :param plain_block: A list of plaintext block hex strings
    :param dec_block: A list of decrypted block hex strings
    :return: The resulting ciphertext as a hex string
    """
    c0 = format(int(plain_block[0],16) ^ int(dec_block[2],16),'x').zfill(BLOCK_SIZE*2)
    c1 = format(int(plain_block[1],16) ^ int(dec_block[1],16),'x').zfill(BLOCK_SIZE*2)
    c2 = format(int(plain_block[2],16) ^ int(dec_block[0],16),'x').zfill(BLOCK_SIZE*2)
    c3 = "00" * BLOCK_SIZE
    return c0 + c1 + c2 + c3

def encryption_attack() -> str:
    # initial Dec(ci) value. This value will be updated.
    initial = "00" * BLOCK_SIZE
    target_plain_text = "logged_username=admin&password=g0ld3n_b0y"
    plain = pad(target_plain_text.encode(),16,style='pkcs7').hex()

    plain_block = [plain[i: i+BLOCK_SIZE*2] for i in range(0, len(plain), BLOCK_SIZE*2)]
    dec_block = [initial] * len(plain_block)

    cipher_text = create_cipher(plain_block, dec_block)
    cipher_text = cipher_text.zfill(len(cipher_text) + len(cipher_text) % BLOCK_SIZE*2)

    # split cipher_text into ciphe_block by BLOCK_SIZE * 2
    cipher_block = [cipher_text[i: i+BLOCK_SIZE*2] for i in range(0, len(cipher_text), BLOCK_SIZE*2)]
    cipher_block.reverse()

    for i in tqdm(range(len(cipher_block)-1)):
        c_target = cipher_block[0]
        c_prev = cipher_block[1]
        
        print("c_prev: {}".format(c_prev))
        print("c_target: {}".format(c_target))
        cipher_block.pop(0)

        m_prime = 1
        c_prev_prime = 0
        dec_ci = b""  # Dec(ci)

        while True:
            if is_valid_padding(c_target, dec_ci, m_prime, c_prev_prime):
                print("0x{:02x}: ".format(c_prev_prime) + "{:02x}".format(m_prime) * m_prime)
                dec_ci = p8(c_prev_prime ^ m_prime) + dec_ci
                m_prime += 1
                c_prev_prime = 0
                if m_prime <= BLOCK_SIZE:
                    continue
                break
            c_prev_prime += 1
            if c_prev_prime > 0xff:
                print("Not Found")
                break
        dec_block[i] = dec_ci.hex().zfill(BLOCK_SIZE*2)
        cipher_text = create_cipher(plain_block, dec_block)
        cipher_block = [cipher_text[j: j+BLOCK_SIZE*2] for j in range(0, len(cipher_text), BLOCK_SIZE*2)]
        cipher_block.reverse()
        for _ in range(i+1):
            cipher_block.pop(0)
    tempered_cipher = create_cipher(plain_block, dec_block)
    print("[+] tempered cipher text:", tempered_cipher)

    return tempered_cipher

if __name__ == "__main__":
    cipher_text = encryption_attack()
    send_cipher(cipher_text)
```

![flippin-bank](/media/flippin-bank/flippin-bank.png)

## Extra(Decryption attack)

Decryption attack can also work for this app.
I took some time to figure out how to decrypt the entire text because this app doesn't include the initial vector in the cipher text.

In the end, I noticed the initial vector can be calculated with the first decrypted block(dec_ci(1)).
The idea is we run the decryption attack with a null IV, then calculate XOR with the first decrypted block and the first block of original plain text. Since the app always uses the same initial vector, we can decrypt the entire text by doing the same attack again with the calculated initial vector.

```python
def get_initial_cipher(username: str, password: str) -> str:
    r = remote(HOST, PORT)
    r.sendafter("username: ", username)
    r.sendafter("password: ", password)
    r.recvuntil("Leaked ciphertext:")
    cipher = r.recvline().decode().strip().replace("\n", "")
    r.sendafter("enter ciphertext: ", "random")
    r.recvall()
    return cipher

def decryption_attack(iv: bytearray = b"\x00" * BLOCK_SIZE) -> str:
    """
    execute a decryption attack to get the initial vector

    :iv: initial vector. the default is null vector
    :return: initial vector
    """
    # plain text of the first cipher block
    # it will be used to calculate the initial vector later
    initial_block_plain = "logged_username="
    
    cipher_text = get_initial_cipher("admin", "password")
    cipher_text = iv.hex() + cipher_text.zfill(len(cipher_text) + len(cipher_text) % BLOCK_SIZE*2)

    # split cipher_text into ciphe_block by BLOCK_SIZE * 2
    cipher_block = [cipher_text[i: i+BLOCK_SIZE*2] for i in range(0, len(cipher_text), BLOCK_SIZE*2)]
    # decrypt cipher_text from the last block
    cipher_block.reverse()

    plain_text = ""
    block_length = len(cipher_block)
    for i in tqdm(range(block_length-1)):
        c_target = cipher_block[0]
        c_prev = cipher_block[1]
        
        print("c_prev: {}".format(c_prev))
        print("c_target: {}".format(c_target))
        cipher_block.pop(0)

        m_prime = 1
        c_prev_prime = 0
        m = ""
        dec_ci = b""

        while True:
            if is_valid_padding(c_target, dec_ci, m_prime, c_prev_prime):
                print("0x{:02x}: ".format(c_prev_prime) + "{:02x}".format(m_prime) * m_prime)
                # m = c' ^ m' ^ c
                m += chr(c_prev_prime ^ m_prime ^ ord(bytes.fromhex(c_prev).decode('latin-1')[::-1][m_prime-1]))
                dec_ci = p8(c_prev_prime ^ m_prime) + dec_ci
                m_prime += 1
                c_prev_prime = 0
                if m_prime <= BLOCK_SIZE:
                    continue
                break
            c_prev_prime += 1
            if c_prev_prime > 0xff:
                print("Not Found")
                break
        dec_ci = dec_ci.hex().zfill(BLOCK_SIZE*2)

        # calculate initial vector from the Dec(1)
        if i == block_length - 2:
            iv = xor(bytes.fromhex(dec_ci), initial_block_plain.encode())

        print("[+] Dec({}): {}".format(len(cipher_block), dec_ci))
        print("[+] m{}: {}".format(len(cipher_block), repr(m[::-1])))
        plain_text = m[::-1] + plain_text
    print("plain text: {}".format(plain_text))
    print("initial vector: {}".format(iv.hex()))

    return iv

if __name__ == "__main__":
    iv = decryption_attack()
    decryption_attack(iv)
```

As you can see the picture below, After the first attack, the first block of the decrypted text is broken, but the initial vector is found.

![iv_found](/media/flippin-bank/iv_found.png)

After the second attack with the initial vector, the text is properly decrypted.

![plain_found](/media/flippin-bank/plain_found.png)