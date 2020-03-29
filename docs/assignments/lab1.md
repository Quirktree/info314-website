# Lab 1 - Core Technical Skills

[Lab 1 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369614)

# Overview

In this lab you will be introduced to working with a headless Linux server. The work you do in this lab will be extremely helpful in becoming more comfortable with your Raspberry Pi.

We will walk you through all of the required steps in lab, but all the required steps are also in this webpage.



# Before you start

## Everyone

Confirm that you have created a DigitalOcean account and applied your GitHub developer credits to the account. This will allow you to create virtual machine “Droplets” on DigitalOcean without incurring any direct cost.

In addition to the email we sent out last week, instructions to complete this task can be found in the Course Preparation assignment guide linked from Canvas.

## Windows

Determine which build of Windows 10 they have installed by running Get-ComputerInfo -Property WindowsVersion in a PowerShell console. 

The minimum version of Windows 10 for this course is 1809, but you may proceed with this lab as long as you are running 1803.

For any prior version of Windows, please install Git for Windows from <https://gitforwindows.org/> and use the Git Bash environment to complete the following set of exercises.

# Critical Concepts

- Users and home directories
- File ownership and permissions
- Root user, admin privileges, and sudo
- Relative vs absolute path
- Services and the systemctl command
- Installing pre-packaged software using apt install
- OpenSSH client utilities, including ssh and scp

# Create SSH client keys

1. With either terminal (for Mac) or command line (for Windows) run the following command: ssh-keygen -t ed25519 -C *YOUR@EMAIL*
2. You will be prompted for a file name for the private key. You may accept the default.
3. You will be prompted for a passphrase. This is important as it is used to protect your private key.

By default, your private key should be stored in your home/user directory under ~/.ssh/id_ed25519. This version of the key will be encrypted and protected with your passphrase. 

You’ll also find a public key in ~/.ssh/id_ed25519.pub.

***(OPTIONAL STEP)*** To avoid having to enter the passphrase every time you use your private key, set up *ssh-agent* as describe in the following tutorial: 

<https://github.com/i314-campbell-wi19/project-tutorials/blob/master/ssh-agent.md>

# Create a Debian 9 Server on DigitalOcean

Log into your DigitalOcean account and create a new “Droplet” based on the following parameters:

| **Image**    | Debian 9.7 x64                                               |
| ------------ | ------------------------------------------------------------ |
| **Plan**     | Starter / $5 per month                                       |
| **SSH Keys** | Public key created in the [previous step](https://docs.google.com/document/d/1hGmM2T07K3A2__Dnb5X9c6ShSYGLvBZAA-v-ax-clh8/edit#heading=h.ig5dgivfw9j0). |

Wait for the droplet to be created and make note of the IP address for use in the next section.

## Connect and Manage your Server

### Log in to the droplet as the root user via SSH

In order to manage our remote server, we’ll use SSH to connect remotely. Since we have already associated a set of SSH keys with the server, we will be able to log in the the server as the root user without directly entering a password.

The syntax for ssh is ssh <USERNAME>@<SERVER>, e.g., ssh root@134.209.4.234.

Use the script command to capture your session with script root-session. Everything you type in this shell will now be captured in a file named root-session within your home directory. Since ssh always launches in the home directory of the logged in user, you should determine the path by running pwd now.

### Create a second user account

In most situations, we will not work directly as the root user, since this would pose additional security risks. In fact, many Linux distributions will prevent direct root login. Let’s create a new user and practice working with this configuration. 

Add a new user adduser clinton

Add the user to the *sudo* group usermod -aG sudo clinton

By default, DigitalOcean prevents users from connecting via SSH without an SSH key. This is the correct decision from the security perspective, but we will disable it temporarily in order to explore beneath the hood.

Modify /etc/ssh/sshd_config to enable password-based login by opening it in a terminal-based editor, e.g., nano /etc/ssh/sshd_config.

Find the PasswordAuthentication no setting and prefix it with a # comment character.

Settings don’t take effect automatically. Use the systemctl tool to restart the sshd service, i.e., systemctl restart sshd.

### Log in as your new user

In a new terminal window, log in as the new user via SSH using the password you created above.

Capture a script of your session by calling script  user-session. The results of this command will be saved in a file named user-session.

When you log in, run pwd and make note of the directory. This is your home directory. Notice that the home directory for each user is different.

Try listing the contents of the root user’s home directory by typing ls -al /root. You should receive a permission denied error. The /root path is owned by the root user and has permissions restricted so that other users cannot read, write, or execute the directory or anything else it contains.

Since root is a special user, the restriction does not apply in the other direction. Demonstrate this difference in permissions by listing the contents of the new user’s home directory from the root user’s shell, e.g., ls -al /home/clinton from your root login and view the contents of my user’s home directory.


By default in basic Linux distributions, the root user has complete control over all system and user resources and is even able to take on the identity of other users without knowing their passwords.

#### Running Administrative Commands

Many of the administration tasks we need to complete throughout the quarter require root level permissions. Since we’ve already established that we will deliberately work as a non-root user, we should determine a method to elevate our privileges. In Linux and other Unix-based operating systems, the command that allows us to do this is called sudo. By prefixing any valid shell command or program name with sudo, we will assume the identity of root at runtime.

Test this out by comparing the results of whoami with the results of sudo whoami.

The next step will require us to log out of our current ssh session. Before you do this, type CTRL-D to end the current script session. You should see a message stating that the script is complete. You may now log out of ssh by running the exit command.

### Add SSH keys for your user

As you’ve seen, we can log into the root account without entering a password because of the SSH keys that we created at the beginning of this lab, but logging into our new user account requires a password (which is a much weaker configuration from a security perspective).

Let’s resolve this by adding our public ssh key to the new user account on our Droplet. First switch back to your root shell and examine the files saved in the .ssh folder of root’s home directory. Remember that you can use ls -al to view a folder. 

The additional options given after the command specify that hidden files (beginning with a period) will be displayed and that the details about each file or directory will also be shown.

Note that you can also use the cat command to view the contents of a file in that directory.

What you should see in the specified path is a file named authorized_keys that contains a copy of your SSH public key on a line by itself. We’ll be creating a similar file in the home directory of our new user. On each login attempt, the SSH server checks for authorized_keys designated for the user and loads 

You can also use ssh to remotely execute the command without having to open a full SSH session:

ssh clinton@134.209.4.234 'mkdir -p ~/.ssh'

With the directory in place, copy your public key (most likely ~/.ssh/id_ed25519.pub) to the Droplet using the scp command. scp is part of the OpenSSH client package and is used to copy files between paths on local and remote hosts. You will notice that it uses ssh syntax to identify the remote location followed by a colon and the actual source or target path at the remote location. 

**NOTE: The following command is entered as one line but is wrapped due to the constraints of the editor.**



scp $HOME/.ssh/id_ed25519.pub clinton@134.209.4.234:.ssh/authorized_keys

Be aware that scp respects file permissions. When connecting as my non-root user, I cannot read or write to locations that are restricted to other users or root. 

### Install a web server

Test that you successfully copied your public key to the server and can access the Droplet as your non-root user without having to enter a server password.

Once you are logged back in, resume your script by running script -a user-session. Note the addition of the -a parameter which will cause script to append to the existing file.

Install the nginx web service using the command sudo apt install nginx. 

Verify that the service installed correctly by running systemctl status nginx and confirming that the nginx service is loaded and running.

Use a web browser to navigate to your IP address and load the default nginx site, e.g., <http://134.209.4.234> 

Once you have completed these tasks, please close out the scripts from your root and user shell and use 

scp 

to transfer them to your computer.