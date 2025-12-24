# my-settings-for-rtw88_8821au-on-debian-13

# Fixing my woes with TP-Link Archer T2U plus on Linux (rtw88_8821au) - SSH from My Windows laptop to Debian host was all janky and jittery. 
# And a quick diagnosis showed me the wifi network latency from the debian host was shooting very high intermittently due to the current driver
# 8821au simply being just inefficient.

Update Debian 13's default 6.12 kernel to the latest one available in trixie backports - 6.17 at the time of writing.

This is because starting Linux kernel 6.14, there's an updated in-kernel driver for the dreaded 8821au type Realtek devices.


The kernel update was not done using standard custom kernel compilation steps but using Debians's own suggested methods of using backports.
STEPS -

# Add backports to sources if not already present-
echo "deb http://deb.debian.org/debian trixie-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
sudo apt update


# Check and install the latest tested kernel available in backports-

sudo apt -t trixie-backports search linux-image-amd64

sudo apt -t trixie-backports install linux-image-amd64


# Cleaned/removed the old kernel leftovers using -

sudo apt autoremove

# You should see the superior (and arguably better) in-kernel driver (rtw88_8821au). In default kernel 6.12.x this driver was 8821au which has issues with aggressive power saving of this USB adaptor chipset.

lsmod | grep -i rtw

rtw88_8821au           12288  0

rtw88_8821a            45056  1 rtw88_8821au

rtw88_88xxa            45056  1 rtw88_8821a

rtw88_usb              32768  1 rtw88_8821au

rtw88_core            282624  3 rtw88_88xxa,rtw88_8821a,rtw88_usb

mac80211             1662976  2 rtw88_core,rtw88_usb

cfg80211             1495040  2 rtw88_core,mac80211

usbcore               425984  6 xhci_hcd,usbhid,rtw88_usb,rtw88_8821au,xhci_pci,hid_logitech_hidpp


# Now the most important part which fixed my issue -
# Create the modprobe file if it doesn't already exist in /etc/modprobe.d-
touch /etc/modprobe/rtw88.conf

And add these below options-

options rtw88_core disable_lps_sleep=Y

options rtw88_core disable_aspm=Y

options rtw88_usb switch_usb_mode=Y

options rtw88_pci disable_aspm=Y disable_msi=Y


# Save the file, exit and reboot the machine. You will immidiately notice the ssh issue gone. At least I did.
