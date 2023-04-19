# **Mac OS keygen guide**
In this quick tutorial, you will configure a user to use key-based authentication for SSH under a macOS desktop. 
I have also included how to put those keys into your favourite linux servers and haow to only allow key based log on your debian based machines


### **macOS configuring SSH Key-based Authentication**
---

The procedure to configuring SSH Key-based Authentication under macOS is as follows:

1. Log in to your macOS-based system.
2. Open the terminal application.
3. Use the ssh-keygen command to generate SSH keys. For extra protection for the private key, do enter a passphrase:

```
Yourname@MacBook ~ % ssh-keygen
```
4. Use the ssh-copy-id command to send the public key of the SSH key pair to Yourname on [subdomain.yourserver.yourdomain] / [yourserver ip address]

```
Yourname@MacBook ~ % ssh-copy-id yourname@subdomain.yourserver.yourdomain

Yourname@MacBook ~ % ssh-copy-id yourname@serverIP
```

5. Test by executing the date command on server1.cyberciti.biz remotely using SSH without accessing the remote interactive shell:

```
Yourname@MacBook ~ % ssh-copy-id yourname@subdomain.yourserver.yourdomain date
```

6. Please witness that the ssh command cued you for the passphrase you used to protect the private key of the SSH key pair. This passphrase shields the private key. Should an attacker or malware gain access to the ssh private key, they cannot use it to access other systems because the private key is protected with a passphrase. The ssh command uses a different passphrase than the one for the Yourname user on subdomain.yourserver.yourdomain, requiring users to know both. This is a security feature and recommends for all macOS users.

7. Is typing passphrase cumbersome for you each time you use the ssh command? Fear not; you can use ssh-agent command, as in the following step, to avoid interactively typing in the passphrase while logging in with SSH. In other words, using ssh-agent is more convenient and secure when logging in to remote Linux or Unix systems regularly for maintenance or troubleshooting.


### **Simple Steps to Set up Passwordless SSH Login on Debian based systems** 

---

There’re basically two ways of authenticating user login with OpenSSH server: password authentication and public key authentication. The latter is also known as passwordless SSH login because you don’t need to enter your password.

Since we have created our private and public key above on Mac OS above, we will be proceeding to log into the server and disabling the password based authentication

1. To disable password authentication, edit /etc/ssh/sshd_config file on the remote server.

```
sudo nano /etc/ssh/sshd_config
```

2. Find this line:

```
PasswordAuthentication yes
```

3. And change it to
```
PasswordAuthentication no
```

4. Then find the ChallengeResponseAuthentication line. Make sure it’s value is set to no like below. If it’s set to yes, you can still use password to login.

```
ChallengeResponseAuthentication no
```
Save the file and restart SSH service.

#### **Debian/Ubuntu**

```
sudo systemctl restart ssh
```

#### **RHEL/CentOS**
```
sudo systemctl restart sshd
```

Now if you don’t have the corresponding private key in ~/.ssh directory, you will see the following error when you try to SSH into your remote server.
```
Permission denied (publickey).
```
or
```
Read: Connection reset by peer
```

That means the remote server only allow SSH login using ssh keys and do not allow password authentication. Note that if you set `PasswordAuthentication` to `no` and `ChallengeResponseAuthentication` to `yes`, then you can still login using password. To disable password login, both of them must be set to no.



