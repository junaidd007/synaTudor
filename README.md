Synaptics Tudor Driver Relinking Project
This project attempts to dynamically relink the driver of the Synaptics Tudor family of fingerprint sensors at run time, allowing them to run and provide their functionality on Linux-based x86-64 systems. It split off from the reverse engineering branch after it hit multiple dead ends, and because it showed the potential of quickly allowing for the creation of something which can at least allow users to use the sensors installed in their hardware.

THIS PROJECT IS PROVIDED AS IS, WITHOUT WARRANTY OR LIABILITY OF ANY KIND, OR FOR ANY RISKS OR SIDE EFFECTS WHICH MIGHT OCCUR FROM USAGE OF ANYTHING PROVIDED AS PART OF THIS PROJECT, INCLUDING, BUT NOT LIMITED TO, BRICKED SENSORS, CORRUPTED FIRMWARE, BYPASSES OF HOST SECURITY, AND VULNERABILITIES IN THE CODE. USE AT YOUR OWN RISK.

NOTE: The project should be fully functional right now, contrary to its earlier state. If there are any issues, please report them.

Updates and Changes Made
The following changes have been made to the original project to include support for the '06cb:00be' fingerprint sensor:

Forked the original project and made necessary changes to include support for the new device.
Updated the README to include instructions for installing the modified driver and configuring the PAM module to support fingerprint authentication as a fallback method.
Installation
To install the modified driver, follow these steps:

Install the required dependencies:
sh
Copy code
sudo apt install meson cmake pkg-config libcrypto++-dev libusb-1.0-0-dev libcap-dev libseccomp-dev libglib2.0-dev libdbus-1-dev libfprint-2-dev libfprint-2-tod-dev libjson-glib-dev innoextract libssl-dev
Clone the modified repository:
sh
Copy code
git clone https://github.com/Popax21/synaTudor
Follow the same installation instructions as mentioned in the original README.
Usage
To configure PAM to support fingerprint authentication as a fallback method, follow these steps:

Find out which file contains fprintd:
sh
Copy code
grep -r "fprintd" /etc/pam.d
Identify the PAM configuration file managing fingerprints (e.g., common-auth).

Create a backup of the configuration file:

sh
Copy code
cp /etc/pam.d/common-auth /etc/pam.d/common-auth.bk
Edit the configuration file to ask for a password first and then fingerprint if the password is incorrect. Replace the original common-auth content with the modified content provided in the common-auth modified section below.
common-auth modified
Here is the modified common-auth content:

#
# /etc/pam.d/common-auth - authentication settings common to all services
#
# This file is included from other service-specific PAM config files,
# and should contain a list of the authentication modules that define
# the central authentication scheme for use on the system
# (e.g., /etc/shadow, LDAP, Kerberos, etc.).  The default is to use the
# traditional Unix authentication mechanisms.
#
# As of pam 1.0.1-6, this file is managed by pam-auth-update by default.
# To take advantage of this, it is recommended that you configure any
# local modules either before or after the default block, and use
# pam-auth-update to manage selection of other modules.  See
# pam-auth-update(8) for details.

# here are the per-package modules (the "Primary" block)
auth    [success=done default=ignore]      pam_unix.so nullok try_first_pass
auth    [success=done default=ignore]      pam_sss.so use_first_pass
auth    [success=1 default=ignore]      pam_fprintd.so max-tries=1 timeout=10  debug
# here's the fallback if no module succeeds
auth	requisite			pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
auth	required			pam_permit.so
# and here are more per-package modules (the "Additional" block)
auth	optional			pam_cap.so 
# end of pam-auth-update config


modify this later using chat gpt
'
then i was having issues with fingerprint service as it failed to reognize fingerprint for consequitive attempts so i used command 'journalctl -fu fprintd' to see its behaviour.

output from journalctl -fu fprintd.


'
the whole fingerprint process starts and then deactivates if it used for some time and this is the whole process of that.

when i use sudo -i in a new terminal, the  'Starting Fingerprint Authentication Daemon.' happens and i can only scan fingerprint after a few seconds after 'Started Fingerprint Authentication Daemon.' happens. but if i dont call for any fingerprint process, it 'fprintd.service: Deactivated successfully.' deactivates.
Apr 22 13:13:14 jd-flex systemd[1]: Starting Fingerprint Authentication Daemon...
Apr 22 13:13:14 jd-flex fprintd[21803]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:13:14 jd-flex fprintd[21803]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:13:14 jd-flex fprintd[21803]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:13:14 jd-flex fprintd[21803]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:13:17 jd-flex systemd[1]: Started Fingerprint Authentication Daemon.
Apr 22 13:15:14 jd-flex fprintd[21803]: Tudor host process hit shut down timeout!
Apr 22 13:15:14 jd-flex systemd[1]: fprintd.service: Deactivated successfully.


Apr 22 13:06:40 jd-flex systemd[1]: Starting Fingerprint Authentication Daemon...
Apr 22 13:06:41 jd-flex fprintd[21495]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:06:41 jd-flex fprintd[21495]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:06:41 jd-flex fprintd[21495]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:06:41 jd-flex fprintd[21495]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:06:44 jd-flex systemd[1]: Started Fingerprint Authentication Daemon.

used wrong fingerprint here a bunch of times and it kills the process and on subsequent fingerprint prompts, it always fails to match fingerprint as it gives error 'Device reported an error during identify: Tudor host process died' and even if i fail the three password matches it and run sudo -i again, it still fails until i run  'sudo service fprintd restart'. after running this it scans fingerprint fine
Apr 22 13:07:45 jd-flex fprintd[21495]: Tudor host process died! Exit Code 134
Apr 22 13:07:51 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:07:54 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:07:56 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died

ran 'sudo -i' here

Apr 22 13:08:04 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:06 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:08 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:17 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:22 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:23 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died
Apr 22 13:08:37 jd-flex fprintd[21495]: Device reported an error during identify: Tudor host process died

ran 'sudo service fprintd restart' here

Apr 22 13:08:39 jd-flex systemd[1]: Stopping Fingerprint Authentication Daemon...
Apr 22 13:08:39 jd-flex fprintd[21495]: libusb: warning [libusb_exit] device 3.3 still referenced
Apr 22 13:08:39 jd-flex fprintd[21495]: libusb: warning [libusb_exit] device 3.1 still referenced
Apr 22 13:08:39 jd-flex systemd[1]: fprintd.service: Deactivated successfully.
Apr 22 13:08:39 jd-flex systemd[1]: Stopped Fingerprint Authentication Daemon.
Apr 22 13:08:39 jd-flex systemd[1]: Starting Fingerprint Authentication Daemon...
Apr 22 13:08:39 jd-flex fprintd[21716]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:08:39 jd-flex fprintd[21716]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:08:39 jd-flex fprintd[21716]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:08:39 jd-flex fprintd[21716]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 13:08:42 jd-flex systemd[1]: Started Fingerprint Authentication Daemon.
Apr 22 13:09:15 jd-flex fprintd[21716]: Tudor host process died! Exit Code 134
Apr 22 13:09:15 jd-flex systemd[1]: fprintd.service: Deactivated successfully.


'

'
Apr 22 14:43:04 jd-flex systemd[1]: Starting Fingerprint Authentication Daemon...
Apr 22 14:43:04 jd-flex fprintd[25881]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 14:43:04 jd-flex fprintd[25881]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 14:43:04 jd-flex fprintd[25881]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 14:43:04 jd-flex fprintd[25881]: libusb: error [udev_hotplug_event] ignoring udev action change
Apr 22 14:43:07 jd-flex systemd[1]: Started Fingerprint Authentication Daemon.
Apr 22 14:44:29 jd-flex fprintd[25881]: Tudor host process died! Exit Code 134
Apr 22 14:44:37 jd-flex fprintd[25881]: Device reported an error during identify for enroll: Tudor host process died

'


'
pr 22 14:08:03 jd-flex fprintd[25141]: Authorization denied to :1.1208 to call method 'Claim' for device 'Synaptics Tudor': Device was already claimed
Apr 22 14:08:05 jd-flex fprintd[25141]: Authorization denied to :1.1208 to call method 'Claim' for device 'Synaptics Tudor': Device was already claimed
Apr 22 14:34:33 jd-flex fprintd[25141]: Authorization denied to :1.1218 to call method 'Claim' for device 'Synaptics Tudor': Device was already claimed
Apr 22 14:34:34 jd-flex fprintd[25141]: Authorization denied to :1.1219 to call method 'Claim' for device 'Synaptics Tudor': Device was already claimed
Apr 22 14:34:46 jd-flex fprintd[25141]: Authorization denied to :1.1208 to call method 'Claim' for device 'Synaptics Tudor': Device was already claimed

'



so i created this script to fix the problem


/usr/local/bin/fingerPrint/restart-fprintd.sh

'
#!/bin/bash

# Define patterns to monitor
pattern1="Device reported an error during identify:"
pattern2="Device reported an error during identify for enroll:"
pattern3="to call method 'Claim' for device 'Synaptics Tudor'"

# Function to restart fprintd if one of the patterns is found
check_and_restart_fprintd() {
    if echo "$1" | grep -qE "$pattern1|$pattern2|$pattern3"; then
        echo "Restarting fprintd due to an error"
        systemctl restart fprintd
    fi
}

# Continuously monitor journalctl output
journalctl -fu fprintd | while read -r line; do
    echo "$line"
    check_and_restart_fprintd "$line"
done

'

To make the script always running, you can create a systemd service that will manage the script execution.

Save the script to a file, for example, /usr/local/bin/restart-fprintd.sh. Ensure it's executable with the following command:

bash
Copy code
sudo chmod +x /usr/local/bin/restart-fprintd.sh
Create a systemd service file by running:

bash
Copy code
sudo nano /etc/systemd/system/restart-fprintd-monitor.service
Add the following content to the service file:

makefile
Copy code
[Unit]
Description=Restart fprintd when the Tudor host process dies
After=network.target

[Service]
ExecStart=/usr/local/bin/restart-fprintd.sh
Restart=always
RestartSec=5
User=root
Group=root

[Install]
WantedBy=multi-user.target
Save and exit the file.

Reload the systemd configuration:

Copy code
sudo systemctl daemon-reload
Enable and start the service:

bash
Copy code
sudo systemctl enable --now restart-fprintd-monitor.service
Now the script will run continuously, managed by systemd. It will restart the fprintd service whenever the specified error is detected in the logs.
'

