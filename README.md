# Configurasi Openvpn

## NOTE : PAKAI TAB YA BIAR CEPET NYARI FILE ATAU CONFIGNYA

## CA bikin sendiri ya
`pakai file ca v2.txt`

## Server Side
1.Install openvpn
`apt install openvpn -y`

cari file
`cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /apt/openvpn/server`

2.edit file server.conf
```text
port 1194
proto udp
dev tun

ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/server/dh.pem

server 10.0.10.0 255.255.255.0
topology subnet

# Push routes to INT (10.0.1.0/24) and DMZ (10.0.2.0/24)
push "route 10.0.1.0 255.255.255.0"
push "route 10.0.2.0 255.255.255.0"

# Push internal DNS server and search domain (update IP to match your DNS server)
push "dhcp-option DNS <INT_DNS_SERVER_IP>"
push "dhcp-option DOMAIN nusantara.id"

# System user authentication via PAM (no client certificate required)
plugin /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so login
verify-client-cert none
username-as-common-name

keepalive 10 120
cipher AES-256-GCM
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
```
3.bikin user vpn
`sudo useradd -m -s /bin/bash -c "Boaz Salossa" boaz`

4.ganti password
`passwd boaz`
isi Skills39!

5.nyalakan
`sudo systemctl enable --now openvpn-server@server`

## Client Side

1.login root
`su -`

2.Create user boaz with passwordless sudo
`sudo useradd -m -s /bin/bash -c "Boaz Salossa" boaz`

3.ganti password
`passwd boaz`
isi Skills39!

4.passwordless sudo
`echo "boaz ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/boaz`
`sudo chmod 440 /etc/sudoers.d/boaz`

5.Install Nusantara Root CA into the client OS trust store
`sudo cp /tmp/ca.pem /usr/local/share/ca-certificates/nusantara-root-ca.crt`
`sudo update-ca-certificates`

5. Install openvpn
`apt install -y openvpn-systemd-resolved openvpn dns-utils`

6.cari file
`cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /apt/openvpn/client.ovpn`

7.bikin folder
`sudo mkdir -p /etc/openvpn/client /etc/openvpn/auth``

6.edit file user.txt
`nano /etc/openvpn/auth/user.txt`
```text

boaz
Skills39s!
```
`sudo chmod 600 /etc/openvpn/auth/user.txt`

7.Copy Root CA for OpenVPN verification
`sudo cp /usr/local/share/ca-certificates/nusantara-root-ca.crt /etc/openvpn/client/ca.crt`

8.Edit File client.ovpn
```text
client
dev tun
proto udp
remote <FW_EXTERNAL_IP_OR_FQDN> 1194
resolv-retry infinite
nobind
persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
remote-cert-tls server
auth-user-pass /etc/openvpn/auth/user.txt

cipher AES-256-GCM
verb 3

script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```
9.nyalakan
`sudo openvpn --config /etc/openvpn/client/client.ovpn`
