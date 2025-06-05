# 📡 OpenVPN – Konfiguracja Serwera i Klienta (Debian)

Przewodnik po instalacji OpenVPN
---

## Założenia

- Serwer: Debian
- Klient: inna maszyna w tej samej sieci
- Pakiety: `openvpn`, `easy-rsa`

---

##  1. Instalacja (na serwerze)

```bash
apt update
apt install openvpn easy-rsa -y
```

---

##  2. Inicjalizacja Easy-RSA

```bash
make-cadir ~/openvpn-ca
```
```bash
cd ~/openvpn-ca
```
```bash
./easyrsa init-pki
```
```bash
./easyrsa build-ca nopass
```
```bash
./easyrsa gen-req serwer nopass
```
```bash
./easyrsa sign-req server serwer
```
```bash
./easyrsa gen-dh
```
```bash
openvpn --genkey --secret ta.key
```
---

##  3. Kopiowanie plików do OpenVPN

```bash
sudo cp pki/ca.crt pki/private/serwer.key pki/issued/serwer.crt pki/dh.pem ta.key /etc/openvpn/
```

---

##  4. Utworzenie pliku konfiguracyjnego serwera

```bash
sudo nano /etc/openvpn/server.conf
```

---


##  5. Konfiguracja serwera `/etc/openvpn/server.conf`

```bash
port 1194                  # Port, na którym działa VPN
proto udp                  # Protokół (UDP = szybszy)
dev tun                    # Użycie interfejsu tunelowego (tun = IP-level VPN)
ca ca.crt                  # Certyfikat CA
cert serwer.crt            # Certyfikat serwera
key serwer.key             # Klucz prywatny serwera
dh dh.pem                  # Parametry DH
auth SHA256                # Algorytm HMAC
tls-auth ta.key 0          # Klucz TLS-Auth i jego rola (0 = serwer)
server 10.8.0.0 255.255.255.0  # Zakres IP dla klientów VPN
keepalive 10 120           # Utrzymywanie połączenia
persist-key
persist-tun
status openvpn-status.log
verb 3                     # Poziom logowania (więcej = bardziej szczegółowo)

```

---

##  6. Uruchomienie serwera

```bash
systemctl start openvpn@server
systemctl enable openvpn@server
systemctl status openvpn@server
```
---

##  7. Tworzenie klienta

```bash
cd ~/openvpn-ca
./easyrsa build-client-full client1 nopass
```
---

##  8. Tworzenie pliku konfiguracyjnego `client1.ovpn`

Zapisz jako `/root/client1.ovpn`:

```bash
cd ~/openvpn-ca
# Przykładowa komenda do wygenerowania klienta OVPN (z certyfikatem, kluczem i ta.key)
cat > ~/client1.ovpn <<EOF
client
dev tun
proto udp
remote IP_SERWERA 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
auth SHA256
key-direction 1
<ca>
$(cat pki/ca.crt)
</ca>
<cert>
$(cat pki/issued/client1.crt)
</cert>
<key>
$(cat pki/private/client1.key)
</key>
<tls-auth>
$(cat ta.key)
</tls-auth>
EOF

```

---

##  9. Kopiowanie `client1.ovpn` na klienta

### A. Włącz SSH na serwerze

```bash
apt install openssh-server -y
systemctl enable ssh
systemctl start ssh
```

### B. Skopiuj plik z klienta

```bash
scp root@IP_SERWERA:/root/client1.ovpn ~/
```

---

##  10. Uruchomienie OpenVPN na kliencie

```bash
apt install openvpn -y
sudo openvpn --config ~/client1.ovpn
```

---

##  Gotowe!

Po połączeniu klienta z serwerem, możesz sprawdzić interfejs `tun0`:

```bash
ip a
```

---
