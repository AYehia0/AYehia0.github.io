---
title: "Getting into hacking [Hacking Challenges]: Can you hack it?"
date: 2023-12-01T21:36:06+02:00
lastmod: 2023-12-01T21:36:06+02:00
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true

authors: []
description: "Attempting to solve CanYouHackIt Challenge."

tags: []
categories: [hacking, linux]
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"
---
This is my attempt to solve the "Can you hack it?" Challenge. I will share the solutions and how to achieve thme, without spoiling the secrets.
<!--more-->

## Can you hack it?
What da heck is `Can you hack it?` challenge?. `Can you hack it?` is a challenge hosted by `AIS - ainfosec` to give participants a chance to join the team. Score a total of 700 points to unlock the ability to submit your score. Your score submission and email will be sent directly to AIS and someone will be in touch.

I like challenges, that's why I am trying to give it a shot, while I might not solve them all, but I will be learning alot by time I give up.


{{< admonition warning "Before getting started!!!" >}}
Before reading this blog, first give it a try, and come for solutions if you get stuck.
{{< /admonition >}}

## How to play?
Navigate to [the challenge](https://hack.ainfosec.com/) and hover over a challenge tile to flip it. Once a challenge is solved the points will appear in the top right. You may need to refresh the page to update the points depending on how you solved the challenge.

## Categories
The challenge has many categories with different difficulties based on the score number (the bigger, the harder):-
- Client-side Protections
- Networking
- Crypto
- Steganography
- Exploitation
- Reverse Engineering
- Input Validation
- Programming

###  Client-side Protections
#### Disabled 
That's really easy and stright-forward challenge. button is disabled on the `HTML`, so just edit that html tag.

#### Button Clicker 
That's really interesting one, first I tried to click the button using `js`:
```js
for (let i = 0; i < 999999999; i++) {
    document.querySelector("#id_of_button").click()
}
```
and yeah, as expected my browser crashed lol. After some time playing with the console (which sometimes can contain the answer!), I found the source code for `ButtonClicker`.

{{< image src="button_clicker.png" caption="Button Clicker Source Code" >}}

setting `ButtonClicker_num_clicks` with the desired number solves this one.

#### Weird Input
This one is funny, coz there is a js function which changes the input to 'a's. Just as the previous one, make this function do nothing and you will be able to solve it.

{{< image src="weird_input.png" caption="Weird Input, aaaaaaaaa!" >}}

#### Paid Content 
This challenge taught me a lot, coz I was searching in the wrong direction. I first checked the source code as usual (and it didn't look that great).
{{< image src="paid_content_attempt1.png" caption="Wtf is that?" >}}

then I inspected the network and the response I found looked like this:
```json
{
    "error": "You're not a paid user.",
    "hc_challenge": {
        "oid": "08be3a909cb31a4f2e80",
        "meta": {
            "points": 50,
            "challenge_id": "paid_content",
            "name": "Paid Content",
            "description": "Pay for things you want! (2.0!!!)",
            "prompt": "You must be a paid user to proceed.",
            "category": {
                "value": "client_side_protections",
                "name": "Client-side Protections"
            },
            "console_message": "[Paid Content]: This challenge got a minor update for the 2023 holidays. It's version 2.0. It's (2 * challenge)!",
            "mobile_friendly": false,
            "encoded": true,
            "error_msg": "You're not a paid user.",
            "version": "2.0",
            "inputs": [],
            "retired": false,
            "bin_hashes": null
        },
        "solved": false,
        "js_file": "...",
        "js_function": "PaidContent",
        "num_solves": 1945,
        "paid": false, <--------- Can you spot this ?
        "challenge": 1964
    }
}
```
can you see it ?, if I could just set this to true before making the request, that would be it.
{{< image src="paid_content_ans2.png" caption="Yes, you can edit the JS code!" >}}

###  Networking
#### HTTP Basic 
That's another easy and stright-forward challenge. Download the `http-auth.cap` and open it with wireshark (not that doo doo doo), and filter all `POST` (`http.request.method == "POST"`) requests (coz you know post requests usually contains data to be sent!).

{{< image src="basic_http.png" caption="Wireshark doo doo doo, sorry :D" >}}

#### WPA2 Deauth 
Hacking networks!, the good old days. In this challenge I used `aircrack-ng` coz why not?
- Extract info from `.cap`
```bash
aircrack-ng ./de-auth.cap  
```
Cracking the password, and here I wasted lot of time trying to find where I put my wordlists lol (for future me, the path is : `/usr/share/hack/`).

{{< image src="wpa2_deauth.png" caption="I love when I find the needle in the hay stack!" >}}

###  Crypto
#### Skip Cipher 
This challenge is easy as it clearly says the cipher, so with quick search you will find many online tools to decode, I used [this](https://www.dcode.fr/skip-cipher).

#### Encoded 
That's a kinda interesting one, it says `Encoding is not cryptography! (2.0!!! Now with even more layers!)`. At the first glance I thought about that mores code challenge on `hackthebox`, and it was close (many layers).
```
504b0304140000000800308281576ed34a1e5c0000005c00000008000000666c61672e74787405c1c10e83200c00d0cf5a3c96c6a106b354d4aab7ed02acb8193501ffdef7286eff57ff546c979ec9653609a0c2194a8019c0812629062c8d7475b48dcd72ea4b7f31ad9c99d2d8caded23bf80faebe31e121e187be562a1c93bb01504b01021403140000000800308281576ed34a1e5c0000005c000000080000000000000000000000800100000000666c61672e747874504b0506000000000100010036000000820000000000
```
This looks like a `zip` format, so let's extract the data inside:
```python
st = "..."
binary_data = bytes.fromhex(st)
with open("output.zip", "wb") as f:
    f.write(binary_data)
```
After extracting the data it show `flag.txt` which contains `base64` string, trying to decode it gives some trash string (but it's not!):
```
BZh91AY&SY��@�@�`�0�i fuxg+ͅW85-F)6cedYhE64SmfCk=knU+>.p!c
```
as you can see `bz`, another compression. Let's extract again (this time using bash):
```bash
echo "$base64_encoded_string" | base64 --decode | bzcat > output.txt
```
output.txt contains binary (great!):
```
110110 110110 110110 1100011 110110 110001 110110 110111 110111 1100010 110110 110111 110111 110101 110111 110010 110110 110110 110110 1100011 110110 110101 110010 1100001 110110 111000 110110 110101 110011 110001 110110 1100011 110011 110000 110010 1100110 110111 110111 110011 110000 110111 110010 110110 1100011 110110 110100 110101 1100110 110110 1100101 110111 110101 110110 1100011 110110 1100011 110101 1100110 110100 110000 110110 110110 110110 1100010 110011 1100100 110111 110011 110111 110101 110110 110100 110110 1100110 110111 1100100
```
Let's convert it to ASCII:
```
666c61677b677572666c652a6865316c302f7730726c645f6e756c6c5f40666b3d7375646f7d
```
Now we have HEX lol (it's convert it to ASCII again):
```bash
echo "$hex_string" | xxd -r -p
```
Finally we get the flag: `flag{gurfle*he1l0/***********}`

#### ENIGMA 
This is really badass challenge!, i wasted too much time in it and I could solve but there are what you should do to be able to solve it properly:
- Inspecting the console, gives us some hints (that's where you should always start from)

{{< image src="enigma_hints.png" caption="Hints but at what cost!" >}}

Let's analysis those hints:
1. `QK JO LU XG DV` looks like plugboard wiring map.
2. `UKW B` is indeed the reflector being used.
3. `3 of 5 Rotors` it's not clear what does that mean but I guess it's the number of used rotors.
4. `First 5 digits of Pi` that's another ambiguous hint, but it could be the (ring settings/Position setting) as `"3 14 15"`
5. `Metasploit Acquired by Rapid7` hmm the date?!, Metasploit acquired by Rapid7 on `20 Oct, 2009` so (ring settings/Position setting) could be (`"20 10 09"`, `"10 20 09"`). there is another possiblity but I just ignored it.

{{< image src="enigma_online.png" caption="How to solve online ?" >}}

There are many possiblities and permutations to try, so it's better to write a code for that
```python
from itertools import permutations
from string import ascii_uppercase

def enigma_decrypt(encrypted_message, rotor_order, reflector_type, plugboard_settings, initial_positions):
    rotor_wirings = {
        'I': 'EKMFLGDQVZNTOWYHXUSPAIBRCJ',
        'II': 'AJDKSIRUXBLHWTMCQGZNPYFVOE',
        'III': 'BDFHJLCPRTXVZNYEIWGAKMUSQO',
        'IV': 'ESOVPZJAYQUIRHXLNFTGKDCMWB',
        'V': 'VZBRGITYUPSDNHLXAWMJQOFECK'
    }

    reflector_wirings = {
        'UKW-B': 'YRUHQSLDPXNGOKMIEBFZCWVJAT'
    }

    plugboard = {char: char for char in ascii_uppercase}
    plugboard.update(plugboard_settings)

    rotor_positions = {rotor: initial_positions.split()[i] for i, rotor in enumerate(rotor_order.split())}

    decrypted_message = ''
    for char in encrypted_message:
        if char in plugboard:
            char = plugboard[char]

        for rotor, position in rotor_positions.items():
            char = rotor_wirings[rotor][(ord(char) - ord('A') + ord(position) - ord('A')) % 26]

        char = reflector_wirings[reflector_type][ord(char) - ord('A')]

        for rotor, position in reversed(rotor_positions.items()):
            char = chr((ord(char) - ord(position) + ord('A')) % 26 + ord('A'))

        if char in plugboard:
            char = plugboard[char]

        decrypted_message += char

        for rotor in rotor_positions:
            rotor_positions[rotor] = chr((ord(rotor_positions[rotor]) - ord('A') + 1) % 26 + ord('A'))

    return decrypted_message

def get_rotor_permutations():
    """Return all the possible permutations"""
    rotor_labels = ['I', 'II', 'III', 'IV', 'V']
    num_rotors = 3

    rotor_combinations = permutations(rotor_labels, num_rotors)

    return list(rotor_combinations)

encoded_message = "RSHDQ VKAXO LONTP SXKHY DGOWH BKUBK MAAGT YEGAJ ZMKIB AJYDV MFFYH ZOWSW SQYMK CEZXK DBLEA GZTIF IHHNQ PARET PSOXE JPRHO RXLYY GSIHG YBIFC NYUSN JSDXF TGHIX KVWVQ GNWBC CCPFU MKOLT PMLDX DCMSX BEGEN USMUQ BJSJC OEREZ SZ"

# Provided information
plugboard_wiring = {'Q': 'K', 'J': 'O', 'L': 'U', 'X': 'G', 'D': 'V'}
reflector_type = 'UKW-B'
first_5_pi = 'CNO'
metasploit_date = 'JTI'

# Generate all possible combinations
all_combinations = []

# 1. Plugboard wiring
plugboard_permutations = permutations(plugboard_wiring.keys())
for plugboard_permutation in plugboard_permutations:
    plugboard_settings = dict(zip(plugboard_wiring.keys(), plugboard_permutation))

    # 2. Rotor order
    rotor_labels = ['I', 'II', 'III', 'IV', 'V']
    rotor_order_permutations = permutations(rotor_labels, 3)
    for rotor_order_permutation in rotor_order_permutations:
        rotor_order = ' '.join(rotor_order_permutation)

        # 3. Reflectors
        reflector_types = ['UKW-B']
        for reflector_type in reflector_types:

            # 4. Initial positions
            initial_position_permutations = permutations('ABCDEFGHIJKLMNOPQRSTUVWXYZ', 3)
            for initial_positions_permutation in initial_position_permutations:
                initial_positions = ' '.join(initial_positions_permutation)

                # Attempt decryption
                decrypted_message = enigma_decrypt(encoded_message, rotor_order, reflector_type, plugboard_settings, initial_positions)

                print(decrypted_message)
                print("-----")
```
#### Ransom 
This one looks tough, coz there are only 3 people who managed to solve it, but it's not!. I managed to solve it in 15m!.

- `ransom` the binary
- `important_company_data_backup.zip.ransomed`: At the first glance I thought it's kinda of compressed randomed file so I tried to extract it but it didn't work!.

1. <b>Basic analysis</b>

{{< image src="ransom_basic.png" caption="Basic analysis using file & strings" >}}

Maybe static analysis is the way to go ?, so I tried `radare2`:

{{< image src="radare2.png" caption="Creepy huh?" >}}

After some playing around I found out that `ransom` checks the stats of a file called `encrypt_me`

{{< image src="strace.png" caption="What happens when running the program ?" >}}

2. <b>Actual work</b>
I renamed the ransomed data file to `encrypt_me` and run the ransom bin, and Voilà something happened.

{{< image src="ransom_ans.png" caption="The flag !!?" >}}

As we can see, running the ransomware on that file modified it and it's compressed file of type `bz2`. To find the flag, just extract it and cat the `flag.txt`.

#### XOR 
Another interesting question, XOR is always challenging to work with, this time it's kinda different as I didn't want to write code lol. That's why I used [dcode](https://www.dcode.fr/xor-cipher) as we also know the length of the key and we need to find that key.

{{< image src="dcode_clues.png" caption="That's a clue!" >}}

I asked chatGPT since AI is really good finding patterns, and here is what I got:
- AI answer : `"It has been discovered that Cj+ provides remarkable facilities"`

With quick googling : I found this [wiki](https://en.uncyclopedia.co/wiki/C%2B%2B#Code_Example):

{{< image src="c++_quote.png" caption="That's a clue!" >}}

We have the decoded message now : `It has been discovered that C++ provides a remarkable facility for concealing the trivial details of a program...`

Finally I used [this](https://github.com/hellman/xortool) to get the key ! (as far as I remember lol)
### Programming
#### Birthday
When I saw that it only gives you 5 points I thought it's going to be easy and I was wrong!, it took me so much time as I didn't know how to properly send the request lol, and what to do as the challenge leave you no where near the solution with really ambiguous hint : `[Birthday]: Yes, you need the year too. Hint: I'm alive and far from retirement.`

Let's get started:
- First Inspect the network : you will find that there exist a GET request when you first click on the challenge which fetches the data (date hashed in `SHA-256` and don't ask me how I know that!)

{{< image src="response_birthday.png" caption="SHA-256 date" >}}

- Try to crack the hash:
The concept is easy as the hint suggests `"far from retirement"`, I will try all possible dates lol to `1900` hehe.

```python
import hashlib

def calculate_sha256_hash(data):
    sha256 = hashlib.sha256()
    sha256.update(data.encode('utf-8'))
    return sha256.hexdigest()

def check_birthday_hash(target_hash):
    for year in range(1900, 2023):  # Adjust the range based on reasonable birth years
        for month in range(1, 13):
            for day in range(1, 32):
                try:
                    birthday = f"{month:02d}/{day:02d}/{year}"
                    calculated_hash = calculate_sha256_hash(birthday)
                    if calculated_hash == target_hash:
                        return birthday
                except ValueError:
                    continue

    return None

if __name__ == "__main__":
    provided_hash = "6c833595c7502119465895442b340b3118a8c1aec222882d35e364f75e57b268"
    birthday = check_birthday_hash(provided_hash)

    print(f"Found birthday: {birthday}")
```

Now what?!, I got the desired date (here I got stuck lol, I didn't know how to send to the server lol). The answer is so easy but I was so dumb lol.

{{< image src="answer_birthday.png" caption="And that's when I know, I am so stupid lol" >}}

And btw, I am the first one to solve it :D

{{< image src="solved_first_birthday.png" caption="I am the first one to solve it :D" >}}

#### Secure OTP
That was an easy one, once you got all the hints which btw can be found by submitting many times with wrong answers or seen in the response which first clicking the problem card, anyway let's jump to the solution.

The hints : seed is given, digits are random int from 0 to 9. So based on these 2 factors we can regenerate the OTP.
```python

import random

seed = 1701957089
def generate_otp(seed):
    random.seed(seed)
    otp = ""
    for _ in range(6):
        digit = random.randint(0, 9)
        otp += str(digit)
    return otp

otp = generate_otp(seed)
print(f"Generated OTP with seed {seed}: {otp}")
```
PS: make sure to submit before the timer.
