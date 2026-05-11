# Snow-crash (42)

This repository contains my notes and solutions for the **Snow-crash** project at 42.

## Access

Use SSH to connect to the first level:

```bash
ssh level00@192.185.64.5 -p 4242
```

## Global workflow: 
For the upcoming levels, the pattern will often involve finding a `vulnerability in a script or binary` owned by `flagXX`. Once you exploit it to `run commands as that user`, you run `getflag` to get the `password` for the next level.

## Level 00

### Goal
Find the password for `flag00`, then use it to retrieve the token for the next level.

### Steps

Navigate through the VM and inspect interesting directories.
In `/usr/sbin`, list files and permissions:
```bash
ls -la
```
Find a file named `john`, owned by `flag00`.
Read its content:
```bash
cat john
```
Output:
```text
cdiiddwpgswtgt
```
The string is not directly usable as a password. Decode it with ROT11:
```bash
echo "cdiiddwpgswtgt" | tr 'a-z' 'l-za-k'
```
Result:
```text
nottoohardhere
```
Switch user to `flag00` with the decoded password:
```bash
su flag00
```
Run:
```bash
getflag
```
Token obtained:
```text
x24ti5gi3x0ol2eh4esiuxias
```
Use this token as the password for `level01`.
---
## Level 01

### Observations

- Running `find / -user flag01` or `find / -group flag02` returned no useful results.
- However, `/etc/passwd` is readable and shows the `flag` users and hashes:

```text
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
flag02:x:3002:3002::/home/flag/flag02:/bin/bash
flag03:x:3003:3003::/home/flag/flag03:/bin/bash
flag04:x:3004:3004::/home/flag/flag04:/bin/bash
flag05:x:3005:3005::/home/flag/flag05:/bin/bash
flag06:x:3006:3006::/home/flag/flag06:/bin/bash
flag07:x:3007:3007::/home/flag/flag07:/bin/bash
flag08:x:3008:3008::/home/flag/flag08:/bin/bash
flag09:x:3009:3009::/home/flag/flag09:/bin/bash
flag10:x:3010:3010::/home/flag/flag10:/bin/bash
flag11:x:3011:3011::/home/flag/flag11:/bin/bash
flag12:x:3012:3012::/home/flag/flag12:/bin/bash
flag13:x:3013:3013::/home/flag/flag13:/bin/bash
flag14:x:3014:3014::/home/flag/flag14:/bin/bash
```

The second field for `flag01` (`42hDRfypTqqnw`) is an encrypted password/hash.

### Approach

1. Try common hash/cracking approaches (MD5, SHA variants, bruteforce, ROTs) — no success.
2. Remember the `john` file found in Level 00: it is likely a hint to use `john` (John the Ripper).

### Commands

Use `john` with an appropriate wordlist to crack the hash (example with `john` installed locally):

```bash

echo '42hDRfypTqqnw' > hashes.txt

john hashes.txt
```
On this box, `john` cracked the password quickly:
```
abcdefg
```
### Result
Switch to `flag01` and retrieve the flag:
```bash
su flag01
getflag
```
Flag obtained:
```text
f2av5il02puano7naaf6adaaf
```
Use this flag/token as the password for `level02`.
---
Next: add `level02` notes in the same style.


# Any tips for other flags :

## Flag05
```
find a user flag05 owning openarenaserver in /usr/sbin
```

## /etc
found
```
-rw-r----- 1 root shadow  4428 Mar  6  2016 shadow
-rw-r----- 1 root shadow  1119 Aug 30  2015 gshadow
```
