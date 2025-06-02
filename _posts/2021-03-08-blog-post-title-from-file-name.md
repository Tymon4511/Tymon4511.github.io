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

**Click on "New Item" -> "Freestyle Project", and scroll a bit down, and select "Build Periodically":**

![obraz](https://github.com/user-attachments/assets/0c054a9e-f114-4899-af57-3065f08fd866)


We want it to make a build every minute, meaning our command will be executed every minute also.  
**Scroll a bit down and "Add a build step" -> "Execute Windows Batch Command"**


![obraz](https://github.com/user-attachments/assets/6185423b-29c0-4afa-9990-8695048807fa)  
Then hit save and apply.

Now we wait for one minute and select: 
**last build -> console output -> check if we have command execution**

![obraz](https://github.com/user-attachments/assets/3e220dc1-4faf-4b76-9003-3df418ff247c)  

Now it's time to upload a reverse shell to the target machine, but here we discovered that it won't be possible likely to the firewall settings that block outbound connection.

![obraz](https://github.com/user-attachments/assets/8aede77d-3c9a-4342-869f-d85dca9cbb83)  

It's time to change approach, we're left with possibility to enumerate the target system.  
First we want to check jenkins home directory.  
We'll exeucte:  

![obraz](https://github.com/user-attachments/assets/36eec344-454a-4133-a86b-55b9f2a57853)

Output:

![obraz](https://github.com/user-attachments/assets/7faa5c88-58fe-4b35-a867-57c020e7bc85)

We've got two intresting directories: secrets, and users.  
First we'll check Users directory.  

![obraz](https://github.com/user-attachments/assets/b55729d4-5bdb-43fd-ac35-7ce8411f697c)

There is a user that we created previously and admin.  
Let's check admin directory:  

![obraz](https://github.com/user-attachments/assets/5055b9c1-1fad-4a15-b694-7007a7f8174c)  

We have a config file, lets see what's in it:  

![obraz](https://github.com/user-attachments/assets/cb7ee68e-4fe9-4132-80f2-e937b572e851)


File Contents:  
```
<?xml version='1.1' encoding='UTF-8'?>
<user>
  <version>10</version>
  <id>admin</id>
  <fullName>admin</fullName>
  <properties>
    <com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty plugin="credentials@2.6.1">
      <domainCredentialsMap class="hudson.util.CopyOnWriteMap$Hash">
        <entry>
          <com.cloudbees.plugins.credentials.domains.Domain>
            <specifications/>
          </com.cloudbees.plugins.credentials.domains.Domain>
          <java.util.concurrent.CopyOnWriteArrayList>
            <com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
              <id>320a60b9-1e5c-4399-8afe-44466c9cde9e</id>
              <description></description>
              <username>oliver</username>
              <password>{AQAAABAAAAAQqU+m+mC6ZnLa0+yaanj2eBSbTk+h4P5omjKdwV17vcA=}</password>
              <usernameSecret>false</usernameSecret>
            </com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
          </java.util.concurrent.CopyOnWriteArrayList>
        </entry>
      </domainCredentialsMap>
    </com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty>
    <hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty plugin="email-ext@2.84">
      <triggers/>
    </hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty>
    <hudson.model.MyViewsProperty>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>all</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty plugin="display-url-api@2.3.5">
      <providerId>default</providerId>
    </org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <jenkins.security.seed.UserSeedProperty>
      <seed>ea75b5bd80e4763e</seed>
    </jenkins.security.seed.UserSeedProperty>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>true</insensitiveSearch>
    </hudson.search.UserSearchProperty>
    <hudson.model.TimeZoneProperty/>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$q17aCNxgciQt8S246U4ZauOccOY7wlkDih9b/0j4IVjZsdjUNAPoW</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <hudson.tasks.Mailer_-UserProperty plugin="mailer@1.34">
      <emailAddress>admin@object.local</emailAddress>
    </hudson.tasks.Mailer_-UserProperty>
    <jenkins.security.ApiTokenProperty>
      <tokenStore>
        <tokenList/>
      </tokenStore>
    </jenkins.security.ApiTokenProperty>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1634793332195</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
  </properties>
</user>
```

This is a file that contain credentials and can be decrypted with a python script, but in order to do that we will need master.key and hudson.util.Secret which both can be found in **Secrets** directory.  

![obraz](https://github.com/user-attachments/assets/0fe4e98d-2575-4692-88d6-38103fadfe5c)

Output:  

![obraz](https://github.com/user-attachments/assets/e69f3363-d8b0-41ef-aef2-0346995f52b0)

We will copy this key to our kali linux.  
With getting hudson.util.Secret I had a problem but finally it worked when converted to base64 with powershell:  

![obraz](https://github.com/user-attachments/assets/0f7bdf6e-0df1-4cfb-9f45-6f9f03e80509)

Output:

![obraz](https://github.com/user-attachments/assets/415ba5c2-ea76-4dd2-86bc-6ebb84ebe281)

Now let's decode that and put in a file:  

![obraz](https://github.com/user-attachments/assets/19ae2ea9-21d9-435f-a710-c8b3d0857ad4)

Now we can put everything together and get plain text password with this script:

```
https://github.com/gquere/pwn_jenkins/blob/master/offline_decryption/jenkins_offline_decrypt.py
```




































