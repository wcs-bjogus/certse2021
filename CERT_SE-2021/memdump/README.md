## memdump:

Looking in the memdump4.7z file we can see a file named memdump4.dmp

```bash
file memdump4.dmp
memdump4.dmp: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style
```
Looks to be a Core dump file.  
Remembering Frissan's comment 
>Anyway, too late to go back... Is the Debian-dump ready?

Can this be a debian dump? 
```bash 
$ strings memdump4.dmp | grep -i 'debian'
4.9.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) ) #1 SMP Debian 4.9.82-1+deb9u3 (2018-03-02)
```

OK Lets get some new tools!  

```bash
$git clone https://github.com/volatilityfoundation/volatility
$cd volatility/tools/linux/
$wget https://raw.githubusercontent.com/volatilityfoundation/profiles/master/Linux/Debian/x64/Debian94.zip
$cp Debian94.zip volatility/plugins/overlays/linux/
```
Now we have a memory forensic tool and the right symobls/overlays to work with the memdump4.dmp file.  

Looking on what commands that has been run:  
```bash
$vol.py -f memdump4.dmp --profile=LinuxDebian94x64 linux_bash
Volatility Foundation Volatility Framework 2.6.1
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
     403 bash                 2021-08-03 10:20:47 UTC+0000   ls
     403 bash                 2021-08-03 10:20:59 UTC+0000   ./SuperSecretLogonTool
```

Looks like we have a Binary called *SuperSecretLogonTool*.  

List files within the memorydump:  
```bash
$vol.py -f memdump4.dmp --profile=LinuxDebian94x64 linux_find_file -L | grep SuperSecretLogonTool
Volatility Foundation Volatility Framework 2.6.1
             125 0xffff9c984e99e718 /var/tmp/tmp/home/user/SuperSecretLogonTool
```

Export the file:  
```bash
$vol.py -f memdump4.dmp --profile=LinuxDebian94x64 linux_find_file -i 0xffff9c984e99e718 -O SuperSecretLogonTool
```

Running the binary:  
```bash
$ ./SuperSecretLogonTool

      ,@@@@@@(    &&&&&&&&&&    %%%%%%%,     &&&&&&&&&&&&&(                 %@@@@@@,    @@@@@@@@@@@
   @@@*       %   @@            @@      @@&       *@@                     @@*      .    @@/
 /@@              @@            @@       @@       *@@                     @@            @@/
 @@               @@,,,,,,,,    @@     /@@,       *@@                      (@@@@/       @@&%%%%%%%
 @@               @@            @@##%@@@(         *@@                           (@@@.   @@/
 @@&              @@            @@      @@        *@@                              @@   @@/
  #@@*            @@            @@       @@       *@@    ,,,,,,,,,,,,,,   @       @@@   @@/
     %@@@@@@@@/   @@@@@@@@@@,   @@        @@,     *@@    ,,,,,,,,,,,,,,    %@@@@@@(     @@@@@@@@@@@
                                                         ,,,,,,,,,,,,,,
 ........................................................,,,,,,,,,,,,,,............................
                                                         ,,,,,,,,,,,,.
                                                            ,,,,,,,,
                                                               .,

Ange lösenord:
```

Looks like we need a Password.  
Strings on the memory dump  
```bash
strings memdump4.dmp | grep -A 2 "Ange l"
Ange l
senord:
h3mlig!
```

Entering h3mlig! as a password and we recive the respons:
>Du fann vårt hemliga lösenord. Men var är flaggan?

Using objdump to decomple the binary:
*objdump -M intel -d SuperSecretLogonTool*

Here we can see a function called print_flag:  
```bash  
objdump -M intel -d SuperSecretLogonTool  | grep flag
00000000004011d6 <print_flag>:
  4012a5:       74 12                   je     4012b9 <print_flag+0xe3>
  4012b7:       eb e9                   jmp    4012a2 <print_flag+0xcc>
  4012d1:       74 10                   je     4012e3 <print_flag+0x10d>
  401401:       eb 21                   jmp    401424 <print_flag+0x24e>
  401428:       79 d9                   jns    401403 <print_flag+0x22d>
```

Lets fire up GDB and see if we can jump to the print_flag function at 00000000004011d6:  
```bash  
$ gdb SuperSecretLogonTool
Reading symbols from SuperSecretLogonTool...
(No debugging symbols found in SuperSecretLogonTool)
(gdb) b main
Breakpoint 1 at 0x4014ed
(gdb) run
Starting program: /tmp/CERT-SE_CTF2021/memdump4/SuperSecretLogonTool

Breakpoint 1, 0x00000000004014ed in main ()
(gdb) set $pc = 0x4011d6
(gdb) c
Continuing.
Bra gjort! Här kommer flaggan: CTF[Stackars_Myrstack]
[Inferior 1 (process 14387) exited normally]
(gdb)
```

Flag:
```
CTF[Stackars_Myrstack]
```