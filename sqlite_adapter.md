# SqLiteDataAdapter készítése

Az előző példa kódolása során több metódussal kezeltük az adatbázis-kezelés egyes feladatait (SELECT,UPDATE).
Felmerül a kérdés, hogy nem lehet ezt egyszerűbben? Lehet egyszerűbben, a megoldás a DataAdapter.

## A grafikus felület

A felület most is egy DataGrid lesz, amely automatikusan jeleníti meg az **ItemsSource**-nál megkapott adatokat. Elhelyezünk még egy gombot is, ezzel frissítjük az adatbázist. Ha vittünk fel új rekordot, akkor beíródik, ha szerkesztettünk egy meglévőt, akkor az módosul, stb.

```XAML
<Grid>
	<StackPanel>
		<DataGrid
			x:Name="adatok"
			AutoGenerateColumns="True"
			ColumnWidth="*" />
		<Button
			x:Name="ButtonUpdate"
			Content="Update"
			Click="ButtonUpdate_Click" />
	</StackPanel>
</Grid>
```

## Adapter osztály

Az adapter osztály arra van, hogy benne egy helyen tudjuk kezelni az összes felmerülő feladatot. Ezeket a **CRUD** jelöléssel is 
szokták illetni (**C**reate,**R**ead,**U**pdate,**D**elete). Használatával a DataGrid segítségével meg tudjuk ezeket csinálni, nem kell külön felület az adatfelvitelhez, módosításhoz. Az adapterben meg tudjuk adni a megfelelő SQL parancsokat.

```csharp

public class Adapter
{
 SQLiteDataAdapter myAdapter=new SQLiteDataAdapter();
 DataTable table=new DataTable();
 //Ide jönnek majd a metódusok
}	
```

### Konstruktor

A konstruktorba kerülnek a megfelelő működést megvalósító SQL parancsok, a megfelelő paraméterekkel.

```csharp
public Adapter(SQLiteConnection conn)
{

}
```
A legalapvetőbb viselkedés az adatok lekérdezése, A **SelectCommand** tárolja az adatok lekérdezését megvalósító parancsot.

```csharp
myAdapter=new SQLiteDataAdapter("select * from tanulok",conn);
myAdapter.SelectCommand=new SQLiteCommand(conn);
myAdapter.SelectCommand.CommandText="select * from tanulok";
```
A következő az adatfelvitel megadása

```csharp
myAdapter.InsertCommand=new SQLiteCommand(conn);
myAdapter.InsertCommand.CommandText="insert into tanulok" +
"(VezetekNev,KeresztNev,AnyjaNeve,SzuletesEve,SzuletesiHely)"+
"values(@vezeteknev,@keresztnev,@anyjaneve,@szuletesiido,@szuletesihely)";
			
myAdapter.InsertCommand.Parameters.Add("@vezeteknev",DbType.String,50,"VezetekNev");
myAdapter.InsertCommand.Parameters.Add("@keresztnev",DbType.String,50,"KeresztNev");
myAdapter.InsertCommand.Parameters.Add("@anyjaneve",DbType.String,50,"AnyjaNeve");
myAdapter.InsertCommand.Parameters.Add("@szuletesiido",DbType.Int32,0,"SzuletesEve");
myAdapter.InsertCommand.Parameters.Add("@szuletesihely",DbType.String,50,"SzuletesiHely");
```
A paraméterek neve bármi lehet, viszont itt a típus megadása után nem változónevek állnak, hanem az adattábla oszlopnevei.

Az adatmódosítás megvalósítása

```csharp
myAdapter.UpdateCommand=new SQLiteCommand(conn);
myAdapter.UpdateCommand.CommandText="update tanulok set " +
"VezetekNev=@vezeteknev, KeresztNev=@keresztnev, AnyjaNeve=@anyjaneve, SzuletesEve=@szuletesiido,SzuletesiHely=@szuletesihely "+
 "where Id=@oldId";
			
myAdapter.UpdateCommand.Parameters.Add("@id",DbType.Int32,50,"Id");
myAdapter.UpdateCommand.Parameters.Add("@vezeteknev",DbType.String,50,"VezetekNev");
myAdapter.UpdateCommand.Parameters.Add("@keresztnev",DbType.String,50,"KeresztNev");
myAdapter.UpdateCommand.Parameters.Add("@anyjaneve",DbType.String,50,"AnyjaNeve");
myAdapter.UpdateCommand.Parameters.Add("@szuletesiido",DbType.Int32,0,"SzuletesEve");
myAdapter.UpdateCommand.Parameters.Add("@szuletesihely",DbType.String,50,"SzuletesiHely");
myAdapter.UpdateCommand.Parameters.Add("@oldId",DbType.Int32,0,"Id").SourceVersion=DataRowVersion.Original;
```
