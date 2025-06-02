## HTB Object - Writeup

Difficulty: Hard

This Active Directory box was especially challenging due to a firewall that blocked many connections, adding an extra layer of difficulty.  
The journey began with decrypting a password from Jenkins, which granted WinRM access as a user.  
From there, abusing ACLs became crucial for escalating privileges.  
Each step deepened my understanding of AD exploitation, making it a rewarding and insightful challenge.  

---

## Nmap 

#### The nmap scan revealed three open ports:

![obraz](https://github.com/user-attachments/assets/efd1d3c9-feb6-4852-9cbe-d5584e14695f)


## Port 80 - Website 

Here’s what the website looks like:

![obraz](https://github.com/user-attachments/assets/282f048d-8c09-46d6-a32a-b91705981b8f)

Right from the initial enumeration, an email address and domain name are revealed.  
Let’s make a note of the email for later and add the domain **object.htb** to /etc/hosts.  
At this point, we can try directory busting, but it didn’t yield any results.  

Lastly there is a link that points to **http://object.htb:8080/**.
  
  
  
  

## Port 8080 - Jenkins

This website hosts a Jenkins instance.  
We can try common default credentials such as admin:password, jenkins:jenkins, and similar combinations, but they didn't work.


<p align="center">
  <img src="https://github.com/user-attachments/assets/3d1d8667-bbe8-443d-a48c-753cf09f64e4" alt="obraz">
</p>

After creating an account, we gain access to the dashboard and notice it’s running version 2.317 of Jenkins.  
We could potentially abuse the Script Console using a Groovy-based reverse shell, but access to it requires administrative privileges.  
My second thought was to create a Jenkins job that, when built, executes a Windows command, eventually leading to shell access.  
This is a popular way to execute commands in Jenkins, let's try that:



