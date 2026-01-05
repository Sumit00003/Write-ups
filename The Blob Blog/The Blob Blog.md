###### **Recon**

First we start our Enumeration with **Nmap Scan** - *got ports 80 & 22*

![nmap](../images/Pasted image 20260105114259.png)

Got nothing in Directory Brute-forcing on ports 80 apache server, but there was some encoded string in the page source.

![[Pasted image 20260105114349.png]]

We pasted the string in Cyberchef , and got another encoded string. After some research on the string we got to know it was encoded in Brainf*ck.

![[Pasted image 20260105114459.png]]

Decoding the another encoded string gave us a sentence which hint for **Port Knocking**

![[Pasted image 20260105114657.png]]

so we tried to knock as mention in the hint
```┌──(root㉿kali)-[/home/kali]
└─# knock 10.66.136.199 1 3 5
```
And now we have more open ports available - *8080, 445, 21*

![[Pasted image 20260105114800.png]]

 There was another hint on the same apache server on port 80, which mention the username Bob and some encoded string which probably is the password.

![[Pasted image 20260105114956.png]]

After decoding the string with **Base58**, we got the password.

![[Pasted image 20260105115046 1 1.png]]
Then we tried to used above decoded password & username in the ftp server.

![[Pasted image 20260105114842.png]]

There was a image file in the ftp server, we downloaded that and used steghide tool to extract files if it contain some. But it required a password to extract the file.

![[Pasted image 20260105115159.png]]

After more Recon, we got another hint in the web page on port 445 which was not running SMB-Server, it was running a Apache server.

![[Pasted image 20260105115232 1.png]]

We used the password and extracted *out.txt* file, which contain a directory location and an encrypted string which look like username:password.

![[Pasted image 20260105115301.png]]
```┌──(root㉿kali)-[/home/kali]
└─# cat out.txt 
zcv:p1fd3v3amT@55n0pr
/bobs_safe_for_stuff
```

We visited /bobs_safe_for_stuff location and got a hint there. After some recon we got to know that the encrypted string is a **Vigenere Cipher** and may be the hint we got is the key for that string.

![[Pasted image 20260105115418.png]]

We visited cryptii.com to dicpher the string. And we got password for user Bob.

![[Pasted image 20260105115449 1.png]]

With the extracted credentials, we tried to login on the port 8080. By using feroxbuster we know that port 8080 contain 3 directory --> /login, /review, /blog

![[Pasted image 20260105115535.png]]

There was nothing special in the website, but by seeing the input field, we tried to do command execution.


![[Pasted image 20260105120122.png]]
And it work's

![[Pasted image 20260105120155.png]]
So we executed basic bash reverse shell to get the reverse connection from the system.

![[Pasted image 20260105120324.png]]
![[Pasted image 20260105120302.png]]

###### Privesc

After getting the shell access, we checked for any file or binary that have SUID bits set
```kali
find / -perm -u=s 2>/dev/null
```

![[Pasted image 20260105120441.png]]

We downloaded the blogFeedback Binary file and used cutter tool to decompile it 


![[Pasted image 20260105120528.png]]
According to the above mention, we pass 6 arguments in reverse order to execute the /bin/sh and get the privilege shell.

![[Pasted image 20260105120631.png]]

And it works and we got our First Flag.

![[Pasted image 20260105120724.png]]

After we got the shell, below given string was appearing in the terminal again and again.

![[Pasted image 20260105120756.png]]

So we downloaded pspy tool using python server in target machine.

![[Pasted image 20260105120926.png]]

And after running the tool, we got a process running, which was compiling a C file then providing execution permission to it then running it as a root and then deleting it.
```process
/bin/sh -c gcc /home/bobloblaw/Documents/.boring_file.c -o /home/bobloblaw/Documents/.also_boring/.still_boring && chmod +x /home/bobloblaw/Documents/.also_boring/.still_boring && /home/bobloblaw/Documents/.also_boring/.still_boring | tee /dev/pts/0 /dev/pts/1 /dev/pts/2 && rm /home/bobloblaw/Documents/.also_boring/.still_boring
```

So we checked if we have permission to write into the file and We have that.
So we searched for a Reverse Shell Code in C language and removed the C file from the target machine and placed our shell code with the same name.

![[Pasted image 20260105121101.png]]

And we got the shell with root privilege and also we got our Root Flag.


![[Pasted image 20260105121148.png]]


