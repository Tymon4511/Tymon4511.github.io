## HTB Object - Writeup

This Active Directory box was especially challenging due to a firewall that blocked many connections, adding an extra layer of difficulty.  
The journey began with decrypting a password from Jenkins, which granted WinRM access as a user.  
From there, abusing ACLs became crucial for escalating privileges.  
Each step deepened my understanding of AD exploitation, making it a rewarding and insightful challenge.  

---

### Nmap 

#### The nmap scan revealed three open ports:

![obraz](https://github.com/user-attachments/assets/efd1d3c9-feb6-4852-9cbe-d5584e14695f)


#### Port 80 - Website 

Here’s what the website looks like:

![obraz](https://github.com/user-attachments/assets/282f048d-8c09-46d6-a32a-b91705981b8f)




