
# Installing WireShark
Install WiresShark using the instructions linked below and test that you are able to run a packet capture over your current wireless connection.

## Instructions (macOS)

Download the macOS installer from the [WireShark download page](https://www.wireshark.org/download.html) and proceed with the instructions at: https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallOSXInstall.html

!!! Important
    The latest release of WireShark shipped with a macOS specific bug that appears with a permission denied error every time the user opens a new terminal window. This is a known bug[^bug] in the latest installer and can be resolved by running `sudo chmod 644 /etc/manpaths.d/Wireshark /etc/paths.d/Wireshark` from the terminal.

[^bug]: https://github.com/Homebrew/homebrew-cask/issues/74548

## Instructions (Windows)
Download the Windows installer (64-bit) from the [Wireshark download page](https://www.wireshark.org/download.html) and proceed with the instructions at: https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallWinInstall.html

## Instructions (Linux)

https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallUnixInstallBins.html
