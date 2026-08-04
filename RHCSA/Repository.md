# 2. feladat – Helyi RHEL 10 repository létrehozása ISO-fájlból

## Feladat

Csatold fel a RHEL 10 telepítő ISO-fájlt a `/mnt` könyvtárba.

Ezután hozz létre két helyi DNF repositoryt:

- BaseOS
- AppStream

A repositoryk legyenek engedélyezve, és ne használjanak GPG-ellenőrzést.

---

## Megoldás

### 1. Az ISO-fájl csatolása

```bash
mount -o loop RHEL-10.iso /mnt
```

A `mount -o loop` lehetővé teszi, hogy az ISO-fájlt úgy csatoljuk fel, mintha fizikai lemez lenne.

---

### 2. Ellenőrizzük az ISO tartalmát

```bash
ls /mnt
```

A következő könyvtáraknak kell megjelenniük:

```text
AppStream
BaseOS
```

---

### 3. Repository konfigurációs fájl létrehozása

```bash
vim /etc/yum.repos.d/local.repo
```

A fájl tartalma:

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

---

### 4. A DNF gyorsítótár törlése

```bash
dnf clean all
```

---

### 5. A repositoryk ellenőrzése

```bash
dnf repolist
```

A listában meg kell jelenniük a következő repositoryknak:

```text
BaseOS      RHEL 10 BaseOS
AppStream   RHEL 10 AppStream
```

---

## További ellenőrzés

A BaseOS repository részletes ellenőrzése:

```bash
dnf repoinfo BaseOS
```

Az AppStream repository részletes ellenőrzése:

```bash
dnf repoinfo AppStream
```

Megnézhetjük az elérhető csomagokat is:

```bash
dnf list available
```

---

## A repository fájl magyarázata

```ini
[BaseOS]
```

Ez a repository egyedi azonosítója.

```ini
name=RHEL 10 BaseOS
```

A repository megjelenő neve.

```ini
baseurl=file:///mnt/BaseOS
```

Megadja, hogy a csomagok a helyi fájlrendszeren találhatók.

A `file:///` három perjelet tartalmaz:

- `file://` a protokoll
- `/mnt/BaseOS` az abszolút elérési út

```ini
enabled=1
```

A repository engedélyezve van.

```ini
gpgcheck=0
```

A GPG-aláírás ellenőrzése ki van kapcsolva.

---

## Hibakeresés

### Az ISO nincs felcsatolva

Ellenőrzés:

```bash
mount | grep /mnt
```

Újracsatolás:

```bash
mount -o loop RHEL-10.iso /mnt
```

---

### A BaseOS vagy AppStream könyvtár nem található

Ellenőrzés:

```bash
ls -l /mnt
```

Valószínűleg nem a megfelelő RHEL DVD ISO lett felcsatolva.

---

### A repository nem jelenik meg

Ellenőrizzük a konfigurációs fájlt:

```bash
cat /etc/yum.repos.d/local.repo
```

Majd:

```bash
dnf clean all
dnf repolist
```

---

### A repository elérési útja hibás

Ellenőrizzük, hogy léteznek-e a repository adatai:

```bash
ls /mnt/BaseOS/repodata
ls /mnt/AppStream/repodata
```

---

## Fontos parancsok

```bash
mount -o loop RHEL-10.iso /mnt
vim /etc/yum.repos.d/local.repo
dnf clean all
dnf repolist
dnf repoinfo BaseOS
dnf repoinfo AppStream
```

---

## Rövid összefoglaló

A RHEL 10 ISO-fájlt a `/mnt` könyvtárba csatoltuk:

```bash
mount -o loop RHEL-10.iso /mnt
```

Ezután létrehoztuk a következő repository fájlt:

```text
/etc/yum.repos.d/local.repo
```

A fájlban beállítottuk a helyi BaseOS és AppStream repositorykat, majd a következő paranccsal ellenőriztük őket:

```bash
dnf clean all
dnf repolist
```
