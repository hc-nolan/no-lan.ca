+++
date = '2023-06-19T01:24:13-04:00'
draft = false
title = 'Setting up Proxmox on a Dell PowerEdge T130'
+++

This is a blog post about my experience getting Proxmox working on a Dell T130 I found on Kijiji for $100.

<!--more-->
---

## Introduction

The other day I picked up a Dell Poweredge T130 for $110.

When I bought it, there was a single 500GB HDD and it was running Windows Server. I bought three 8TB HDDs, and a 500GB M.2 NVMe SSD for the operating system, for which I used a PCIe adapter to install in the server.

I plan to move all my data over, then shuck the 14TB external drive I'm currently using and replace the 500GB drive with it. I also want to put a storage SSD in the optical drive. I don't really need it since I've got plenty of space on the boot drive, but maybe I'll use it for something that benefits from faster storage access.

## Booting

The NVMe drive was recognized immediately, and I flashed [Proxmox](https://www.proxmox.com/en/) to it, but it wouldn't boot.

I did some research and realized that this generation of Poweredge servers doesn't support booting from NVMe drives natively - but I knew that it could somehow, because I got the idea from [someone else](https://old.reddit.com/r/homelab/comments/fwepd6/plex_questions_poweredge_t130_proxmox/) who had done it successfully.

It turns out that there is a project called [Clover Bootloader](https://github.com/CloverHackyColor/CloverBootloader). You flash it to a USB, and it is able to boot the NVMe drive. So I flashed it to the USB, plugged it in, and booted up the server... aaand no USB recognized. I tried the other USB ports, but still nothing.

I then decided to flash Proxmox to the hard drive that came with the device, since I knew it was able to boot Windows. However, that too wouldn't boot, and to make matters slightly worse, I had formatted the drive by installing Proxmox, so I now no longer had the Windows installation to fall back on in the absolute worst case scenario. (I didn't have the password and I don't know if the seller had wiped the drive so I don't know if he would have given it to me anyway, but it still felt like a safety net) I even tried flashing Windows Sever to it again, but still nothing would boot.

## BIOS

Eventually I realized that the BIOS had never been updated, it was still v1.0.1. I tried to use [Dell Repository Manager](https://www.dell.com/support/kbdoc/en-ca/000177083/support-for-dell-emc-repository-manager-drm) to create a bootable USB that would handle all the updates, but I couldn't get it to run on my laptop and didn't care to try to troubleshoot that. Instead I found the archive of all the BIOS versions, and a [forum post](https://www.dell.com/community/en/conversations/systems-management-general/dell-t330-bios-and-idrac-upgrade-path/647f8e6af4ccf8a8def4ae51#M30182) from a user in a similar situation, where a Dell employee suggested that updating from such an old version directly to one of the latest versions could cause issues, so I ended up having to update the BIOS in 7 stages. It took forever and was quite annoying.

With an updated BIOS, Proxmox successfully booted off the hard drive. However, the USB with Clover still wasn't being recognized, leaving the NVMe drive unbootable still. After hours of pulling my hair out, I finally figured out the issue: the USB drive. I had been using various drives during the process of updating the BIOS and attempting to flash Proxmox and Windows Server before that. I don't know specifically why, but some of the drives I was using (which were the same brand/model) simply don't work in the server. They are pretty old so it must be some sort of compatibility issue. After flashing Clover to one of the other USBs which I knew the server recognized, I finally was able to boot into Clover with the help of a highly informative [Reddit post](https://old.reddit.com/r/homelab/comments/tcp2rz/dell_poweredge_r730_boot_from_pcie_m2_device/) detailing all the necessary steps, and in turn boot into Proxmox on the NVMe drive.

## Misc

There were a couple other minor hiccups. When I got the server, it had 16GB RAM. It took a while to track down compatible RAM, but I managed to find some and ordered 16GB as a test. They ended up sending me the wrong sticks and their support annoyed me so I found another vendor. I'm still waiting for it to come; hopefully it works.

There was also no iDRAC. For those unaware, iDRAC is both a physical card and piece of software that allows you to manage your server remotely as if you had physical access to it. While I had physical access to my machine, Dell has included things in iDRAC that make operations like updating the BIOS or flashing a new OS less tedious than doing it with a USB drive.

Another thing that iDRAC would've been helpful for is controlling the fan speed. When you install any PCIe adapter into the T130, the fans shoot up to max speed and it sounds like a jet engine. At first I thought this was just how loud the server was, and was thinking I made a horrible mistake. In the end, I was able to use [ipmitool](https://github.com/ipmitool/ipmitool) to adjust the fan speed, with some black magic wizardry in the form of raw hex commands which are undocumented and exist in the form of a [forum post](https://www.dell.com/community/en/conversations/poweredge-hardware-general/dell-poweredge-t130-fan-issue/647f639af4ccf8a8defd6919) from an anonymous user. Except the commands are different when sending remotely, which another user [pointed out](https://www.dell.com/community/en/conversations/poweredge-hardware-general/dell-poweredge-t130-fan-issue/647f639af4ccf8a8defd6919?commentId=647f6d1ff4ccf8a8dea6206f#M47465).

They have iDRAC cards on [usedservers.ca](https://usedservers.ca/) for a very reasonable price, so I planned to buy one, but they note that you have to buy the license separately. It took a while to figure out how to even purchase an individual license, but when I finally did, I realized it would cost €‎400, which is more than I am willing to spend on the software. So I ended up buying a card and license from a random Chinese eBay vendor. Which isn't ideal, but oh well.

I had originally planned to virtualize [OPNsense](https://opnsense.org/) on the server, but I found it to be kind of a hassle, and I also realized that it wasn't a great idea to have my network backbone inside a server which I may sometimes want to power down but still have Internet on other devices. I decided to make another addition to my homelab in the form of a dedicated OPNsense device. I found this really cool company, [Protectli](https://protectli.com/), and bought their FW4B: 4x 1G ports (my heart wanted 2.5G, but this $110 server had already ended up being more than $1000 in total), Intel J3160, 8GB DDR3, 480GB mSATA SSD. I'm super excited, and going to make another post about it when it comes later this week.

I would also like to say thanks to the people over at [infosec.exchange](https://infosec.exchange) who offered some suggestions as I struggled in real-time.

