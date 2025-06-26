#Installation

These are instructions I originally found [here](https://www.dslreports.com/forum/r30661088-PBX-FreePBX-for-the-Raspberry-Pi) in early 2025, with some changes to make it easier to understand. It may not remain up-to-date but worked in June 2025!

The included script (install) and archive (install.tar.gz) will build FreePBX 15, 16, or 17 plus Asterisk 18, 20, 21, or 22 on a Raspberry Pi.
iptables, dnsmasq, and exim4 are also installed.

## Install steps

Installation takes approximately 15 minutes to complete on a Raspberry Pi 5.

1. Download the latest Raspberry Pi OS Lite image (32-bit/64-bit Bullseye/Bookworm).

1. Write the image to an 8 GB or larger SD card. To accomplish this, I recommend Etcher or imageUSB: https://etcher.io/ or http://osforensics.com/downloads/imageusb.zip

1. Create an empty file named `ssh` in the `/boot/` directory (type NUL > ssh).

1. Create a text file named `userconf` in the `/boot/` directory containing the following:

   `pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0`

   You'll be setting up to users for the system, one regular one with username `pi` and one with `root` which will have admin permssions.

1. Connect the Raspberry Pi to your LAN using an Ethernet cable.

1. Insert the SD card and power up the Raspberry Pi.

1. Now we have to find the IP address of the Pi in order to login via `ssh`. You can use something like https://nmap.org/ - or if you have a Mac you can run `arp -a` in the terminal and get a list of devices connected to your network. The IP addresses will be the first number in round brackets, and will look something like this `123.456.0.7`.

1. In your terminal/console log in using `ssh pi@{ip-address-from-above}`.

1. Copy the files `install` and `install.tar.gz` to the `/home/pi directory`. To accomplish this, I recommend WinSCP: https://winscp.net/eng/download.php

1. Make the install script executable:

   `$ chmod +x install`

1. Run the install script:

   `$ sudo ./install`

1. When prompted:

   - Set pi user password
   - Set root user password
   - Select FreePBX version
   - Select Asterisk version
   - Answer Edge option
   - Answer IPv6 option ('No' recommended)
   - Review selections
   - Set Hostname (Item 2 / N1 - Hostname: FreePBX)
   - Set Localisation Options - Locale (Item 4 / I1)
   - Set Localisation Options - Timezone (Item 4 / I2 - in US, use America, not US)
   - Expand Filesystem (Item 7 / A1)
   - Finish / Reboot Now: No

   The Raspberry Pi will reboot.

1. Log in as `root`.

1. If desired, enable PuTTY logging when prompted.

1. The system will be updated and then reboot.

1. Log in as root.

1. If desired, enable PuTTY logging when prompted.

1. Confirm install.

   Installation will proceed unattended and then reboot.

1. Log in as root.

   Installation will complete.

You should now be able to login into the FreePBX UI from a browser window, by entering the IP address from above.

A number of utility scripts are included in /root.

# GVSIP

To use Google Voice SIP trunks, Asterisk 18, 20, or 21 MUST be used.

Configure FreePBX settings as follows:

Settings -> Advanced Settings -> Dialplan and Operational
SIP Channel Driver = both

Settings -> Asterisk SIP Settings -> General SIP Settings tab -> Media Transport Settings
STUN Server Address = stun.l.google.com:19302

Settings -> Asterisk SIP Settings -> Chan PJSIP Settings tab -> tls
tls - 0.0.0.0 - All = Yes

Settings -> Asterisk SIP Settings -> Chan PJSIP Settings tab -> 0.0.0.0 (udp)
Port to Listen On = 5060

Settings -> Asterisk SIP Settings -> Chan PJSIP Settings tab -> 0.0.0.0 (tls)
Port to Listen On = 5061

Settings -> Asterisk SIP Settings -> Chan SIP Settings tab -> Advanced General Settings
Bind Port = 5160

Settings -> Asterisk SIP Settings -> Chan SIP Settings tab -> Advanced General Settings
TLS Bind Port = 5161

If any changes are necessary, reboot after all changes have been submitted/applied and recheck everything.

Running:
asterisk -rx "module show like pj"
should display around 49 loaded modules with all but around 3 of them displaying a status of "Running".

Install Certificate Manager module (if not already installed).

Run: mv /root/obihai.\* /etc/asterisk/keys/

Run: chown asterisk:asterisk /etc/asterisk/keys/obihai\*

Click: Admin -> Certificate Management -> Import Locally

Settings -> Asterisk SIP Settings -> Chan PJSIP Settings tab -> TLS/SSL/SRTP Settings
Certificate Manager = obihai

Settings -> Asterisk SIP Settings -> Chan PJSIP Settings tab -> TLS/SSL/SRTP Settings
SSL Method = tlsv1_2

Configure gvsip.dat for your Google Voice account(s). If you have more than one Google Voice account, copy
the five [gvsip1] sections to [gvsip2], [gvsip3], etc. Then edit each of the five [gvsipN] groups as follows:

Change (3 places):
NNNNNNNNNN to {10-digit Google Voice number}

Update:
refresh_token={Google Voice Refresh Token}
oauth_clientid={Google Voice Client ID}
oauth_secret={Google Voice Client Secret}
contact_header_params=obn={Google Voice SIP Name}

Upon completion, copy gvsip.dat to /etc/asterisk/pjsip_custom_post.conf:
cp gvsip.dat /etc/asterisk/pjsip_custom_post.conf

For each Google Voice account, create a Custom Trunk as follows:

Connectivity -> Trunks -> Add Trunk -> Add Custom Trunk - General tab
Outbound CallerID = <+{10-digit Google Voice number}+>

Connectivity -> Trunks -> Add Trunk -> Add Custom Trunk - General tab
CID Options = Force Trunk CID

Connectivity -> Trunks -> Add Trunk -> Add Custom Trunk - custom Settings tab
Custom Dial String = PJSIP/+$OUTNUM$@gvsipN (Replace 'gvsipN' with the [gvsipN] group number from gvsip.dat)

Edit /etc/ssl/openssl.cnf and ensure the following section exists with the values shown:

[system_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT@SECLEVEL=1

Upon completion of GVSIP configuration, run: fwconsole restart

# HylaFAX fax server

1. Execute install-fax: ./install-fax

2. Execute add-fax-extension: ./add-fax-extension

Multiple fax exntsions may be added to support simultaneous sending and/or receiving of faxes.

# SendFax

SendFax is a program to send a fax file from Windows to a HylaFAX fax server.
No installation is required and no changes are made to your system.
Supported file tpyes are pdf, ps, tif, and tiff.
A cover page can be generated and prepended to outgoing faxes.
Leaving 'File to Send' empty will send only a cover page.

To configure, click Edit -> Options:

IP Address: (the IP address of your HylaFAX server)
Port Number: (the port number of your HylaFax server, normally 4559)
Username: (your username on your HylaFAX server, normally root)
Password: (your password on your HylaFAX server, normally blank)
Email Address: (the email address to deliver notifications to)
Notifications: (notification types to be sent)
Page Chop: (which pages to chop trailing whitespace from)
Threshold: (minimum trailing whitespace (in.) before chopping is used)
Modem: (which modem to use for outgoing faxes, normally blank)
Cover Folder: (folder to save cover page information in)
