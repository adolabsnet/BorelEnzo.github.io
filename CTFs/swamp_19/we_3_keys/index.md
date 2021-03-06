# We Three Keys - Crypto

### [~$ cd ..](../)

>In this modern world of cyber, keys are the new kings of the world. In fact, we will let you use the 3 best keys that I could find, make sure to see what they offer to you!
>
>nc chal1.swampctf.com 1441
>
>-= Created by noopnoop =-

This challenge was easy, but quite fun! We were given the [code](serv.py) of the service running on port 1441:

```python
#!/usr/bin/env python2
from Crypto.Cipher import AES
from keys import key1, key2, key3

def pkcs7_pad(msg):
    val = 16 - (len(msg) % 16)
    if val == 0:
        val = 16
    pad_data = msg + (chr(val) * val)
    return pad_data

def encrypt_message(key, IV):
    print("What message woud you like to encrypt (in hex)?")
    ptxt = raw_input("<= ")
    ptxt = pkcs7_pad(ptxt.decode('hex'))
    cipher = AES.new(key, AES.MODE_CBC, IV)
    ctxt = cipher.encrypt(ptxt)
    print ctxt.encode('hex')

def decrypt_message(key, IV):
    print("What message would you like to decrypt (in hex)?")
    ctxt = raw_input("<= ")
    ctxt = ctxt.decode('hex')
    if (len(ctxt) % 16) != 0:
        print "What a fake message, I see through your lies"
        return
    cipher = AES.new(key, AES.MODE_CBC, IV)
    ptxt = cipher.decrypt(ctxt)
    print ptxt.encode('hex')

def new_key():
    print("Which key would you like to use now? All 3 are great")
    key_opt = str(raw_input("<= "))
    if key_opt == "1":
        key = key1
    elif key_opt == "2":
        key = key2
    elif key_opt == "3":
        key = key3
    else:
        print("Still no, pick a real key plz")
        exit()
    return key

def main():
    print("Hello! We present you with the future kings, we three keys!")
    print("Pick your key, and pick wisely!")
    key_opt = str(raw_input("<= "))
    if key_opt == "1":
        key = key1
    elif key_opt == "2":
        key = key2
    elif key_opt == "3":
        key = key3
    else:
        print("Come on, I said we have 3!")
        exit()
    while True:
        print("1) Encrypt a message")
        print("2) Decrypt a message")
        print("3) Choose a new key")
        print("4) Exit")
        choice = str(raw_input("<= "))
        if choice == "1":
            encrypt_message(key, key)
        elif choice == "2":
            decrypt_message(key, key)
        elif choice == "3":
            key = new_key()
        else:
            exit()


if __name__=='__main__':
    main()
```

The major flaw here is the fact the pair key/IV is reused, and furthermore, they are the same. It means that if we can leak the IV, we have the key, and therefore we can decrypt or encrypt whatever we want. Since there is no real flag in this script, we guessed that we had to use the concatenation of the 3 keys to get the flag.

As a reminder, CBC mode works as follows (wikipedia):

![CBC enc](https://upload.wikimedia.org/wikipedia/commons/8/80/CBC_encryption.svg)
![CBC dec](https://upload.wikimedia.org/wikipedia/commons/2/2a/CBC_decryption.svg)

The idea is then to provide a plaintext full of null bytes (padded with a block full of 0x10, but we don't care), and therefore we can obtain (padding is ignored): `enc(IV ^ 0x00000000000000000000000000000000) = enc(IV) = enc(KEY)`

Then, we can ask for the decryption of `0x00000000000000000000000000000000 || enc(KEY) || padding` (`||` being the concatenation). Indeed, the second block, once decrypted will be XOR'ed with null-bytes, meaning that we have the key as plain text in the second block:

>plaintext=   
>dec(0x00000000000000000000000000000000)^IV +  
>dec(enc(KEY))^0x00000000000000000000000000000000 +  
>dec(padding^enc(KEY))

Step by step procedure for the first key:

* Encrypt a full-of-null-byte-block
```
Hello! We present you with the future kings, we three keys!
Pick your key, and pick wisely!
<= 1
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 1
What message woud you like to encrypt (in hex)?
<= 00000000000000000000000000000000
c921fca7080c90790bb85def50eaf466fda2febcef1d1a4cd6552dea3a65617e
```
* The encrypted key is the first block: `c921fca7080c90790bb85def50eaf466`. We then ask for the decryption of `16 * 0x00 || enc(KEY) || padding`
```
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 2
What message would you like to decrypt (in hex)?
<= 00000000000000000000000000000000c921fca7080c90790bb85def50eaf46610101010101010101010101010101010
9c53fc4f03020389898c77d7cdaa0f74666c61677b7730775f776834745f6c349ebc490a69a7b455acecd4cae69b4c36
```
* 2nd block is the decrypted IV: `666c61677b7730775f776834745f6c34`

We can use the same procedure for the second and third key:

```
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 3
Which key would you like to use now? All 3 are great
<= 2
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 1
What message woud you like to encrypt (in hex)?
<= 00000000000000000000000000000000
a38dc860ac85e5e6f107cdaca1877674e2ed66cd60ecce94e6765efdee280f30
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 2
What message would you like to decrypt (in hex)?
<= 00000000000000000000000000000000a38dc860ac85e5e6f107cdaca187767410101010101010101010101010101010
bf164d661c22f304765e22c443bf0fb57a795f6b33797a5f6d7563685f7733344a4b2726883dd3f52b3900d2773a2449
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 3
Which key would you like to use now? All 3 are great
<= 3
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 1
What message woud you like to encrypt (in hex)?
<= 00000000000000000000000000000000
7d5c6981c493ff37c629ca89f3bd9229b327b56f69fa0dd4d5880389129426a9
1) Encrypt a message
2) Decrypt a message
3) Choose a new key
4) Exit
<= 2
What message would you like to decrypt (in hex)?
<= 000000000000000000000000000000007d5c6981c493ff37c629ca89f3bd922910101010101010101010101010101010
dbccb2cd811df9b97b9e34789c010e056b5f6372797074305f6634316c73217d427e7d9e1eb49c1e155ac42213f74403
```

By concatenating the 3 keys, we obtained: `666c61677b7730775f776834745f6c347a795f6b33797a5f6d7563685f7733346b5f6372797074305f6634316c73217d`, and once hex-decoded: **flag{w0w_wh4t_l4zy_k3yz_much_w34k_crypt0_f41ls!}**

EOF
