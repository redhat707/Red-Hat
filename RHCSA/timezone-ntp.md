# 4. feladat – Időzóna és NTP-időszinkronizálás beállítása

## Feladat

Állítsd be a rendszer időzónáját erre:

```text
America/New_York
```

Állítsd be a Chrony szolgáltatást úgy, hogy a következő NTP-kiszolgálót használja:

```text
pool.ntp.org
```

A szolgáltatás rendszerindításkor automatikusan induljon el, és az időszinkronizálás újraindítás után is működjön.

---

# Megoldás

## 1. A jelenlegi időbeállítás ellenőrzése

```bash
timedatectl
```

A parancs megmutatja többek között:

- a helyi időt;
- az UTC-időt;
- az aktuális időzónát;
- az NTP-szinkronizálás állapotát.

---

## 2. Az időzóna beállítása

```bash
timedatectl set-timezone America/New_York
```

Ellenőrzés:

```bash
timedatectl
```

A kimenetben ennek kell szerepelnie:

```text
Time zone: America/New_York
```

---

## 3. Az elérhető időzónák megjelenítése

Ha nem vagyunk biztosak az időzóna pontos nevében:

```bash
timedatectl list-timezones
```

Keresés egy adott időzónára:

```bash
timedatectl list-timezones | grep New_York
```

A várt eredmény:

```text
America/New_York
```

---

# Chrony NTP-szolgáltatás beállítása

## 4. A Chrony telepítésének ellenőrzése

```bash
rpm -q chrony
```

Ha nincs telepítve:

```bash
dnf install -y chrony
```

---

## 5. A Chrony konfigurációs fájl szerkesztése

```bash
vim /etc/chrony.conf
```

Adjunk hozzá egy ilyen sort:

```conf
pool pool.ntp.org iburst
```

---

## A konfiguráció jelentése

```conf
pool pool.ntp.org iburst
```

A `pool` azt jelenti, hogy a Chrony egy NTP-kiszolgálócsoportból választ elérhető szervereket.

A `pool.ntp.org` egy NTP-kiszolgálókat biztosító cím.

Az `iburst` felgyorsítja a kezdeti időszinkronizálást azzal, hogy induláskor több gyors lekérdezést küld.

---

## 6. Meglévő NTP-beállítások ellenőrzése

A konfigurációban már lehetnek `server` vagy `pool` sorok.

Ellenőrzés:

```bash
grep -E '^(server|pool)' /etc/chrony.conf
```

Példa:

```text
pool 2.rhel.pool.ntp.org iburst
pool pool.ntp.org iburst
```

Ha a feladat kifejezetten azt kéri, hogy csak a megadott NTP-kiszolgálót használd, akkor a többi `server` és `pool` sort ki kell kommentelni vagy törölni.

Példa kikommentelésre:

```conf
# pool 2.rhel.pool.ntp.org iburst
pool pool.ntp.org iburst
```

Ha a feladat csak azt kéri, hogy add hozzá a megadott szervert, akkor a meglévő sorok maradhatnak.

---

# A szolgáltatás aktiválása

## 7. A Chrony engedélyezése és elindítása

```bash
systemctl enable --now chronyd
```

Ez a parancs egyszerre:

- elindítja a `chronyd` szolgáltatást;
- engedélyezi az automatikus indulását.

---

## 8. A konfiguráció újratöltése

Ha a `chronyd` már futott a konfiguráció módosításakor:

```bash
systemctl restart chronyd
```

A teljes biztonságos megoldás:

```bash
systemctl enable --now chronyd
systemctl restart chronyd
```

---

# Ellenőrzés

## 9. A szolgáltatás állapotának ellenőrzése

```bash
systemctl status chronyd
```

Rövidebb ellenőrzés:

```bash
systemctl is-active chronyd
```

A várt eredmény:

```text
active
```

Az automatikus indulás ellenőrzése:

```bash
systemctl is-enabled chronyd
```

A várt eredmény:

```text
enabled
```

---

## 10. Az időzóna és az NTP állapotának ellenőrzése

```bash
timedatectl
```

A fontos sorok:

```text
Time zone: America/New_York
System clock synchronized: yes
NTP service: active
```

A szinkronizálás nem mindig történik meg azonnal, ezért a `System clock synchronized` értéke kezdetben még lehet:

```text
no
```

Néhány másodperc vagy perc után újra ellenőrizhető.

---

## 11. Az NTP-források ellenőrzése

```bash
chronyc sources
```

Részletesebb megjelenítés:

```bash
chronyc sources -v
```

Példa:

```text
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ntp-server.example.org        2   6   377    20   +15us[ +20us] +/- 15ms
```

---

# A `chronyc sources` jelölései

## `^*`

Az aktuálisan használt NTP-kiszolgáló:

```text
^*
```

## `^+`

Elfogadható másodlagos időforrás:

```text
^+
```

## `^-`

Elérhető, de jelenleg nem használt időforrás:

```text
^-
```

## `^?`

Az időforrás még nem elérhető, vagy nincs elegendő mérési adat:

```text
^?
```

Közvetlenül a szolgáltatás újraindítása után a `^?` jelölés átmenetileg normális lehet.

---

## 12. A szinkronizálás részletes ellenőrzése

```bash
chronyc tracking
```

A parancs többek között megmutatja:

- melyik referencia-időforrást használja a rendszer;
- mekkora az időeltérés;
- szinkronizálva van-e a rendszeróra.

---

## 13. Az NTP-kiszolgáló konfigurációjának ellenőrzése

```bash
grep -E '^(server|pool)' /etc/chrony.conf
```

A kimenetben meg kell jelennie:

```text
pool pool.ntp.org iburst
```

---

# Beállítások újraindítás után

Az időzóna beállítása tartós, ezért újraindítás után is megmarad:

```bash
timedatectl set-timezone America/New_York
```

A Chrony konfigurációja ebben a fájlban található:

```text
/etc/chrony.conf
```

A szolgáltatás automatikus indítását ez a parancs biztosítja:

```bash
systemctl enable chronyd
```

Mivel ezt használtuk:

```bash
systemctl enable --now chronyd
```

a szolgáltatás újraindítás után is automatikusan elindul.

Újraindítás után ellenőrizhető:

```bash
timedatectl
```

```bash
systemctl is-enabled chronyd
```

```bash
systemctl is-active chronyd
```

```bash
chronyc sources -v
```

---

# Hibakeresés

## A `chronyd` szolgáltatás nem található

Ellenőrzés:

```bash
rpm -q chrony
```

Telepítés:

```bash
dnf install -y chrony
```

Ezután:

```bash
systemctl enable --now chronyd
```

---

## A szolgáltatás nem indul el

Állapot ellenőrzése:

```bash
systemctl status chronyd
```

Napló megtekintése:

```bash
journalctl -u chronyd
```

Az aktuális rendszerindítás naplója:

```bash
journalctl -u chronyd -b
```

---

## A konfiguráció módosítása nem lépett életbe

Indítsuk újra a szolgáltatást:

```bash
systemctl restart chronyd
```

Ezután:

```bash
chronyc sources -v
```

---

## Az NTP-forrás mellett `^?` jelenik meg

Ez azt jelentheti, hogy:

- a rendszernek még időre van szüksége;
- nincs hálózati kapcsolat;
- a DNS-feloldás nem működik;
- az NTP-szerver nem érhető el;
- a tűzfal vagy a hálózat blokkolja az NTP-forgalmat.

Hálózati kapcsolat ellenőrzése:

```bash
ping -c 3 pool.ntp.org
```

DNS-feloldás ellenőrzése:

```bash
getent hosts pool.ntp.org
```

A Chrony újraindítása:

```bash
systemctl restart chronyd
```

Majd várjunk rövid ideig, és ellenőrizzük újra:

```bash
chronyc sources -v
```

---

## A `timedatectl` szerint nincs szinkronizálva az idő

Ellenőrzés:

```bash
chronyc tracking
```

```bash
chronyc sources -v
```

A Chrony azonnali órakorrekciója:

```bash
chronyc makestep
```

Ezután:

```bash
timedatectl
```

---

## Nem megfelelő az időzóna neve

Az időzónák keresése:

```bash
timedatectl list-timezones | grep -i new
```

A helyes időzóna:

```text
America/New_York
```

A beállítás:

```bash
timedatectl set-timezone America/New_York
```

---

# Rövid vizsgamegoldás

## Időzóna beállítása

```bash
timedatectl set-timezone America/New_York
```

## Chrony konfiguráció szerkesztése

```bash
vim /etc/chrony.conf
```

Hozzáadandó sor:

```conf
pool pool.ntp.org iburst
```

## Szolgáltatás engedélyezése és újraindítása

```bash
systemctl enable --now chronyd
```

```bash
systemctl restart chronyd
```

## Ellenőrzés

```bash
timedatectl
```

```bash
chronyc sources -v
```

```bash
chronyc tracking
```

---

# Gyors, parancssoros megoldás

A konfigurációs sor hozzáadása szerkesztő nélkül:

```bash
echo 'pool pool.ntp.org iburst' >> /etc/chrony.conf
```

Ezután:

```bash
timedatectl set-timezone America/New_York
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources -v
```

Fontos: az `echo >>` minden futtatáskor újabb ugyanolyan sort adhat a fájlhoz. Vizsgán biztonságosabb előtte ellenőrizni:

```bash
grep -F 'pool pool.ntp.org iburst' /etc/chrony.conf
```

Csak akkor adjuk hozzá, ha még nincs benne.

---

# Összefoglalás

Az időzónát a következő paranccsal állítottuk be:

```bash
timedatectl set-timezone America/New_York
```

A Chrony konfigurációjához hozzáadtuk:

```conf
pool pool.ntp.org iburst
```

A szolgáltatást elindítottuk és engedélyeztük a rendszerindításhoz:

```bash
systemctl enable --now chronyd
```

A beállításokat ezekkel ellenőriztük:

```bash
timedatectl
chronyc sources -v
chronyc tracking
```

Az időzóna és a Chrony konfiguráció újraindítás után is megmarad.
