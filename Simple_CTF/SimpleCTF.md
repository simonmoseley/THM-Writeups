# TryHackMe - [Simple CTF](https://tryhackme.com/room/easyctf)

## This lab is a simple, beginner level CTF with plain and simple instructions.
### Deploy the machine and attempt the questions! 

### The questions are: 

1. How many services are running under port 1000?
2. What is running on the higher port?
3. What is the CVE you're using against the application?
4. To what kind of vulnerability is the application vulnerability?
5. What is the password?
6. Where can you login with the details obtained?
7. What's the user flag?
8. Is there any other user in the home directory? What is its name?
9. What can you leverage to spawn a privileged shell?
10. What's the root flag?


## Enumeration!

I begin by running Nmap on the IP address we are given to see what ports are open. I also send the output to an initial_scan file.


![nmap1](https://user-images.githubusercontent.com/90977933/134067866-59186214-18a9-4d9c-88d4-ae4de982122b.png)


The results of the scan show 3 open ports. 21, 80 and 2222. With this I can answer **question 1.**
I now run nmap again with the -A flag, on those 3 ports to see what additional info we can find.


![nmap2](https://user-images.githubusercontent.com/90977933/134067903-51c16fa5-0269-4b38-be82-380c1ae45ff4.png)


Ok, lots of good info here. From the top I can see the FTP service on port 21 allows anonymous logins, there is an Apache web server running on port 80 with a robots.txt file, and The SSH service is running on port 2222. With this we can answer **question 2.**

Now I enter the IP address into my browser to see what's there. Its an Apache2 Default page.


![browser1](https://user-images.githubusercontent.com/90977933/134068007-34eeed00-e0b9-42bd-9424-fe545ae4bfbf.png)


Remembering the scan results, I visit the robots.txt file and find what looks like a note explaining what the Robots.txt is used for.


![Robots1](https://user-images.githubusercontent.com/90977933/134068654-925ca810-d763-4281-89e6-a28b59968da4.png)


Visiting the disallowed directory is a dead end, so I move on and use gobuster to see what hidden directories, files or redirects I can find.


![gobuster1](https://user-images.githubusercontent.com/90977933/134069180-182bbd79-77e0-4d76-a94b-e0de358f2f01.png)


In our gobuster results there is a directory called /simple. 
 

![gobuster2](https://user-images.githubusercontent.com/90977933/134069224-4e3f9849-3851-4e3c-8657-11aafee5abc9.png)


Navigating to it we find the CMS Made Simple CMS.


![Browser2](https://user-images.githubusercontent.com/90977933/134069281-4cc6b40d-43d7-41e7-a6fc-753d3857e463.png)


Since there's a CMS here, there must be a login portal somewhere that could be a way to gain access to the web server. I run gobuster again against the new directory to see what else I can find. In the results I find a /admin directory, navigating to it I find a login portal


![browser3](https://user-images.githubusercontent.com/90977933/134069408-3341c6bc-4c6f-4c0d-b6e0-6f15a557fb7f.png)


We dont have a username or anything yet so I decided to try a different route. I stop with the web portion here and go to see what I can find with FTP using the anonymous login. (probably should have explored this way first but...)
After logging I find a directory called pub.


![ftp1](https://user-images.githubusercontent.com/90977933/134069901-5d59985c-1e4e-489d-b710-03428961a2ba.png)


In it I find a txt file called ForMitch. Lets GET it and see what it says.


![ftp2](https://user-images.githubusercontent.com/90977933/134069947-cdbee143-5f5b-4ff5-9e12-b37a20161ab3.png)
![ftp3](https://user-images.githubusercontent.com/90977933/134069975-5260a7b0-c515-429b-93cd-4ffd58968a67.png)



In the file we get a message for Mitch with a huge hint!



![mitch](https://user-images.githubusercontent.com/90977933/134070160-52f2748a-c3b1-4165-a77b-31661961fe5d.png)



So now we have a possible username and that Mitch uses a weak password repeatedly. Always a big no no!
My first thought, why not try hydra to try and bruteforce SSH using Mitch as the username.  

### SUCCESS!!


![hydra1](https://user-images.githubusercontent.com/90977933/134070805-f56812a4-d93c-40c8-90bb-75da810808dc.png)


Ok so from here we can answer **questions 4** and **5**. But we skipped **2** and **3** so lets I go back


Going back to the the Simple CMS directory we started at, all the way at the bottom at the bottom of the page that the version is 2.2.8


![cms1](https://user-images.githubusercontent.com/90977933/134071358-014c117d-6e65-46df-839e-9ac158c7c10f.png)


I use searchsploit to see if there are any known vulnerabilities and find a python SQLI [exploit](https://www.exploit-db.com/exploits/46635) for CMS Made Simple. Looking at it shows the answers for questions 3 and 4.


![exploit1](https://user-images.githubusercontent.com/90977933/134072190-87fd2b3b-7785-458f-9a51-88ee94aecf1c.png)
![exploit2](https://user-images.githubusercontent.com/90977933/134072228-03269914-f0a7-4c6e-9935-1856fef004da.png)


## Gain Access!


This exploit will give me the username and password I would need to gain accesss through SQLI. Since I already have the Username and password for SSH login, the steps above were only to answer the challenge questions. Getting back on track, I use the credentials we found with hydra to login to the machine through ssh.


![ssh1](https://user-images.githubusercontent.com/90977933/134074685-b651bdc2-81a8-4d72-aec2-f7bd84e8d743.png)


In Mitch's home folder I find the **user.txt** file that contains the user flag and the answer to **question 7.** 


![userFlag](https://user-images.githubusercontent.com/90977933/134074855-fc4b2b63-6d5a-49a5-accf-729b09ddced4.png)


**Question 8** asks if there are other users. I move up one level and list the contents.


![ssh2](https://user-images.githubusercontent.com/90977933/134075108-9ed77661-f235-469d-a748-0de97e0dfeef.png)



## Privesc!


Running *sudo -l* shows Mitch can run */usr/bin/vim* as root with no password. Also the answer to **question 9.**


![ssh3](https://user-images.githubusercontent.com/90977933/134075454-a74fc54d-e383-471f-884b-371bd378e989.png)


Finally, I use vim to escalate my privileges to **root.** Run the command *sudo /usr/bin/vim*. From there I spawn the root shell by typing *:!sh*. I type *whoami* to verify and **BAM** I am root! 



![pe1](https://user-images.githubusercontent.com/90977933/134075938-bac2d54e-8b5e-4661-8206-20b453ab6870.png)
![pe2](https://user-images.githubusercontent.com/90977933/134075948-c87d06c1-d6a7-40da-b9bf-4321d345607f.png)
![pe3](https://user-images.githubusercontent.com/90977933/134075957-2e441bd8-ec5e-4021-8831-3bf76741d9a8.png)



Now as **root** I can navigate to the root folder, cat our root flag and answer the final question.

![rootFlag](https://user-images.githubusercontent.com/90977933/134076044-7d2f9d1e-1aa5-4a39-a05d-b034a3540dc8.png)


-Simon 
