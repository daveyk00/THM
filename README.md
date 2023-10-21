# THM
Walkthrough of tryhackme.com boxes

# MTU
I occasionally need to change MTU details for the VPN connection
When this is the case SSH fails to connect - verbose logging shows:
`debug1: expecting SSH2_MSG_KEX_ECDH_REPLY`

To change the MTU
`sudo ip li set mtu 1200 dev tun0`
