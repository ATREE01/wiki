# VPN

## WireGuard

下面這個檔案基本上可以用GUI介面生成，就稍微寫一下代表甚麼

```ini
# ws.conf
[Interface]
PrivateKey = 
# Peer的IP
Address = 10.0.0.2/32 
# DNS 通常是填8.8.8.8 1.1.1.1 或 WireGuard host的位置
DNS = 

[Peer]
PublicKey = 
PresharedKey =
# 限制有哪些IP需要走VPN 下面這種寫法就是全部都走VPN(不建議)
AllowedIPs = 0.0.0.0/0, ::/0
# Wireguard host的IP Port 預設是51820
Endpoint = 140.115.16.180:51820
PersistentKeepAlive = 25
```