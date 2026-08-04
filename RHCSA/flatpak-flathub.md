# 5. feladat – Flatpak alkalmazás telepítése Flathub repositoryból

## Feladat

Add hozzá a Flathub távoli repositoryt a Flatpakhoz, majd telepítsd a GNOME Text Editor alkalmazást.

Az alkalmazás Flatpak-azonosítója:

```text
org.gnome.TextEditor
```

A beállítás és az alkalmazás újraindítás után is maradjon elérhető.

---

# Megoldás

## 1. A Flatpak telepítésének ellenőrzése

```bash
rpm -q flatpak
```

Ha a Flatpak nincs telepítve:

```bash
dnf install -y flatpak
```

Ellenőrzés:

```bash
flatpak --version
```

---

## 2. A Flathub repository hozzáadása

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

A `--if-not-exists` megakadályozza, hogy a parancs hibát adjon, ha a Flathub repository már létezik.

---

## 3. A repository ellenőrzése

```bash
flatpak remotes
```

A listában meg kell jelennie:

```text
flathub
```

Részletes ellenőrzés:

```bash
flatpak remotes --show-details
```

---

## 4. Az alkalmazás megkeresése

```bash
flatpak search Text Editor
```

Az alkalmazás azonosítója:

```text
org.gnome.TextEditor
```

---

## 5. Az alkalmazás telepítése

```bash
flatpak install -y flathub org.gnome.TextEditor
```

A parancs jelentése:

- `install` – alkalmazás telepítése;
- `-y` – automatikusan elfogadja a kérdéseket;
- `flathub` – a használandó repository;
- `org.gnome.TextEditor` – az alkalmazás azonosítója.

---

# Ellenőrzés

## A telepített Flatpak alkalmazások listázása

```bash
flatpak list
```

Csak az alkalmazások megjelenítése:

```bash
flatpak list --app
```

A listában meg kell jelennie:

```text
org.gnome.TextEditor
```

---

## Egy adott alkalmazás ellenőrzése

```bash
flatpak info org.gnome.TextEditor
```

A parancs megmutatja többek között:

- az alkalmazás nevét;
- az alkalmazás azonosítóját;
- a telepített verziót;
- az architektúrát;
- a telepítés helyét;
- a használt repositoryt.

---

## Az alkalmazás elindítása

```bash
flatpak run org.gnome.TextEditor
```

Grafikus környezet nélkül az alkalmazás telepíthető, de a grafikus felülete nem feltétlenül indítható el.

---

# Rendszerszintű és felhasználói telepítés

## Rendszerszintű telepítés

Root felhasználóként a Flatpak általában rendszerszinten telepíti az alkalmazást.

Egyértelműen megadható a `--system` opcióval:

```bash
flatpak remote-add --system --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

```bash
flatpak install --system -y flathub org.gnome.TextEditor
```

A rendszerszintű alkalmazást minden felhasználó használhatja.

---

## Felhasználói telepítés

Csak az aktuális felhasználó számára:

```bash
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

```bash
flatpak install --user -y flathub org.gnome.TextEditor
```

Vizsgán mindig a feladat megfogalmazását kell követni.

Ha nincs külön megadva, rootként a rendszerszintű telepítés a legegyértelműbb megoldás.

---

# Megmarad-e újraindítás után?

Igen.

A hozzáadott Flathub repository és a telepített alkalmazás automatikusan megmarad újraindítás után.

Nem szükséges:

- `/etc/fstab` bejegyzés;
- külön systemd szolgáltatás;
- automatikus indítás beállítása.

Rendszerszintű Flatpak-adatok jellemző helye:

```text
/var/lib/flatpak/
```

Felhasználói Flatpak-adatok jellemző helye:

```text
~/.local/share/flatpak/
```

Újraindítás után ellenőrizhető:

```bash
flatpak remotes
```

```bash
flatpak list
```

```bash
flatpak info org.gnome.TextEditor
```

---

# Részletes magyarázat

## Flatpak

A Flatpak alkalmazásokat elkülönített, úgynevezett sandbox környezetben futtatja.

Ez csökkenti annak szükségességét, hogy az alkalmazás közvetlenül módosítsa az alapvető rendszerfájlokat.

---

## Flathub

A Flathub egy Flatpak alkalmazásokat tartalmazó távoli repository.

A hozzáadása:

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

---

## Flatpak alkalmazásazonosító

A Flatpak alkalmazások fordított DNS-formátumú azonosítót használnak.

Példa:

```text
org.gnome.TextEditor
```

Felépítése:

```text
org.gnome.AlkalmazásNeve
```

---

# Hibakeresés

## A `flatpak` parancs nem található

Ellenőrzés:

```bash
rpm -q flatpak
```

Telepítés:

```bash
dnf install -y flatpak
```

---

## A Flathub repository már létezik

Ellenőrzés:

```bash
flatpak remotes
```

A következő parancs nem okoz hibát akkor sem, ha már létezik:

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

---

## A repository nem jelenik meg

Ellenőrzés:

```bash
flatpak remotes --show-details
```

Rendszerszintű repositoryk:

```bash
flatpak remotes --system
```

Felhasználói repositoryk:

```bash
flatpak remotes --user
```

Előfordulhat, hogy a repositoryt másik telepítési szinthez adtuk hozzá.

---

## Az alkalmazás nem található

Az alkalmazás keresése:

```bash
flatpak search TextEditor
```

vagy:

```bash
flatpak search "Text Editor"
```

A Flathub frissítése:

```bash
flatpak update
```

Ezután újra:

```bash
flatpak install -y flathub org.gnome.TextEditor
```

---

## Nincs hálózati kapcsolat

Kapcsolat ellenőrzése:

```bash
ping -c 3 dl.flathub.org
```

DNS-feloldás ellenőrzése:

```bash
getent hosts dl.flathub.org
```

HTTPS-kapcsolat ellenőrzése:

```bash
curl -I https://dl.flathub.org
```

---

## A telepítéshez nincs jogosultság

Rendszerszintű telepítés rootként:

```bash
sudo flatpak install --system -y flathub org.gnome.TextEditor
```

Felhasználói telepítéshez:

```bash
flatpak install --user -y flathub org.gnome.TextEditor
```

---

## Az alkalmazás telepítve van, de nem indul

Ellenőrzés:

```bash
flatpak info org.gnome.TextEditor
```

Indítás terminálból:

```bash
flatpak run org.gnome.TextEditor
```

Ha a rendszernek nincs grafikus környezete, a grafikus alkalmazás nem tud ablakot megjeleníteni.

A telepítés ettől még sikeres lehet.

---

# Az alkalmazás frissítése

Minden Flatpak alkalmazás frissítése:

```bash
flatpak update -y
```

Csak a Text Editor frissítése:

```bash
flatpak update -y org.gnome.TextEditor
```

---

# Az alkalmazás eltávolítása

```bash
flatpak uninstall -y org.gnome.TextEditor
```

A már nem használt futtatókörnyezetek eltávolítása:

```bash
flatpak uninstall --unused -y
```

---

# Rövid vizsgamegoldás

## Flatpak telepítése

```bash
dnf install -y flatpak
```

## Flathub hozzáadása

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## Alkalmazás telepítése

```bash
flatpak install -y flathub org.gnome.TextEditor
```

## Ellenőrzés

```bash
flatpak remotes
```

```bash
flatpak list
```

```bash
flatpak info org.gnome.TextEditor
```

---

# Egysoros megoldás

```bash
dnf install -y flatpak && flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo && flatpak install -y flathub org.gnome.TextEditor
```

Ellenőrzés:

```bash
flatpak list
```

---

# Összefoglalás

A Flathub repositoryt ezzel adtuk hozzá:

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

A GNOME Text Editor alkalmazást ezzel telepítettük:

```bash
flatpak install -y flathub org.gnome.TextEditor
```

A telepítést ezzel ellenőriztük:

```bash
flatpak list
```

A Flathub repository és a telepített alkalmazás újraindítás után is megmarad.
