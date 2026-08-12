# How to Bypass Gigahub for DHCP (GPON Only)

***It is recommended that if you are on Bell Aliant you should wait until XGSPON brought online (Upgrades currently expected to be complete Summer 2026)***

**Purchase SFP Module Alcatel Lucent G-010S-P**
> [AliExpress](https://www.aliexpress.com/item/1005007300051703.html)

**Media Converter (Not required but I find it useful during configuration)**
> [Amazon Canada](https://a.co/d/08YZrmC7)

**Retrieve ONT Serial Number from Back of Gigahub. Begins with “SMB”**

**Login to Gigahub interface > Advanced Tools & Settings > WAN**

*Verify Mode GPON (If it says XG-PON these instructions will not work)*
<img src="https://i.imgur.com/DIXJxvk.png">

**Determine if your Giga Hub is a PPTP or VEIP installation. Perform the following on a computer connected on your LAN while the Giga Hub is still connected.**

1. Install Python3 <a href="https://www.geeksforgeeks.org/python/download-and-install-python-3-latest-version/">Instructions</a>
2. Run the following in a command prompt / terminal window
`python3 -m venv .venv source .venv/bin/activate pip3 install https://github.com/up-n-atom/sagemcom-modem-scripts/releases/download/v0.0.4/xmo_remote_client-0.0.4-py3-none-any.whl`
4. Then run the following (use your Gigahubs Serial in place of the __________
`xmo-remote-client --password=DM_________ get-onu-mode`
5. Note whether the message says PPTP or VEIP

**Download / Extract Firmware**
> https://mega.nz/file/lckmXYrA#tvu0ZuUYufVdpO_Ewi1foxz9vhaLSnx-e4ebpN7p_LQ
> - Duplicate the following file `alcatel-g010sp_new_busybox_theme-squashfs.image`
> - Rename `alcatel-g010sp_new_busybox_theme-squashfs.image` to `firmware.img`

**Secure Copy Firmware to SFP**

*From Linux / Mac open Terminal*

`scp -O -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-dss firmware.img ONTUSER@192.168.1.10:/tmp/firmware.img` 

*Password* `SUGAR2A041`

**Configure SFP over SSH**

`ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -oKexAlgorithms=+diffie-hellman-group14-sha1 ONTUSER@192.168.1.10`

**Verify image currently being used**

`fw_printenv committed_image`

> *If on Zero, image to 1 (Enter these commands)*
>
>`mtd -e image1 write /tmp/firmware.img image1`
>
>`fw_setenv image1_version firmware` (if you have internet/tv/phone replace firmware with 3FE56853BOPD09)
>
>`fw_setenv image1_is_valid 1`
> 
>`fw_setenv target oem-generic`
>
>`fw_setenv committed_image 1`
>
>`reboot`
         
> *If on One, image to 0 (Enter these commands)*
> 
>`mtd -e image0 write /tmp/firmware.img image0`
> 
>`fw_setenv image0_version firmware` (if you have internet/tv/phone replace firmware with 3FE56853BOPD09)
> 
>`fw_setenv image0_is_valid 1`
> 
>`fw_setenv target oem-generic`
> 
>`fw_setenv committed_image 0`
>
>`reboot`

After imaging and doing a reboot go back and image the opposite

**Switch to Custom mibs file**

If PPTP
`uci set gpon.onu.mib_file='/etc/mibs/data_1g_8q_us1280_ds512.ini'`

If VEIP
`uci set gpon.onu.mib_file='/etc/mibs/data_1v_8q.ini'`


`uci commit`

`reboot`

**If you have TV/Phone through you Gigahub**
If your internet service also includes TV and/or phone, run the following on the SFP while connected via SSH:
`omci_pipe.sh md | grep -E '^\| +(130) \|`

If there are two services listed as 130 | 4354 and 130 | 4355, run the following:
`omci_pipe.sh med 130 4355`

An errorcode of 0 confirms that the 130 | 4355 service has been terminated. NOTE: This previous command (to terminate 130 | 4355) will need to be re-run every time the SFP is rebooted.

**Access Web Interface (192.168.1.10) [Firefox/Safari may cause problems]**

Login *root/SUGAR2A041*

**Expand the Cog**
<img src="https://i.imgur.com/n8Zv3xL.png">

**Select First Menu Item**
<img src="https://i.imgur.com/PG3ySHQ.png">

**Select the 3rd tab and change your language to English     - Scroll down to save** (you will have to do the apply twice for it to take effect)
<img src="https://i.imgur.com/AnKdgug.png">

**Select the GPON Tab and add your SMB serial number**
<img src="https://i.imgur.com/FADAdnl.png">

**Scroll down and ensure this box it checked for custom mibs**

*Scroll down to save*
<img src="https://i.imgur.com/DLhBaNE.png">

**Click the unsaved/changes link**

*Click Save/Apply*
<img src="https://i.imgur.com/2DuWsMw.png">

**Remove SFP and Place in Router**

*Configure Router with VLAN 35 - Note: Use 34 if you have phone/tv/internet service*
<img src="https://i.imgur.com/xl34tRv.png">

---

> ***Note After comitting to a new image your SSH key and Login will change you can access the SFP to do further configuration with the following (you may have to delete your stored key from your known keys file)***
> 
> `ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -oKexAlgorithms=+diffie-hellman-group14-sha1 root@192.168.1.10`


---

Thank you to the 8311 Discord Community for helping sort this out

https://github.com/rajkosto/alcatel_lucent-lantiq_falcon/tree/master
