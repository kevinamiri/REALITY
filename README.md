# REALITY

### THE NEXT FUTURE

Server side implementation of REALITY protocol, a fork of package tls in Go 1.19.5.  
For client side, please follow https://github.com/XTLS/Xray-core/blob/main/transport/internet/reality/reality.go.  

TODO List: TODO

## VLESS-XTLS-uTLS-REALITY example for [Xray-core](https://github.com/XTLS/Xray-core) [English]

```json5
{
    "inbounds": [ // Server inbound configuration
        {
            "listen": "0.0.0.0",
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "", // Required, execute ./xray uuid to generate, or a 1-30 byte string
                        "flow": "xtls-rprx-vision" // Optional, if present, the client must enable XTLS
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false, // Optional, if true, output debugging information
                    "dest": "example.com:443", // Required, same format as VLESS fallbacks dest
                    "xver": 0, // Optional, same format as VLESS fallbacks xver
                    "serverNames": [ // Required, a list of serverNames available to the client, currently does not support * wildcard
                        "example.com",
                        "www.example.com"
                    ],
                    "privateKey": "", // Required, execute ./xray x25519 to generate
                    "minClientVer": "", // Optional, minimum client Xray version, format is x.y.z
                    "maxClientVer": "", // Optional, maximum client Xray version, format is x.y.z
                    "maxTimeDiff": 0, // Optional, allowed maximum time difference, in milliseconds
                    "shortIds": [ // Required, a list of shortIds available to the client, can be used to distinguish between different clients
                        "" // If this item is present, client shortId can be empty
                        "0123456789abcdef" // 0 to f, length is a multiple of 2, the maximum length is 16
                    ]
                }
            }
        }
    ]
}
```

If REALITY replaces TLS, **it eliminates server-side TLS fingerprint characteristics** while still providing forward secrecy, **and certificate chain attacks are ineffective, making its security surpass conventional TLS**  
**You can point to someone else's website** without having to buy a domain name or configure a TLS server yourself, making it more convenient and **achieving full true TLS with a specified SNI presented to the man in the middle**  

For general proxy purposes, the minimum standard for target websites is: **foreign websites that support TLSv1.3 and H2, and domain names that are not used for redirection** (main domain names might be used for redirection to www)  
Additional points: IP proximity (more similar and lower latency), encrypted handshake messages after Server Hello (e.g., dl.google.com), and OCSP Stapling  
Configuration bonus points: **Disallow traffic from returning to the country, forward TCP/80 and UDP/443 as well** (REALITY outwardly appears as port forwarding, and a less popular target IP might be better)  

**REALITY can also be used with proxy protocols other than XTLS**, but it is not recommended because they have obvious and targeted TLS in TLS characteristics  
The next main goal of REALITY is "**pre-built mode**", i.e., collecting target website features in advance, and the next main goal of XTLS is **0-RTT**  

```json5
{
    "outbounds": [ // Client outbound configuration
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "", // Domain name or IP of the server
                        "port": 443,
                        "users": [
                            {
                                "id": "", // Consistent with the server
                                "flow": "xtls-rprx-vision", // Consistent with the server
                                "encryption": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false, // Optional, if true, output debugging information
                    "fingerprint": "chrome", // Required, use uTLS library to simulate client TLS fingerprint
                    "serverName": "", // One of the serverNames from the server
                    "publicKey": "", // Public key corresponding to the server's private key
                    "shortId": "", // One of the server's shortIds
                    "spiderX": "" // Spider initial path and parameters, it is recommended that each client has a different configuration
                }
            }
        }
    ]
}
```

The REALITY client should receive a "**temporary trusted certificate**" issued by the "**temporary authentication key**", but in the following three cases, it will receive the target website's real certificate:

1. The REALITY server rejects the client's Client Hello, and the traffic is directed to the target website
2. The client's Client Hello is redirected to the target website by a man in the middle
3. A man-in-the-middle attack, which could involve the target website's assistance or a certificate chain attack

The REALITY client can perfectly distinguish between temporary trusted certificates, real certificates, and invalid certificates, and decide on the next course of action:

1. When receiving a temporary trusted certificate, the connection is usable, and everything proceeds as normal
2. When receiving a real certificate, the client enters spider mode
3. When receiving an invalid certificate, a TLS alert is triggered, and the connection is disconnected
