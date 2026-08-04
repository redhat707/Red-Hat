# 3. feladat – Statikus IPv4- és IPv6-cím beállítása NetworkManagerrel

## Feladat

Hozz létre egy új NetworkManager kapcsolatot az `enp0s3` hálózati interfészhez.

A kapcsolat neve legyen:

```text
lab-link
```

Állítsd be a következő hálózati adatokat:

### Elsődleges IPv4-cím

```text
192.168.1.10/24
```

### IPv4 átjáró

```text
192.168.1.1
```

### DNS-szerver

```text
8.8.8.8
```

### IPv6-cím

```text
fd00::10/64
```

### IPv6 átjáró

```text
fd00::1
```

### Másodlagos IPv4-cím

```text
10.0.0.5/24
```

A kapcsolat rendszerindításkor automatikusan aktiválódjon.

---

# Megoldás

## 1. A hálózati interfész ellenőrzése

```bash
nmcli device status
```

vagy:

```bash
ip link show
```

A feladatban használt interfész:

```text
enp0s3
```

A vizsgán mindig a tényleges interfésznevet kell használni.

---

## 2. A kapcsolat létrehozása

```bash
nmcli connection add \
con-name lab-link \
ifname enp0s3 \
type ethernet \
ipv4.method manual \
ipv4.addresses 192.168.1.10/24 \
ipv4.gateway 192.168.1.1 \
ipv4.dns 8.8.8.8 \
ipv6.method manual \
ipv6.addresses fd00::10/64 \
ipv6.gateway fd00::1 \
connection.autoconnect yes
```

A parancs létrehozza a `lab-link` nevű kapcsolatot, és beállítja az elsődleges IPv4- és IPv6-címeket.

---

## 3. Másodlagos IPv4-cím hozzáadása

```bash
nmcli connection modify lab-link +ipv4.addresses 10.0.0.5/24
```

A `+` jel azt jelenti, hogy az új IP-cím hozzáadódik a már meglévő címhez.

Nagyon fontos:

```bash
+ipv4.addresses
```

Ha a `+` jel nélkül adjuk meg:

```bash
ipv4.addresses 10.0.0.5/24
```

akkor a korábbi IPv4-cím felülíródhat.

---

## 4. A kapcsolat aktiválása

```bash
nmcli connection up lab-link
```

---

# Ellenőrzés

## Az IP-címek ellenőrzése

```bash
ip address show enp0s3
```

Rövidebb formában:

```bash
ip a show enp0s3
```

A kimenetben szerepelnie kell:

```text
192.168.1.10/24
10.0.0.5/24
fd00::10/64
```

---

## A kapcsolat ellenőrzése

```bash
nmcli connection show lab-link
```

Csak a fontosabb beállítások megjelenítése:

```bash
nmcli -f connection.id,connection.autoconnect,ipv4.method,ipv4.addresses,ipv4.gateway,ipv4.dns,ipv6.method,ipv6.addresses,ipv6.gateway connection show lab-link
```

---

## Az aktív kapcsolatok ellenőrzése

```bash
nmcli connection show --active
```

A listában meg kell jelennie:

```text
lab-link
```

---

## Az IPv4 útvonal ellenőrzése

```bash
ip route
```

A kimenetben várhatóan szerepel:

```text
default via 192.168.1.1 dev enp0s3
```

---

## Az IPv6 útvonal ellenőrzése

```bash
ip -6 route
```

A kimenetben várhatóan szerepel:

```text
default via fd00::1 dev enp0s3
```

---

## A DNS-beállítás ellenőrzése

```bash
nmcli device show enp0s3 | grep DNS
```

vagy:

```bash
cat /etc/resolv.conf
```

A DNS-szerver:

```text
8.8.8.8
```

---

# Beállítások újraindítás után

A NetworkManagerrel létrehozott kapcsolat alapértelmezetten tartós.

A kapcsolat profilja a következő könyvtárban található:

```text
/etc/NetworkManager/system-connections/
```

A kapcsolat automatikus indítását ez biztosítja:

```bash
connection.autoconnect yes
```

Ellenőrzés:

```bash
nmcli -g connection.autoconnect connection show lab-link
```

A várt eredmény:

```text
yes
```

Újraindítás után ellenőrizhető:

```bash
reboot
```

Majd:

```bash
nmcli connection show --active
ip address show enp0s3
```

A kapcsolatnak és az IP-címeknek újraindítás után is meg kell maradniuk.

---

# Részletes magyarázat

## `nmcli connection add`

Új NetworkManager kapcsolatprofilt hoz létre.

```bash
nmcli connection add
```

---

## `con-name lab-link`

A létrehozott kapcsolat neve:

```text
lab-link
```

---

## `ifname enp0s3`

Meghatározza, hogy a kapcsolat melyik hálózati interfészhez tartozik.

```text
enp0s3
```

---

## `type ethernet`

A kapcsolat típusa vezetékes Ethernet-kapcsolat.

```bash
type ethernet
```

---

## `ipv4.method manual`

Kikapcsolja a DHCP használatát, és kézi IPv4-beállítást engedélyez.

```bash
ipv4.method manual
```

---

## `ipv4.addresses`

Az IPv4-cím és a hálózati prefix:

```text
192.168.1.10/24
```

A `/24` hálózati maszkja:

```text
255.255.255.0
```

---

## `ipv4.gateway`

Az alapértelmezett IPv4-átjáró:

```text
192.168.1.1
```

---

## `ipv4.dns`

A DNS-szerver:

```text
8.8.8.8
```

---

## `ipv6.method manual`

Kézi IPv6-beállítást engedélyez.

```bash
ipv6.method manual
```

---

## `ipv6.addresses`

Az IPv6-cím:

```text
fd00::10/64
```

---

## `ipv6.gateway`

Az IPv6 alapértelmezett átjáró:

```text
fd00::1
```

---

## `connection.autoconnect yes`

A kapcsolat automatikusan aktiválódik a rendszer indulásakor.

```bash
connection.autoconnect yes
```

---

# Hibakeresés

## A hálózati interfész neve nem `enp0s3`

Ellenőrzés:

```bash
nmcli device status
```

vagy:

```bash
ip link show
```

Ezután az `enp0s3` helyett a tényleges interfésznevet kell használni.

Például:

```text
ens160
```

vagy:

```text
enp1s0
```

---

## A `lab-link` kapcsolat már létezik

Ellenőrzés:

```bash
nmcli connection show
```

A meglévő kapcsolat törlése:

```bash
nmcli connection delete lab-link
```

Ezután újra létrehozható.

---

## A régi kapcsolat ütközik az új kapcsolattal

Az aktív kapcsolatok ellenőrzése:

```bash
nmcli connection show --active
```

A régi kapcsolat leállítása:

```bash
nmcli connection down "régi-kapcsolat-neve"
```

Az új kapcsolat aktiválása:

```bash
nmcli connection up lab-link
```

---

## A másodlagos IP-cím felülírta az elsődleges címet

Ez akkor történhet meg, ha a `+` jel kimaradt.

Helytelen:

```bash
nmcli connection modify lab-link ipv4.addresses 10.0.0.5/24
```

Helyes:

```bash
nmcli connection modify lab-link +ipv4.addresses 10.0.0.5/24
```

Az IPv4-címek ellenőrzése:

```bash
nmcli -g ipv4.addresses connection show lab-link
```

---

## A módosítás nem jelenik meg azonnal

A kapcsolat újraaktiválása:

```bash
nmcli connection down lab-link
nmcli connection up lab-link
```

vagy:

```bash
nmcli device reapply enp0s3
```

Vizsgán a legbiztosabb:

```bash
nmcli connection up lab-link
```

---

## Nincs hálózati kapcsolat

Az interfész állapotának ellenőrzése:

```bash
nmcli device status
```

Az átjáró tesztelése:

```bash
ping -c 3 192.168.1.1
```

A DNS-szerver tesztelése:

```bash
ping -c 3 8.8.8.8
```

Az IPv6-átjáró tesztelése:

```bash
ping6 -c 3 fd00::1
```

A ping csak akkor működik, ha a megadott címek ténylegesen elérhetők a laborhálózaton.

---

# Rövid vizsgamegoldás

```bash
nmcli connection add \
con-name lab-link \
ifname enp0s3 \
type ethernet \
ipv4.method manual \
ipv4.addresses 192.168.1.10/24 \
ipv4.gateway 192.168.1.1 \
ipv4.dns 8.8.8.8 \
ipv6.method manual \
ipv6.addresses fd00::10/64 \
ipv6.gateway fd00::1 \
connection.autoconnect yes
```

```bash
nmcli connection modify lab-link +ipv4.addresses 10.0.0.5/24
```

```bash
nmcli connection up lab-link
```

```bash
ip address show enp0s3
```

```bash
nmcli connection show lab-link
```

---

# Egysoros változat

```bash
nmcli con add con-name lab-link ifname enp0s3 type ethernet ipv4.method manual ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv6.method manual ipv6.addresses fd00::10/64 ipv6.gateway fd00::1 connection.autoconnect yes
```

```bash
nmcli con mod lab-link +ipv4.addresses 10.0.0.5/24
```

```bash
nmcli con up lab-link
```

```bash
ip addr show enp0s3
```

---

# Összefoglalás

A `lab-link` kapcsolatot az `enp0s3` interfészhez hoztuk létre.

A beállított címek:

```text
IPv4:             192.168.1.10/24
Másodlagos IPv4:  10.0.0.5/24
IPv4 gateway:     192.168.1.1
DNS:              8.8.8.8
IPv6:             fd00::10/64
IPv6 gateway:     fd00::1
```

A másodlagos IPv4-címet a `+ipv4.addresses` beállítással adtuk hozzá, így az elsődleges cím nem íródott felül.

A kapcsolat újraindítás után is megmarad, mert a NetworkManager tartós kapcsolatprofilt hoz létre, és az automatikus kapcsolódás engedélyezve van:

```bash
connection.autoconnect yes
```
