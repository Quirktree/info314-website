## Install TShark
To perform captures from your pi you will need to install `tshark` (command line version of wireshark) using the `apt` command. When prompted to allow non-root users to run packet captures, say **yes**.

After install, run `sudo usermod -aG wireshark pi` and **reboot**. This command is adding the `pi` user to the `wireshark` group. This group was created when you installed `tshark` and it is used to allow non-root users to run packet captures.

After rebooting, test `tshark` by running `tshark -i wlan0` and confirming that packets are captured. Type ++ctrl+c++ to quit capturing.

!!! info
    Use the command `man tshark` from the pi for more info about tshark and how to use it.

## Performing a Capture
Tshark is a terminal based tool, but that doesn't mean you are limited to conducting all of your analysis on the commandline. Follow along below to explore two different approaches to get your tshark captures into wireshark for analysis.

### Capturing to a file
The first approach we present is the easiest, but requires some manual effort to complete the capture and copy back to your computer with ssh/scp. Once the file is saved to your computer, you can open it in Wireshark and perform analysis just as you would with a native Wireshark capture.

`tshark` allows you to specify the interface on which to capture using the `-i` argument as well as a filename to record the capture using `-w`. You may also pass a capture filter as the final argument of the  command.

```
# Capture DNS traffic from wlan0 and write the output into a file named cap.pcapng
tshark -i wlan0 -w cap.pcapng 'port 53'
```

### Live Capture to Wireshark
With a little more effort, Wireshark can leverage tshark for a remote capture. In this case, we are launching a capture with tshark and piping the results back through the SSH session into Wireshark (instead of saving to a file on the Pi). There is an ulterior motive for sharing this method. For some students, the live capture does provide a more natural workflow. However, this is a good opportunity to give you a small taste of what SSH can accomplish[^beast-mode].

[^beast-mode]: Live Wireshark captures are cool, but we've hardly scratched the surface of secure shell's super powers.

!!! important "Windows users"
    Add the wireshark program directory (`c:\program files\wireshark\`) to the system path and then use the standard command prompt rather than Powershell to launch the remote capture. 

The following command will launch a remote capture on your Pi (see additional notes for Windows below): 

```
ssh pi@titan.local tshark -s0 -F libpcap -i wlan0 -w - 'port 53' | wireshark -k -i -
```