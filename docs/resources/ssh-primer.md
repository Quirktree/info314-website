# Getting Started with SSH

SSH stands for 'Secure Shell'. A 'shell' is a command line interface, such as the one's you've seen in Terminal, PowerShell, or Git Bash. Throughout this course, you'll be using the shell, remotely, to configure Linux. 

## Connecting to a Server

The basic syntax for ssh is:   

`ssh <USERNAME>@<HOST>`   

For example, in order to connect to a new install of Raspberry Pi OS, enter `ssh pi@raspberrypi.local`. If successful, you should receive a message and prompt that you must respond to in order to continue:

!!! example "Unknown Host Prompt"
    ```
    The authenticity of host 'raspberrypi.local (fe80::0f0e:0d0c:0b0a:0908%en9)' can't be established.
    ED25519 key fingerprint is SHA256:gy7+oAsjNc5gvP7aaR9k9Wl/oMdCLjAy2MJL8ZaQ9N0.
    This key is not known by any other names
    Are you sure you want to continue connecting (yes/no/[fingerprint])?
    ```

This message indicates that your local SSH client is attempting to authenticate the remote server. While SSH clients have the option to authenticate with a password, SSH servers are only able to authenticate themselves using keys. 

Since we've never connected to this server, the client is asking whether to trust the public key information that has been presented. If you accept, SSH will store a fingerprint locally and pull it up whenever you reach out to the same hostname or IP address. This mechanism is known as _Trust on First Use (TOFU)_.

Type `yes` and press ++return++ to continue. 

If the server authenticates you successfully, it should return a login message and a prompt. If authentication fails, you should be prompted for a password or disconnected (if the user doesn't exist or password authentication is disabled).

!!! example "Successful connection with client certificates"

    ```
    Warning: Permanently added 'raspberrypi.local' (ED25519) to the list of known hosts.
    Linux raspberrypi 5.10.103-v8+ #1530 SMP PREEMPT Tue Mar 8 13:06:35 GMT 2022 aarch64

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Thu Mar 31 00:32:28 2022 from fe80::0102:0304:0506:0708%eth0
    pi@raspberrypi:~ $
    ```

## Create SSH client keys

When you are connecting to a remote server, you'll need a way to authenticate yourself and login to your server. While you can use a standard password, it is not a smart security practice. Rather, it's important to learn to generate and use a public keypair for SSH.

A keypair is composed of a **private** key and a **public** key. Your **public** key can be given out, to say, GitHub or DigitalOcean, while your **private** key should never leave your device. 

OpenSSH is shipped with a utility called `ssh-keygen` that will generate a keypair for you.

**To create your SSH keypair, use `ssh-keygen` as follows:**

1. With either Terminal (for Mac) or PowerShell/Git Bash (for Windows) run the following command with your own email address:  
`ssh-keygen -t ed25519 -C <DESCRIPTION>`
1. When prompted for a file name for the private SSH key. Accept the default by pressing ++Enter++.[^location]
1. You will be prompted for a passphrase. This is important as it is used to protect your private SSH key. Make it something strong, like you would a password, and commit it to memory or write it down somewhere.

Assuming that you accepted the default location above, your keys will be saved to `~/.ssh/id_ed25519` (private) and `~/.ssh/id_ed25519.pub` (public).

***(OPTIONAL STEP)*** To avoid having to enter the passphrase every time you use your private key, continue to set up the *ssh-agent* as described [in this tutorial](/resources/ssh-agent.md).

[^location]: By default, your SSH keys will be saved to **`~/.ssh`**. If you are prompted to overwrite an existing key, you can cancel the process by typing `n` and pressing ++Enter++.

## Register Client Keys with a Server
In order to connect to a server using an SSH keypair, the server will need a copy of the public key to associate with a user account on the system. By default, each user will have a `.ssh` folder in their home directory. When a user attempts to log in via an SSH connection, the server will look for a file named `~/.ssh/authorized_keys` containing a list of public keys. The client proceeds to authenticate itself by proving to the server that it is in possession of a matching private key.

To use this feature on a Linux server, create the `.ssh` directory in your home directory (on the server). If you're already logged in, you can create the directory using `mkdir -p ~/.ssh`. After that is done, type `exit` to terminate the connection and return to your local command prompt.  

!!! tip "You are here!!!"
    The following commands are run from your **local** shell. Running them inside of an existing SSH session will not have the effect you are looking for.

With the directory in place, copy your public key (most likely `~/.ssh/id_ed25519.pub`) to the server using the `scp` command[^scp_syntax]

```bash
scp $HOME/.ssh/id_ed25519.pub pi@raspberrypi.local:.ssh/authorized_keys
```

[^scp_syntax]:
    The first parameter to scp specifies the name of the local file. The example is copying the public key named id_ed25519.pub from the current user's home directory.
    
    The second parameter to scp names the user (`pi`), the host (`raspberrypi.local`), and the remote destination for the file (`.ssh/authorized_keys`). 
    
    Like `ssh`, `scp` is a user-based tool and will be executed relative to the home directory of the authenticated user (`/home/pi`). We can override the default by specifying an absolute path beginning with a `/`, e.g., `/home/pi/.ssh/authorized_keys`.
