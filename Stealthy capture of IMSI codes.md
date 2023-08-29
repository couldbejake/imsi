Title: The stealthy capture of an IMSI code from a mobile phone.
Date: Aug 3, 2020
Author: couldbejake

Every SIM has an IMSI (International Mobile Subscriber Identity) - not to be confused with the IMEI, an IMSI is a 15 digit string of numbers. It can be use to identify a single SIM, and therefore in most situations a person. The IMSI is connected in the Mobile Provider's databases to your banking information, home address and more. A given IMSI for a SIM cannot change and stays the same throughout it's lifetime. 

Everyone in a nearby vicinity connects to a mobile tower, authenticating using their IMSI. If you could just collect those special 15 digits, you'd be a god.

Well that's what some people at defcon have tried to do. In Chris Paget's talk of 2013 he setup a base-station which captured the IMSI by tricking the mobile phone into believing it was a legitamate cell phone tower.

Although this is a neat trick, it is slightly... VERY illegal as it requires broadcasting on restricted frequencies.

Capturing an IMSI during Probe or Connection to a Network

Method 1: WiFi Calling exploit

From the methods shown at Defcon, both methods require connecting to the WiFi network. The first method demonstrates a past flaw with WiFi calling which sends the initial auth data over plaintext. After that a pseudo identifier is used. After the talk came out in 2016, most mobiles switched over to transporting this information through TLS, requiring decryption. Making this method virtually impossible. Additionally while testing I found that my mobile phone decided not to use WiFi calling when it had at least 1 bar of connectivity, further research would be needed.

Method 2: EAP Authentication

EAP Authentication (Extensible Authentication Protocol), is just another method of authenticating with a WiFi network, similar to typing a password. When authenticating a ‘Request, Identity’ packet is sent over EAP. Originally this was used by corporate companies to log in using a username and password. EAP-AKA / EAP-SIM is a subset of EAP authentication that passes the IMS as the identity. 

(IMSI during Probe?) I assumed the IMSI would be sent out as a probe request. I set up airodump-ng to capture packets being sent to my home ap. While there were many packets being sent during authentication, none matched EAP. Using airodump-ng I found that up to half of the frames captured during authentication were lost; For this reason I attempted a capture a pcap of 20 attempts (automated using WiFi deauthentication), but still did not find an EAP packet. Additionally, after some research, I found that when a device tries to authenticate it goes through a routine of channel switching. To capture all 12 channels, 12 seperate WiFi adapters would be required, even then, some frames might be lost. My problem here was that I was looking for an EAP authentication protocol while the network wasn’t using EAP.
The talk at DefCon was slightly mis-represented. Sure you could capture an IMSI in plaintext, but the network has to be using EAP with it’s method set to SIM. 

(IMSI via Authentication) After realizing the IMSI can only be captured during authentication, I set up a library called EAPHammer which has no native support for EAP-SIM but base EAP. I was able to setup an evil twin hotspot which required a username / password. The library logs the output to the screen. Note: This attack requires an SSL certificate which can be self signed. The library has no native support for EAP-SIM but makes use of ‘hostapd’.
When connecting to a WiFi network that uses EAP, the device often ignores suggested EAP methods, maybe as a security feature. This means that the user has to manually click on EAP-SIM in the connection panel. This may be a problem as it defeats the purpose of this project; stealthy capturing a device’s IMSI without the user needing direct instructions. However I was able to capture a users IMSI number from their phone if they connect to my network and connect via SIM.

[ SOURCES ]

Wi-Fi Roaming Analysis with Wireshark and AirPcap — Revolution Wi-Fi

IMSI Catching over WIFI Networks | LinkedIn

airodump-ng [Aircrack-ng]

CaptureSetup/WLAN - The Wireshark Wiki

Tracking People & Devices with WiFi

How to Log Wi-Fi Probe Requests from Smartphones & Laptops with Probemon « Null Byte Log Wi-Fi Probe Requests from Smartphones & Laptops with Probemon

TLS Certificates from EAP Network Traffic - Black Hills Information Security

WiFi Pineapple Wiki

The web interface – Hak5

Which EAP types do you need for which identity projects? | Network World

s0lst1c3/eaphammer: Targeted evil twin attacks against WPA2-Enterprise networks. Indirect wireless pivots using hostile portal attacks.

Setup Guide · s0lst1c3/eaphammer Wiki

II. Stealing RADIUS Credentials Using EAPHammer · s0lst1c3/eaphammer Wiki

Home · s0lst1c3/eaphammer Wiki

How RADIUS Server Authentication Works

EAPHammer Version 1.8.0 - EAP downgrade attacks · solstice.sh

EAP-PEAP with Mschapv2: Decrypted and D... - Cisco Community

wpa supplicant - Test EAP-SIM with hostapd and wpa_supplicant - Stack Overflow

sqlite - fatal error: sqlite3.h: No such file or directory - Stack Overflow

hostapd: compiling instructions · IntelOpenDesign/MakerNode Wiki

EbMajor - Nocturne: hostapd/wpa_supplicant hwsim test scripts

hostapd / wpa_supplicant - GIT access

hostapd/hlr_auc_gw.txt - platform/external/wpa_supplicant_8 - Git at Google

https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf

wireless - hostapd compile error - Ask Ubuntu

hostap/hostapd.radius_clients at master · wertarbyte/hostap

How To Setup A Wireless Access Point On Linux OS using hostapd - Shellvoide

https://w1.fi/cgit/hostap/plain/hostapd/hostapd.eap_user

https://android.googlesource.com/platform/external/wpa_supplicant_8/+/refs/heads/master/hostapd/hlr_auc_gw.c

Number Lookup API: Ensure Mobile Number Validity | Infobip


