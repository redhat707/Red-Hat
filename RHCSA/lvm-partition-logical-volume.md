# 6. feladat – LVM logikai kötet létrehozása és bővítése

## Feladat

A `/dev/sdb` lemezen hozz létre egy `1 GiB` méretű partíciót.

A partícióból készíts:

- fizikai kötetet: **PV**
- kötetcsoportot: **VG**
- logikai kötetet: **LV**

A beállítások:

```text
Lemez:             /dev/sdb
Partíció:          /dev/sdb1
Volume Group neve: myvg
Logical Volume:    mylv
Kezdeti LV-méret:  500 MiB
Fájlrendszer:      ext4
Csatolási pont:    /mylv
Bővítés:           +500 MiB
```

A fájlrendszer újraindítás után is automatikusan legyen csatolva.

> **Figyelem:** a particionálás adatvesztést okozhat. Vizsgán mindig ellenőrizd, hogy valóban a feladatban megadott lemezen dolgozol.

---

# Megoldás

## 1. A rendelkezésre álló lemezek ellenőrzése

```bash
lsblk
```

Részletesebb ellenőrzés:

```bash
fdisk -l
```

A feladatban használt lemez:

```text
/dev/sdb
```

Ellenőrizzük, hogy nincs-e rajta fontos adat vagy csatolt fájlrendszer:

```bash
lsblk -f /dev/sdb
```

---

# Partíció létrehozása

## 2. A lemez megnyitása az `fdisk` programmal

```bash
fdisk /dev/sdb
```

Az interaktív parancsok:

```text
n
p
Enter
Enter
+1G
w
```

Jelentésük:

```text
n       új partíció létrehozása
p       elsődleges partíció
Enter   alapértelmezett partíciószám
Enter   alapértelmezett kezdőszektor
+1G     1 GiB méret
w       változtatások mentése
```

Modern rendszeren előfordulhat, hogy az `fdisk` nem kér külön `p` választást. Ilyenkor az alapértelmezett értékeket kell elfogadni.

---

## 3. Az új partíciós tábla beolvastatása

```bash
partprobe /dev/sdb
```

Ellenőrzés:

```bash
lsblk /dev/sdb
```

A kimenetben meg kell jelennie:

```text
sdb
└─sdb1
```

Ha a `/dev/sdb1` nem jelenik meg azonnal:

```bash
udevadm settle
```

Majd:

```bash
lsblk
```

---

# Physical Volume létrehozása

## 4. A partíció inicializálása LVM fizikai kötetként

```bash
pvcreate /dev/sdb1
```

Ellenőrzés:

```bash
pvs
```

vagy:

```bash
pvdisplay /dev/sdb1
```

---

# Volume Group létrehozása

## 5. A `myvg` kötetcsoport létrehozása

```bash
vgcreate myvg /dev/sdb1
```

Ellenőrzés:

```bash
vgs
```

vagy:

```bash
vgdisplay myvg
```

A kötetcsoport neve:

```text
myvg
```

---

# Logical Volume létrehozása

## 6. Az 500 MiB méretű logikai kötet létrehozása

```bash
lvcreate -n mylv -L 500M myvg
```

A parancs jelentése:

```text
-n mylv    a logikai kötet neve
-L 500M    a logikai kötet mérete
myvg       a kötetcsoport neve
```

Ellenőrzés:

```bash
lvs
```

vagy:

```bash
lvdisplay /dev/myvg/mylv
```

A logikai kötet elérési útja:

```text
/dev/myvg/mylv
```

Ugyanez device mapper formában:

```text
/dev/mapper/myvg-mylv
```

---

# Fájlrendszer létrehozása

## 7. Ext4 fájlrendszer létrehozása

```bash
mkfs.ext4 /dev/myvg/mylv
```

Ellenőrzés:

```bash
lsblk -f
```

A fájlrendszer típusának ennek kell lennie:

```text
ext4
```

---

# Csatolási pont létrehozása

## 8. A `/mylv` könyvtár létrehozása

```bash
mkdir -p /mylv
```

---

# Tartós csatolás beállítása

## 9. A logikai kötet UUID-jának lekérdezése

```bash
blkid /dev/myvg/mylv
```

Példa:

```text
/dev/mapper/myvg-mylv: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="ext4"
```

Az UUID külön lekérdezése:

```bash
blkid -s UUID -o value /dev/myvg/mylv
```

---

## 10. Az `/etc/fstab` fájl szerkesztése

```bash
vim /etc/fstab
```

Ajánlott UUID-alapú bejegyzés:

```fstab
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mylv ext4 defaults 0 0
```

Az `xxxxxxxx-...` helyére a saját `blkid` kimenetedben szereplő UUID-t kell írni.

Használható az eszköz elérési útja is:

```fstab
/dev/myvg/mylv /mylv ext4 defaults 0 0
```

Az UUID használata azonban megbízhatóbb.

---

## 11. Az `/etc/fstab` ellenőrzése

Először töltsük újra a systemd konfigurációját:

```bash
systemctl daemon-reload
```

Ezután:

```bash
mount -a
```

Ha nincs hibaüzenet, ellenőrizzük:

```bash
findmnt /mylv
```

vagy:

```bash
df -hT /mylv
```

A fájlrendszernek a `/mylv` könyvtárba kell csatolódnia.

---

# A logikai kötet és a fájlrendszer bővítése

## 12. A szabad hely ellenőrzése

```bash
vgs
```

Részletesen:

```bash
vgdisplay myvg
```

A `VFree` vagy `Free PE / Size` értéknek legalább körülbelül `500 MiB` szabad helyet kell mutatnia.

---

## 13. A logikai kötet bővítése 500 MiB-tal

```bash
lvextend -r -L +500M /dev/myvg/mylv
```

A parancs jelentése:

```text
lvextend    a logikai kötet bővítése
-r          a fájlrendszer automatikus bővítése
-L +500M    további 500 MiB hozzáadása
```

A `+` jel nagyon fontos.

Helyes:

```bash
-L +500M
```

Ez hozzáad 500 MiB-ot a jelenlegi mérethez.

A `+` jel nélkül:

```bash
-L 500M
```

a parancs a teljes méretet próbálná 500 MiB-ra állítani, nem pedig további 500 MiB-ot hozzáadni.

---

# Ellenőrzés

## 14. Az LVM méretének ellenőrzése

```bash
lvs
```

Részletesebben:

```bash
lvdisplay /dev/myvg/mylv
```

A logikai kötet végső mérete körülbelül:

```text
1 GiB
```

---

## 15. A fájlrendszer méretének ellenőrzése

```bash
df -hT /mylv
```

vagy:

```bash
lsblk -f
```

A fájlrendszernek is körülbelül `1 GiB` méretűnek kell lennie.

---

## 16. A tartós csatolás ellenőrzése

```bash
cat /etc/fstab
```

```bash
findmnt /mylv
```

Újraindítás után:

```bash
reboot
```

Majd:

```bash
findmnt /mylv
```

```bash
df -hT /mylv
```

Ha a beállítás helyes, a fájlrendszer automatikusan felcsatolódik.

---

# Részletes magyarázat

## Physical Volume – PV

A Physical Volume az LVM számára előkészített fizikai lemez vagy partíció.

Létrehozása:

```bash
pvcreate /dev/sdb1
```

Ellenőrzése:

```bash
pvs
```

---

## Volume Group – VG

A Volume Group egy vagy több fizikai kötetből létrehozott tárolóterület.

Létrehozása:

```bash
vgcreate myvg /dev/sdb1
```

Ellenőrzése:

```bash
vgs
```

---

## Logical Volume – LV

A Logical Volume a Volume Group szabad területéből létrehozott logikai tároló.

Létrehozása:

```bash
lvcreate -n mylv -L 500M myvg
```

Ellenőrzése:

```bash
lvs
```

---

## Az LVM felépítése

```text
/dev/sdb
   └── /dev/sdb1
          └── PV
               └── VG: myvg
                      └── LV: mylv
                             └── ext4
                                    └── /mylv
```

---

# Alternatív bővítési módszer

A logikai kötet külön is bővíthető:

```bash
lvextend -L +500M /dev/myvg/mylv
```

Ezután az ext4 fájlrendszert külön kell megnövelni:

```bash
resize2fs /dev/myvg/mylv
```

A két művelet egyetlen paranccsal:

```bash
lvextend -r -L +500M /dev/myvg/mylv
```

Vizsgán ez a gyorsabb és biztonságosabb megoldás.

---

# Hibakeresés

## A `/dev/sdb` nem a megfelelő lemez

Mindig ellenőrizd:

```bash
lsblk
```

```bash
fdisk -l
```

Soha ne dolgozz automatikusan a `/dev/sdb` lemezen csak azért, mert a példában ez szerepel. A vizsgán a feladatban megadott lemezt kell használni.

---

## A `/dev/sdb1` nem jelenik meg

```bash
partprobe /dev/sdb
```

```bash
udevadm settle
```

```bash
lsblk
```

---

## A fizikai kötet már létezik

Ellenőrzés:

```bash
pvs
```

Ha a `/dev/sdb1` már egy PV része, nem kell újra létrehozni.

---

## Nincs elég szabad hely a bővítéshez

Ellenőrzés:

```bash
vgs
```

```bash
vgdisplay myvg
```

Ha nincs legalább 500 MiB szabad hely a Volume Groupban, a bővítés nem hajtható végre.

---

## A `mount -a` hibát jelez

Ellenőrizzük az `/etc/fstab` fájlt:

```bash
cat /etc/fstab
```

Az UUID ellenőrzése:

```bash
blkid /dev/myvg/mylv
```

A csatolási pont ellenőrzése:

```bash
ls -ld /mylv
```

Rendszernapló:

```bash
journalctl -b
```

Az `/etc/fstab` hibáit mindig újraindítás előtt javítsuk ki.

---

## A logikai kötet bővült, de a fájlrendszer nem

Ellenőrzés:

```bash
lvs
```

```bash
df -hT /mylv
```

Ext4 fájlrendszer kézi bővítése:

```bash
resize2fs /dev/myvg/mylv
```

Ha az `lvextend -r` sikeresen lefutott, erre külön nincs szükség.

---

# Rövid vizsgamegoldás

## Partíció létrehozása

```bash
fdisk /dev/sdb
```

```text
n
p
Enter
Enter
+1G
w
```

```bash
partprobe /dev/sdb
```

## LVM létrehozása

```bash
pvcreate /dev/sdb1
vgcreate myvg /dev/sdb1
lvcreate -n mylv -L 500M myvg
```

## Fájlrendszer létrehozása

```bash
mkfs.ext4 /dev/myvg/mylv
mkdir -p /mylv
```

## Tartós csatolás

```bash
blkid /dev/myvg/mylv
vim /etc/fstab
```

Az `/etc/fstab` bejegyzése:

```fstab
UUID=SAJÁT-UUID /mylv ext4 defaults 0 0
```

Ezután:

```bash
systemctl daemon-reload
mount -a
```

## Bővítés

```bash
lvextend -r -L +500M /dev/myvg/mylv
```

## Ellenőrzés

```bash
pvs
vgs
lvs
findmnt /mylv
df -hT /mylv
```

---

# Egyszerű, eszköznév-alapú változat

```bash
fdisk /dev/sdb
partprobe /dev/sdb
pvcreate /dev/sdb1
vgcreate myvg /dev/sdb1
lvcreate -n mylv -L 500M myvg
mkfs.ext4 /dev/myvg/mylv
mkdir -p /mylv
echo '/dev/myvg/mylv /mylv ext4 defaults 0 0' >> /etc/fstab
systemctl daemon-reload
mount -a
lvextend -r -L +500M /dev/myvg/mylv
```

Ellenőrzés:

```bash
lsblk
pvs
vgs
lvs
df -hT /mylv
```

---

# Összefoglalás

A `/dev/sdb1` partícióból létrehoztuk a fizikai kötetet:

```bash
pvcreate /dev/sdb1
```

Létrehoztuk a `myvg` kötetcsoportot:

```bash
vgcreate myvg /dev/sdb1
```

Létrehoztuk az 500 MiB méretű `mylv` logikai kötetet:

```bash
lvcreate -n mylv -L 500M myvg
```

Ext4 fájlrendszert készítettünk rajta, majd tartósan a `/mylv` könyvtárhoz csatoltuk.

Végül a logikai kötetet és a fájlrendszert további 500 MiB-tal bővítettük:

```bash
lvextend -r -L +500M /dev/myvg/mylv
```

A végeredmény egy körülbelül 1 GiB méretű, újraindítás után is automatikusan csatolt ext4 fájlrendszer.
