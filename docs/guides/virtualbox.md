# Convert VirtualBOX VDI to Hyper-V VHD

Conversion is very simple although it requires the instalation of full VirtualBox. To make the conversion you need to use VirtualBoxManage Tool. VirtualBoxManage is not available by itself as of this writing

#Install Virtual Box Here: https://www.virtualbox.org/wiki/Downloads
#If you are on Windows VirtualBoxManage Tool will be located in C:\ProgramFiles\Oracle\VirtualBox. 
#* It is easiest to add this directory to your system path as it makes the conversion command easier as your source and destination paths can be relative to the directory you call virtualboxmanage from
#Enter Command: vboxmanage clonehd <source .vdi file> <destination .vhd file> --format VHD
#* example: vboxmanage clonehd WinXP.vdi F:\winxp.vhd --format VHD
#Wait for it to complete. For about 10 gigs it will take about 2 minutes. 100gigs can take roughly 30min

==Notes==

==Sources==
http://www.sysprobs.com/vdi-vhd-convert-virtualbox-virtual-machines-virtual-pc
