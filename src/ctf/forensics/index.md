# Forensics

## Unknown file

```bash
$ file hardshells
hardshells: Zip archive data, at least v1.0 to extract
$ mv hardshells hardshells.zip
```

## Data file tricks

```bash
$ file dat
dat: data
```

Use `hexedit` or `bless` to open.

If you find:

`IHDR` or `IDAT` - are section headers for PNG. Search for png magic headers/bytes

`@ICC_PROFILE` or `Adobe` anywhere - Search for JPEG Magic headers/bytes

## GIF

### Split frames of GIF
`$ convert picasso.gif %02d.png`

## PNG

### Convert white pixels into transparent pixels (several files)
`$ ls *.png | while read filename; do convert $filename -transparent white $filename; done`

### Stack/Overlay images on top of each other
`$ ls *.png | while read filename; do convert $filename 00.png -gravity center -composite 00.png; done`

## JPEG

### placeholder


## Zip

`PK` - ZIP Magic File Header

### Bruteforce Zip Password
Download rockyou.txt (it's a wordlist)

`$ fcrackzip -v -D -u -p rockyou.txt hardshells.zip`

## Filesystems

```bash
$ file dat
dat: Minix filesystem, V1, 30 char names, 20 zones
$ mkdir mountpoint && sudo mount dat mountpoint/
```

## PCAP (Packet Capture)

`$ tcpflow -r thunder.pcap` - Will output files that go from one IP to another

`$ binwalk -e thunder.pcap`

`$ foremost thunder.pcap`

`$ strings thunder.pcap | grep -r "flag"` - Shot in the dark

## Wireshark

Given a `.pem` file, go to Settings > Preferences > Protocol > SSL 
Add RSA key list with `.pem` for IP Address you need to decrypt
Enter name for SSL debug file

## Steganography

[https://0xrick.github.io/lists/stego/](https://0xrick.github.io/lists/stego/)