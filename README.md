# 📡 OpenVPN – Konfiguracja Serwera i Klienta (Debian)

Minimalny, ale kompletny tutorial instalacji i konfiguracji OpenVPN na systemie Debian (serwer + klient) z użyciem Easy-RSA.

---

## ✅ Założenia

- Serwer: Debian (np. `192.168.0.107`)
- Klient: inna maszyna w tej samej sieci
- Uprawnienia: `root` lub `sudo`
- Pakiety: `openvpn`, `easy-rsa`, `openssh-server` (na serwerze)

---

## 🧱 1. Instalacja (na serwerze)

```bash
apt update
apt install openvpn easy-rsa -y
```

---

## 📂 2. Inicjalizacja Easy-RSA

```bash
mkdir ~/openvpn-ca && cd ~/openvpn-ca
ln -s /usr/share/easy-rsa/* .
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key
```

---

## 📁 3. Kopiowanie plików do OpenVPN

```bash
cp pki/ca.crt pki/issued/server.crt pki/private/server.key pki/dh.pem ta.key /etc/openvpn/
```

---

## ⚙️ 4. Konfiguracja serwera `/etc/openvpn/server.conf`

```ini
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
auth SHA256
tls-auth ta.key 0
server 10.8.0.0 255.255.255.0
persist-key
persist-tun
keepalive 10 120
status openvpn-status.log
verb 3
```

---

## 🚀 5. Uruchomienie serwera

```bash
systemctl start openvpn@server
systemctl enable openvpn@server
systemctl status openvpn@server
```

---

## 👤 6. Tworzenie klienta

```bash
cd ~/openvpn-ca
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

```bash
cp pki/issued/client1.crt pki/private/client1.key /etc/openvpn/
```

---

## 🧾 7. Tworzenie pliku konfiguracyjnego `client1.ovpn`

Zapisz jako `/root/client1.ovpn`:

```ini
client
dev tun
proto udp
remote 192.168.0.107 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
auth SHA256
tls-auth ta.key 1
verb 3

<ca>
(wklej zawartość ca.crt)
</ca>

<cert>
(wklej zawartość client1.crt)
</cert>

<key>
(wklej zawartość client1.key)
</key>

<tls-auth>
(wklej zawartość ta.key)
</tls-auth>
```

---

## 🔄 8. Kopiowanie `client1.ovpn` na klienta

### A. Włącz SSH na serwerze

```bash
apt install openssh-server -y
systemctl enable ssh
systemctl start ssh
```

### B. Skopiuj plik z klienta

```bash
scp root@192.168.0.107:/root/client1.ovpn ~/
```

---

## 🧪 9. Uruchomienie OpenVPN na kliencie

```bash
apt install openvpn -y
sudo openvpn --config ~/client1.ovpn
```

---

## ✅ Gotowe!

Po połączeniu klienta z serwerem, możesz sprawdzić interfejs `tun0`:

```bash
ip a
```

---

## 🛡️ Uwagi bezpieczeństwa

- Produkcyjnie warto zabezpieczyć klucze hasłem (`build-req`)
- Użyj `tls-crypt` zamiast `tls-auth` (szyfruje + uwierzytelnia)
- Dodaj NAT i routowanie, jeśli chcesz dostęp do sieci

---

## 📚 Źródła

- [OpenVPN Documentation](https://openvpn.net/community-resources/)
- [Easy-RSA GitHub](https://github.com/OpenVPN/easy-rsa)
