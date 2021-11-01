## Stegnography challange:

giveup.jpg file is found in the pcap file.  
You can use both wireshark or NetworkMiner to extract the file.  

Looking at the image we can see that it is a QR code:
```
aHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUQ==
```
```
echo "aHR0cHM6Ly95b3V0dS5iZS9kUXc0dzlXZ1hjUQ==" | base64 -d 
https://youtu.be/dQw4w9WgXcQ | "Rick Astley - Never Gonna Give You Up (Official Music Video)"
```
Doh!

The index file that served the jpg file includes "<! –– This is a part of CERT-SE's CTF2021 ––>"

Lets look more in deapth on the file: 
```
file giveup.jpg
giveup.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 90x90, segment length 16, baseline, precision 8, 200x200, components 3
```

Generating my own QR-code "my-giveup.jpg" noticing a file size diffrance. 
Might be something hidden in that giveup.jpg  

```
$ steghide --info giveup.jpg
"giveup.jpg":
  format: jpeg
  capacity: 765.0 Byte
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "flag.txt":
    size: 15.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

Lets try to extract the file:
```
$ steghide extract -sf giveup.jpg
Enter passphrase:
wrote extracted data to "flag.txt".
```
No password needed for this one.

```
cat flag.txt
CTF[chameleon]
```
