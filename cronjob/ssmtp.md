
https://www.atlantic.net/vps-hosting/how-to-use-ssmtp-to-send-an-email-from-linux-terminal/

Step 3 – Configure SSMTP
Next, you will need to define your Gmail or other SMTP servers in SSMTP configuration file. You can define it in the /etc/ssmtp/ssmtp.conf file:

nano /etc/ssmtp/ssmtp.conf
Add the following lines:

FromLineOverride=YES
root=postmaster
mailhub=smtp.your-domain.com:587
hostname=ubuntu2004
AuthUser=hiteshjethva@your-domain.com
AuthPass=your-password
FromLineOverride=YES
UseSTARTTLS=YES
Save and close the file when you are finished.

SSMTP is now configured to use your SMTP server address to send an email.



https://vitux.com/three-ways-to-send-email-from-ubuntu-command-line/
f you know the real power of the command line, you wouldn’t want to leave the comfort of the Terminal and go somewhere else to do any of your daily technical activities. There is always a way to do almost all of our stuff right inside the Terminal. So, why should sending emails be any different! Using the Terminal makes certain tasks more efficient and even faster. The command-line tools do not use too many resources and thus form great alternatives to the widely used graphical applications, especially if you are stuck up with older hardware. Sending emails from the Terminal becomes especially handy when you can write shell scripts to send emails and automate the whole process.

In this article, we will describe three ways through which you can send email on the Ubuntu command line (from your configured email ID).
ssmtp command
sendmail command
mutt command
We have run the commands and procedures mentioned in this article on a Ubuntu 18.04 LTS system.

Open the Terminal application either through the application launcher search bar, or the Ctrl+Alt+T shortcut, and then use one of the following methods for sending emails.


Method 1: Send email with ssmtp command
ssmtp is a send-only sendmail emulator for machines that normally pick their mail up from a centralized mail hub (via pop, imap, nfs mounts or other means). It provides the functionality required for humans and programs to send mail via the standard or /usr/bin/mail user agents. If your system does not have this utility installed, run the following command to install it:

$ sudo apt- get update
And then,

$ sudo apt-get install ssmtp
The following command can then be used to compose and then send an email:

Setup
Update Ubuntu Repository


sudo apt-get update
Install the ssmtp package


sudo apt-get install ssmtp

configure ssmtp
Edit the configuration file. The lines without the # are the ones we are interested in.


sudo nano /etc/ssmtp/ssmtp.conf
Adjust and add as necessary to match the following parameters

Change "MyEmailAddress" and "MyPassword" to your own.


# Config file for sSMTP sendmail
#
# The person who gets all mail for userids < 1000
# Make this empty to disable rewriting.
#root=postmaster
root=MyEmailAddress@gmail.com

# The place where the mail goes. The actual machine name is required no
# MX records are consulted. Commonly mailhosts are named mail.domain.com
#mailhub=mail
mailhub=smtp.gmail.com:587

AuthUser=MyEmailAddress@gmail.com
AuthPass=MyPassword
UseTLS=YES
UseSTARTTLS=YES

# Where will the mail seem to come from?
#rewriteDomain=
rewriteDomain=gmail.com

# The full hostname
#hostname=MyMediaServer.home
hostname=MyMediaServer.home

# Are users allowed to set their own From: address?
# YES - Allow the user to specify their own From: address
# NO - Use the system generated From: address
FromLineOverride=YES

$ ssmtp username@domain.com
Hit Enter and then input the subject in the following format:
Subject: sample subject comes here
As you hit Enter, you will be allowed to enter the body of the email. Once you are done with entering the email body, hit Ctrl+D. This will mark the end of the email body and send it to the respective receiver ID.


https://www.techrepublic.com/article/use-ssmtp-to-send-e-mail-simply-and-securely/

Use sSMTP to send e-mail simply and securely


Secure SMTP server authentication:
Not only is sSMTP a simple, straightforward tool for handling outgoing mail, but it is a secure tool as well — when used properly. An important component of e-mail security, in addition to use of digital signatures and e-mail encryption, is protecting your authentication exchanges for connections to SMTP and incoming mail servers. Whenever you connect to any kind of mail server, you should be using a username and password to authenticate yourself:

