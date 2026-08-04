# 2. feladat – RHEL 10 repository beállítása

## Feladat

Állíts be használható BaseOS és AppStream repositorykat RHEL 10 rendszeren.

A vizsgán vagy laborban a repository forrása háromféleképpen lehet megadva:

1. az ISO virtuális DVD-meghajtóként van csatlakoztatva;
2. az ISO egy fájlként található a rendszeren;
3. a repository hálózati címen érhető el.

Mindig a feladatban megadott elérési utat vagy URL-t kell használni.

---

# 1. változat – ISO virtuális DVD-meghajtóban

Ebben az esetben az ISO már be van helyezve a virtuális CD/DVD-meghajtóba.

## A meghajtó megkeresése

```bash
lsblk
```

A virtuális DVD-meghajtó általában:

```text
sr0
```

Az eszköz teljes elérési útja:

```text
/dev/sr0
```

## Csatolási pont létrehozása

```bash
mkdir -p /mnt
```

## A DVD csatolása

```bash
mount /dev/sr0 /mnt
```

Mivel a DVD csak olvasható, használható ez is:

```bash
mount -o ro /dev/sr0 /mnt
```

## Ellenőrzés

```bash
ls /mnt
```

A következő könyvtáraknak kell megjelenniük:

```text
BaseOS
AppStream
```

## Fontos

Virtuális DVD-meghajtó esetén nem kell a `loop` opció.

Helyes:

```bash
mount /dev/sr0 /mnt
```

Helytelen:

```bash
mount -o loop sr0 /mnt
```

A `/dev/` előtagot is ki kell írni.

---

# 2. változat – Az ISO fájlként található a rendszeren

Ebben az esetben az ISO például ilyen helyen található:

```text
/root/RHEL-10.iso
```

vagy:

```text
/tmp/RHEL-10.iso
```

## Az ISO megkeresése

```bash
find / -type f -name "*.iso" 2>/dev/null
```

## Csatolási pont létrehozása

```bash
mkdir -p /mnt
```

## Az ISO-fájl csatolása

Példa:

```bash
mount -o loop /root/RHEL-10.iso /mnt
```

Csak olvasható módban:

```bash
mount -o loop,ro /root/RHEL-10.iso /mnt
```

## Ellenőrzés

```bash
ls /mnt
```

A következő könyvtáraknak kell megjelenniük:

```text
BaseOS
AppStream
```

## Fontos

A `loop` opció csak akkor kell, amikor valódi ISO-fájlt csatolunk.

Példa:

```bash
mount -o loop /root/RHEL-10.iso /mnt
```

---

# 3. változat – Hálózati repository

Ebben az esetben nem kell ISO-fájlt vagy DVD-meghajtót csatolni.

A feladat megadja a BaseOS és AppStream repository URL-jét.

Példa:

```text
http://server.example.com/rhel10/BaseOS
http://server.example.com/rhel10/AppStream
```

Először ellenőrizzük a hálózati kapcsolatot:

```bash
ping -c 2 server.example.com
```

Az URL elérhetőségét is ellenőrizhetjük:

```bash
curl -I http://server.example.com/rhel10/BaseOS/
```

---

# Repository konfiguráció létrehozása

A repository konfigurációs fájl helye:

```text
/etc/yum.repos.d/local.repo
```

A fájl létrehozása:

```bash
vim /etc/yum.repos.d/local.repo
```

---

# Repository fájl DVD vagy ISO használatakor

Ha a DVD vagy ISO a `/mnt` könyvtárba van csatolva, a fájl tartalma:

```ini
[BaseOS]
name=RHEL 10 BaseOS
baseurl=file:///mnt/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=RHEL 10 AppStream
baseurl=file:///mnt/AppStream
enabled=1
gpgcheck=0
```

A `file:///` azt jelenti, hogy a repository a helyi fájlrendszeren található.

---

# Repository fájl hálózati repository esetén

A feladatban megadott címeket kell használni.

Példa:

```ini
[BaseOS]
name=RHEL 10 BaseOS
baseurl=http://server.example.com/rhel10/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=RHEL 10 AppStream
baseurl=http://server.example.com/rhel10/AppStream
enabled=1
gpgcheck=0
```

A példában szereplő URL-t mindig a vizsgafeladatban megadott címre kell cserélni.

---

# A repositoryk ellenőrzése

A DNF gyorsítótár törlése:

```bash
dnf clean all
```

A repositoryk listázása:

```bash
dnf repolist
```

Sikeres beállítás esetén megjelenik:

```text
BaseOS
AppStream
```

Részletes ellenőrzés:

```bash
dnf repoinfo BaseOS
```

```bash
dnf repoinfo AppStream
```

A repository metaadatainak újraépítése:

```bash
dnf makecache
```

Egy csomag keresésével is tesztelhetjük:

```bash
dnf list available
```

vagy:

```bash
dnf search bash
```

---

# Gyors döntési segédlet

## Ha az `lsblk` parancsban látszik az `sr0`

```bash
mount /dev/sr0 /mnt
```

Ezután a repository elérési útja:

```ini
baseurl=file:///mnt/BaseOS
```

és:

```ini
baseurl=file:///mnt/AppStream
```

---

## Ha kaptál egy `.iso` fájlt

Példa:

```bash
mount -o loop /root/RHEL-10.iso /mnt
```

Ezután a repository elérési útja:

```ini
baseurl=file:///mnt/BaseOS
```

és:

```ini
baseurl=file:///mnt/AppStream
```

---

## Ha HTTP- vagy HTTPS-címet kaptál

Nem kell semmit csatolni.

A megadott URL-eket közvetlenül a repository fájlba kell írni:

```ini
baseurl=http://server.example.com/rhel10/BaseOS
```

és:

```ini
baseurl=http://server.example.com/rhel10/AppStream
```

---

# Hibakeresés

## A `/mnt` már használatban van

Ellenőrzés:

```bash
mount | grep /mnt
```

Lecsatolás:

```bash
umount /mnt
```

Ezután újra csatolható az ISO vagy a DVD.

---

## Az `sr0` nem található

Ellenőrzés:

```bash
lsblk
```

```bash
ls -l /dev/sr*
```

Ha nincs `/dev/sr0`, akkor valószínűleg nincs ISO behelyezve a virtuális DVD-meghajtóba.

---

## Hiba: failed to setup loop device

Ez általában akkor történik, ha blokkos eszközt próbálunk ISO-fájlként csatolni.

Helytelen:

```bash
mount -o loop sr0 /mnt
```

Helyes virtuális DVD esetén:

```bash
mount /dev/sr0 /mnt
```

Helyes ISO-fájl esetén:

```bash
mount -o loop /root/RHEL-10.iso /mnt
```

---

## A BaseOS vagy AppStream könyvtár hiányzik

Ellenőrzés:

```bash
ls -l /mnt
```

Repository metaadatok ellenőrzése:

```bash
ls /mnt/BaseOS/repodata
```

```bash
ls /mnt/AppStream/repodata
```

Ha ezek nem léteznek, valószínűleg nem a teljes DVD ISO lett csatolva.

---

## A repository nem jelenik meg

A konfiguráció ellenőrzése:

```bash
cat /etc/yum.repos.d/local.repo
```

Ezután:

```bash
dnf clean all
dnf makecache
dnf repolist
```

---

## Csak a saját repositoryk tesztelése

```bash
dnf --disablerepo="*" --enablerepo="BaseOS,AppStream" repolist
```

Csomaglista tesztelése:

```bash
dnf --disablerepo="*" --enablerepo="BaseOS,AppStream" list available
```

---

# Fontos különbség

## Virtuális DVD-meghajtó

```bash
mount /dev/sr0 /mnt
```

## ISO-fájl

```bash
mount -o loop /root/RHEL-10.iso /mnt
```

## Hálózati repository

```ini
baseurl=http://server.example.com/rhel10/BaseOS
```

---

# Rövid vizsgamegoldás

## Virtuális DVD esetén

```bash
mkdir -p /mnt
mount /dev/sr0 /mnt
vim /etc/yum.repos.d/local.repo
dnf clean all
dnf repolist
```

## ISO-fájl esetén

```bash
mkdir -p /mnt
mount -o loop /root/RHEL-10.iso /mnt
vim /etc/yum.repos.d/local.repo
dnf clean all
dnf repolist
```

## Hálózati repository esetén

```bash
vim /etc/yum.repos.d/local.repo
dnf clean all
dnf repolist
```

---

# Összefoglalás

A repository forrását mindig a feladat határozza meg.

- Virtuális DVD esetén a `/dev/sr0` eszközt kell csatolni.
- ISO-fájl esetén a `mount -o loop` parancsot kell használni.
- Hálózati repository esetén a megadott URL-t közvetlenül a repo-fájlba kell írni.

A végső ellenőrzés minden esetben:

```bash
dnf clean all
dnf repolist
dnf makecache
```
