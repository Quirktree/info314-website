# Using the SSH Agent to Manage Login Keys
SSH keys can provide a notable improvement over password-based security (with
respect to certain threats), but they are not without inconvenience or risk.
Most notably, if we don't use encryption with strong passphrases to keys on 
disk, an intruder or piece of malware can more easily retrieve our keys and 
use them to access any resources we are permitted to access. Moreover, when we 
encrypt our private keys, we face the potential inconvenience of supplying the 
passphrase each time we run `ssh`.

To ease the burden of using keys in the proper manner, OpenSSH provides a 
helper service, i.e., `ssh-agent`, to help you manage your private keys. Once 
an identity has been added to the `ssh-agent`, the agent can use the private 
key to log into servers without requiring you to enter a password for the 
the server or supply the key's passphrase.

The `ssh-agent` helper is available for both macOS and Windows 10 (as well as 
Linux), but the configuration process varies for each system. By following 
the instructions provided in this guide when you first set up your private 
keys, you'll be able to take advantage of the security improvements offered 
by key-based SSH authentication with OpenSSH on your macOS or Windows 10 device.

## Configuring `ssh-agent` for macOS
1. Configure OpenSSH to work with `ssh-agent` and macOS KeyChain
2. Launch the `ssh-agent` as a background service.
3. Use `ssh-add` to load your keys into the agent.

### Step 1
Edit or create `~/.ssh/config` so that `ssh-agent` and macOS KeyChain are used automatically to manage private keys and passphrases. This configuration is required by the version of `ssh` distributed by Apple since macOS Sierra 10.12.2. 

```
Host *
    AddKeysToAgent yes
    UseKeyChain yes
```

### Step 2
Launch the `ssh-agent` as a background service.
```bash
# The eval $() syntax is used to update the current environment based on the output of the ssh-agent command
$ eval "$(ssh-agent -s)"
```

Add your key to the `ssh-agent` using the `ssh-add` utility. 
### Step 3
```bash
# The -K option is required for some versions of macOS. Omit if it returns an error.
ssh-add -K ~/.ssh/id_ed25519

# Check that your key is loaded by listing current keys in the agent
ssh-add -l
```

## Configuring `ssh-agent` for Windows 10
1. Enable the `ssh-agent` Windows Service (disabled by default).
2. Configure Powershell to launch `ssh-agent` when you open a console.
3. Use `ssh-add` to load your keys into the agent.

### Step 1
Launch Powershell with administrator privileges and set the startup mode for the `ssh-agent` service to `Manual` since it is configured by default as `Disabled`.

```powershell
Set-Service -Name ssh-agent -StartupType Manual
```

### Step 2
Open a normal (non-administrator) Powershell console and edit/create a profile script to automatically launch the `ssh-agent` service. 

In order to do this, we need to enable the ability to run basic scripts on the system and then find the location of the script (automatically assigned by Windows to the `$PROFILE` variable).

```powershell
# Modify Powershell policy to allow basic powershell scripts to run
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Test whether the profile script exists
Test-Path $PROFILE

# If the previous command returns False, create the file
New-Item -path $PROFILE -type file -force

# Launch Notepad.exe to create a script
notepad.exe $PROFILE
```

Use Notepad to add the following code to your profile script. This will be run each time you launch a new console.
```powershell
# Launch the service if it's not already running
$agentService = Get-Service -Name ssh-agent
if ($agentService.Status -ne "Running") {
  Start-Service -Name ssh-agent
}
```

### Step 3
Open a fresh Powershell console and add your keys to the agent.
```powershell
# Add the private key to the ssh-agent
ssh-add $HOME\.ssh\id_ed25519

# Check that your key is loaded by listing current keys in the agent
ssh-add -l
```