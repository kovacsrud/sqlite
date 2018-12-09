# Adatbázisok kezelése Sqlite-al és C#-al
---
## Adatbázis létrehozása az SqliteStudio-val

Egyszerű egy táblából álló adatbázist készítünk.

### A feladatok:
- [ ] Új adatbázis létrehozása
- [ ] Adattábla létrehozása
- [ ] Néhány rekord felvitele, hogy legyen mit lekérdezni
---
#### Az új adatbázis létrehozása

**Database->Add a database**
**Database Type**:Sqlite3
**File**: A zöld **+**-on kell kattintani mappát választani, adabázis fájl nevét megadni.
![Minta 1](sqlite_demo_2.png)

#### Adattábla létrehozása
Az adattábla a következő oszlopokból fog állni:
+ Id ( A tanuló tábla elsődleges kulcsa)
+ VezetekNev
+ KeresztNev
+ AnyjaNeve
+ SzuletesEve
+ SzuletesiHely

A **Tables**-on jobb gomb, majd **Create table**

**Table name**: tanulok

Kattintani az **Add column** gombra

**Column name**:Id

**Data type**:integer

Bejelölni a továbbiakat:

- [x] Primary Key
- [x] Not NULL

A Primary Key mellett a **Configure** gombra kattintani, majd itt bejelölni

- [x] Autoincrement
- [x] On Conflict :Abort

![Tábla létrehozás](table_create_1.png)

A következő mező a **VezetekNev**

Kattintani az **Add column** gombra

**Column name**:VezetekNev

**Data type**:varchar, **Size**:50

Bejelölni a továbbiakat:

- [x] Not NULL

A többi szöveget tartalmazó mező beállítása ugyanaz.

A **SzuletesEve** mező integer legyen.

Ha megvan akkor **Commit structure changes**-gombra kattintva a tábla létrejön.

![Commit](table_create_2.png)

Megnézhetjük, hogy milyen SQL-utasítást készít a program:

![SQL](table_create_3.png)

#### Rekordok felvitele

A **Tanulok** táblán jobb gomb, majd **Generate query for table** **INSERT**

Kapunk egy nyers INSERT Sql utasítást, csak kicsit át kell írni.
Az **Id**-t kivesszük, hiszen **Auto Increment**-et használunk. A többi értelemszerű.

![Insert](table_create_4.png)

Készüljön 4-5 rekord!
















