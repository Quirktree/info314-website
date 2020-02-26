# Getting Started with Free Range Routing (FRR) on Raspbian

## Initial Setup

1. Review the project homepage at https://frrouting.org
2. Configure the FRR Debain Repository
    - a. Download the public key used to verify application packages: `curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -`
    - b. Add the repository into the list of available resources for apt: `echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | sudo tee -a /etc/apt/sources.list.d/frr.list`
3. Use `apt` to install FRR: `sudo apt update && sudo apt install frr`
4. Give the Pi user permission to run the FRR managment client
    - a. Run `sudo usermod -aG frrvty pi` to add the `pi` user to the `frrvty` management group
    - b. Logout (exit SSH) and return in order for Linux to recognize your new privileges
5. Enable the BGP routing daemon
    - a. Edit `/etc/frr/daemons` and set `bgpd=yes`. 
    - b. Repeat this process for other routing daemons (if applicable).
6. Restart FRRouting: `sudo systemctl restart frr`

## Learn the basics of navigating VTY Shell Basics

### Launch VTY shell
To manage FRR, we will use a dedicated UI called the VTY shell. Launch the shell with the `vtysh` command.

### VTYSH modes
While using the VTY shell, you'll need to switch between _modes_. While most of our work will be conducted in _Configuration Mode_, the shell will launch initially in _Enable mode_. To enter Configuration Mode, type `configure terminal` at the VTY shell prompt. To exit Configuration Mode and return to Enable Mode, type `exit`.

### Save Configuration (from enable mode)
Configuration changes you make from VTY shell will take effect right away, but they do not persist between reboots due to a distinction FRR makes between the _Running Config_ and the _Startup Config_. To save your changes to disk, return to Enable Mode and type `write memory` or `copy running-config startup-config`.

### Show the Running Config (from enable mode)
To view the active Running Config, return to Enable Mode and enter `show running-config` or `write terminal` at the prompt.

### Shortcuts
Command abbreviations are accepted in VTY shell as long as the abbreviation is unambiguous. The shell will do its best to determine your intent and complete the command.

- `configure terminal` can be abbreviated `conf t`
- `quit` can be abbreviated `q`
- `write memory` can be abbreviated `w m` 
