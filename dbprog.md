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

# Az előbbi adatbázis kezelése C#-ból

## Új solution: TanulokDb

Készítsünk egy új Wpf solution-t.

Ahhoz, hogy egy Sqlite adatbázist használni tudjunk a programból, pár dolgot telepíteni kell.

A projekten/solution-on jobb gomb, majd **Manage Packages**

A keresőbe beírni: **Sqlite**, és a **System.Data.Sqlite(x86/x64)**-et választani.

![sqliteinstall](sqlite_install.png)

## A feladatok
- [ ] Valamilyen felület kreálása, adatlekérdezés, és betöltés valamilyen Control-ba (célszerű egy DataGrid-et csinálni)
- [ ] Adatfelvitel megvalósítása
- [ ] Adat módosításának megvalósítása

### Egyszerű select lekérdezés kódolása

Először is, nem lesz annyira egyszerű :) Nem maga a lekérdezés kivitelezése probléma, hanem az ad munkát, hogy a lekérdezés eredményét átömlesszük valami olyan adatszerkezetbe, amelyet a DataGrid meg tud jeleníteni.

#### Először készítünk egy Db nevű osztályt, nem kódolunk bele az ablakba.

Most is érdemes Dependency Injection-el átadni a Db osztálynak az ablakot, így könnyen elérhető majd a Grid, vagy valami más elem.

Adjunk hozzá a projekthet egy új osztályt **New Item-Class**. A neve legyen **Db**

A kiindulási állapot:

```C#
using System;
using System.Data.SQLite;
using System.Collections.Generic;
using System.Linq;
using System.Windows.Media;
using System.Windows.Controls;
using System.Windows;
using System.Diagnostics;
using System.Data;

namespace TanulokDb
{
	/// <summary>
	/// Description of Db.
	/// </summary>
	public class Db
	{	
		public Db()
		{
						
		}
		
	}
}
```
Először gondoskodni kell az ablak átadásáról, a Db osztályban deklaráljuk ezt:

```C#
public class Db
	{
		Window1 window1;
		
		public Db(Window1 window)
		{
			this.window1=window;
			
		}
	}	
```
Az ablak függvényébe pedig ez kell:

```C#
public partial class Window1 : Window
	{
		Db db;
		public Window1()
		{
			InitializeComponent();
			db=new Db(this);
			
		}
	}
```
Így már elérhető lesz az ablak.

Hozzunk létre az XAML-ben egy DataGrid-et. Hagyjuk az automatikus oszlopgenerálást.

```XAML
<Window x:Class="TanulokDb.Window1"
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	Title="TanulokDb" Height="500" Width="300"
	>
	<Grid>
		<DataGrid x:Name="adatok" AutoGenerateColumns="True" ColumnWidth="*" />
	</Grid>
</Window>
```
Az adatok lekérdezéséhez h Db osztályunkba létre kell hozni egy új metódust, DbLekerdez() néven. 
Minden ami a lekérdezéshez kell ebbe kerül.

```C#
public void DbLekerdez()
	{
		
	}
```
Az első lépés az adatbázishoz való kapcsolódás deklarálása:

```C#
SQLiteConnection db_connect=new SQLiteConnection("Data Source=g:\\adatok\\szoftver_tanf_peldak\\tanulo_nyilvantartas\\tanulo_v1.db;Version=3;");
	db_connect.Open();
```
Ezt követően adjuk meg az elvégzendő műveletet:

```C#
	string sql="select * from Tanulok";
	SQLiteCommand comm=new SQLiteCommand(sql,db_connect);
	var eredm=comm.ExecuteReader();
```
A lekérdezett adatok az _**eredm**_ nevű eredményhalmazba kerülnek, ez a megjelenítéshez további feldolgozást igényel.

Az adatokból adattáblát készítünk, ez egyszerűbb, mintha osztályt készítenénk az adatokhoz, és azt példányosítgatnánk.

```C#
DataTable tanulok=new DataTable();
			
	tanulok.Columns.Add("Vezetéknév");
	tanulok.Columns.Add("Keresztnév");
	tanulok.Columns.Add("Anyja neve");
	tanulok.Columns.Add("Születési idő");
	tanulok.Columns.Add("Születési hely");
			
```
Ezt követően egy ciklussal végig kell olvasni az eredményhalmazt. A cikluson belül egy DataRow osztályú adatsort hozunk létre, ezt adjuk hozzá a DataTable-hoz.

```C#
while (eredm.Read()) {
	DataRow ujsor=tanulok.NewRow();
			
	ujsor["Vezetéknév"]=eredm["VezetekNev"];
	ujsor["Keresztnév"]=eredm["KeresztNev"];
	ujsor["Anyja neve"]=eredm["AnyjaNeve"];
	ujsor["Születési idő"]=eredm["SzuletesEve"];
	ujsor["Születési hely"]=eredm["SzuletesiHely"];
			
	tanulok.Rows.Add(ujsor);
								
				
}
```
A kész DataTable-t már tudja fogadni a DataGrid, majd zárjuk az adatbázis kapcsolatot.

```C#
	window1.adatok.ItemsSource=tanulok.DefaultView;
		
	db_connect.Close();
```
### Adatfelvitel megvalósítása (INSERT INTO)

Az adatfelvitel megvalósításához módosítani kell a program felületét, hiszen a gridünk erre most nem használható. Szerencsére az XAML lehetőséget ad arra, hogy könnyen módosítsuk a felületet. Valami olyan kellene, amelynek segítségével több egymástól független lapot is meg lehet jeleníteni. Erre tökéletesen jó lesz a **TabControl**. Ezzel tetszőleges számú lap készíthető, és a megfelelőre tudjuk helyezni az összetartozó funkciókat, elemeket.

Módosítsuk az XAML-t

```XAML
<Window x:Class="TanulokDb.Window1"
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	Title="TanulokDb" Height="500" Width="300"
	>
	<TabControl>
	<TabItem Header="Lekérdezés">
	<Grid>
		<DataGrid x:Name="adatok" AutoGenerateColumns="True" ColumnWidth="*" />
	</Grid>
	</TabItem>
	</TabControl>
</Window>
```

Ezzel a lekérdezésünk egy külön lapra került. További **TabItem**-ek használatával további lapokat adhatunk a felülethez.

A **Lekérdezés** fül **</TabItem>** eleme után jön a következő, ez lesz az adatfelvitelé.

```XAML
<TabItem Header="Új tanuló felvitel">
<Grid>
</Grid>
</TabItem>
```
Az adatfelvitelhez szükség lesz **Label** illetve **TextBox** elemekre. Mindezt valamibe bele kellene rendezni, erre egy **StackPanel** megfelelő lesz.

A **Grid**-en belülre:

```XAML
<StackPanel HorizontalAlignment="Left">
	<Label Content="Vezetéknév"/>
	<TextBox x:Name="vezeteknev" MinWidth="200" MaxWidth="200" MaxLength="50"/>
	...
	<Button Content="Felvesz" x:Name="ButtonUjAdat" />
</StackPanel>	
```
A Db osztályban folytatjuk, készítünk egy új metódust, mondjuk **DbUjAdat** néven. Azt tervezzük,hogy paraméterként átadjuk a metódusnak azokat az adatokat, amelyeket aztán az **INSERT INTO** parancs majd felvisz mint új rekordot.

```C#
public void DbUjadat(string vezeteknev,string keresztnev,string anyjaneve,int szuleteseve,string szuletesihely)
	{
		//Ide jönnek a parancsok
	}
```
Kapcsolódás az adatbázishoz:
```C#
//Az adatbázis elérését nyilván az adott környezethez kell igazítani
SQLiteConnection db_connect=new SQLiteConnection("Data Source=d:\\adatok\\c#\\tanulo_v1.db;Version=3;");
db_connect.Open();
```
Megépítjük az SQL parancsot (A sortörés csak a jobb olvashatóság miatt kell) 
```C#
comm.CommandText="insert into Tanulok" +
 "(VezetekNev,KeresztNev,AnyjaNeve,SzuletesEve,SzuletesiHely)" +
 "values(@vezeteknev,@keresztnev,@anyjaneve,@szuleteseve,@szuletesihely)";
```
Felmerül a kérdés, hogy mik azok a @keresznev ... stb? Ezek az SQL parancs **paraméterei**. Lehet úgy is sql parancsot építeni, hogy stringeket fűzünk össze, csak tilos :) Nem, nem tilos, csak sebezhetőséget visz a programba (SQL injection), mert ha valaki ezt a string-et kívülről manipulálni tudja esetleg, akkor mi van ha beletesz egy **DELETE** parancsot? A paraméterek használatával ez elkerülhető.

Hozzáadjuk a paramétereket:

```C#
comm.Parameters.Add("@vezeteknev",DbType.String).Value=vezeteknev;
comm.Parameters.Add("@keresztnev",DbType.String).Value=keresztnev;
comm.Parameters.Add("@anyjaneve",DbType.String).Value=anyjaneve;
comm.Parameters.Add("@szuleteseve",DbType.Int32).Value=szuleteseve;
comm.Parameters.Add("@szuletesihely",DbType.String).Value=szuletesihely;
```
Majd a végrehajtás és a kapcsolat zárása:

```C#
comm.ExecuteNonQuery();		
db_connect.Close();
```
Ezzel ez a metódus elkészült, de a meghívásáról gondoskodni kell.

A gomb már megvan ugyan, de közvetlenül nem férünk hozzá. Először meg kell írni az eseménykezelő függvényt.

```C#
public void DbUjAdatRogzit(object sender, RoutedEventArgs e)
	{
		
	}
```
Ebből a függvényből hívjuk meg az adatokat rögzítő metódust a megfelelő paraméterekkel:

```C#
public void DbUjAdatRogzit(object sender, RoutedEventArgs e)
{

DbUjadat(window1.vezeteknev.Text,window1.keresztnev.Text,window1.anyjaneve.Text,Convert.ToInt32(window1.szuleteseve.Text),window1.szuletesihely.Text);
}
```
Megvan az eseménykezelő, de ezt a függvényt még fel kell "iratkoztatni" a gomb **Click** eseményére, ezt a Db osztály konstruktorában kell(célszerű) megtenni.

```C#
public Db(Window1 window)
	{
	this.window1=window;
	window1.ButtonUjAdat.Click+=DbUjAdatRogzit;	
	}
```
Ezek után már mennie kell az új rekord felvételének, de tennivaló akad még (pl. hibakezelések).





