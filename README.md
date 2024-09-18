# TryHackMe_Pickle_Rick_Write-Up

This is a walkthrough of my process completing the Pickle Rick room on TryHackMe.com.

---

# Planning and Reconnaissance
*In this room, we are roleplaying as Morty from the hit TV show, "Rick and Morty".* Defining our scope: Rick has turned into a pickle and needs 3 ingredients for a potion to turn back into a human. Unfortunately, these ingredients are hidden on his computer, and Pickle Rick has forgotten his password. Our task is to retrieve his password, access his account, and locate the 3 necessary files containing the 3 ingredients. Pickle Rick provided us with the target machine's IP address, 10.10.209.241.

# Scanning and Enumeration
We begin with NMap. Using the command "nmap -Pn -vvv 10.10.209.241" quickly checks the most common ports active on a machine. This scan reviews ports 22 (**SSH**) and 80 (**HTTP**) are open, and likely running these services. 

Attempting the command "ssh rick@10.10.209.241" reveals password login via SSH is prohibited. 

Let's check if there is a website running on that IP address, "HTTP://10.10.209.241". 

There is!

Let's review the website's HTML. In the body of the HTML, we can see Rick left his username, -. Checking the directories of the webpage, we see there are some files. Opening these up only reveals some files, however, one, in particular, is a portal. Attempting to access this, unfortunately, doesn't reveal anything so let's dig deeper.

