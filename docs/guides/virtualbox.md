# Convert VirtualBOX VDI to Hyper-V VHD

Conversion is very simple although it requires the instalation of full VirtualBox. To make the conversion you need to use VirtualBoxManage Tool. VirtualBoxManage is not available by itself as of this writing

1. Install Virtual Box Here: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
2. If you are on Windows VirtualBoxManage Tool will be located in `C:\ProgramFiles\Oracle\VirtualBox` 
    * It is easiest to add this directory to your system path as it makes the conversion command easier as your source and destination paths can be relative to the directory you call virtualboxmanage from
3. Enter Command: 
    ```bash
    vboxmanage clonehd <source .vdi file> <destination .vhd file> --format VHD
    ```
    **Example:**
    ```bash
    vboxmanage clonehd WinXP.vdi F:\winxp.vhd --format VHD
    ```
4. Wait for it to complete. For about 10 gigs it will take about 2 minutes. 100gigs can take roughly 30min



## Resources
* [http://www.sysprobs.com/vdi-vhd-convert-virtualbox-virtual-machines-virtual-pc](http://www.sysprobs.com/vdi-vhd-convert-virtualbox-virtual-machines-virtual-pc)
