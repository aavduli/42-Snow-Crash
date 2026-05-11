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
Next step: document `level01` in the same format.


# Any tips for other flags :

## Flag05
```
find a user flag05 owning openarenaserver in /usr/sbin
```
