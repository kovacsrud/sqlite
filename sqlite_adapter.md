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
Az adatok egy DataTable-be kerülnek, ezt kapja majd a DataGrid.

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
### Adatlekérdező metódus

Eléggé egyszerű

```csharp
public DataTable GetTableData()
{		
	myAdapter.Fill(table);
	return table;
}
```
### Update metódus

Ez pedig elvégzi a módosításokat, a DataGrid-ben elvégzett változtatások az adatbázisba is beíródnak.

```csharp
public void UpdateData()
{
	myAdapter.Update(table);
		
}
```
A főprogram kódja a következőképpen alakul:

```csharp
public partial class Window1 : Window
{
//Az eseményvezérlők miatt itt deklarálni kell az adaptert
Adapter adapter;
	
public Window1()
{
			
			
}

void ButtonUpdate_Click(object sender, RoutedEventArgs e)
{
			
	adapter.UpdateData();
	adatok.Background=Brushes.White;
			
}
		
public void ColorTheRow(object sender, DataGridRowEditEndingEventArgs e)
{
	//Színezzük a szerkesztett rekordokat
	e.Row.Background=Brushes.Aqua;
}
													
}
```
A konstruktor kódja

```csharp
public Window1()
{
	InitializeComponent();
	SQLiteConnection db_connect=new SQLiteConnection("Data Source=d:\\csharp_proj\\tanulo_v1.db;Version=3;");
	adapter=new Adapter(db_connect);
		
	adatok.ItemsSource=adapter.GetTableData().DefaultView;
	adatok.RowEditEnding+=ColorTheRow;
			
			
}
```
Kell egy metódus, amely a gomb lenyomásakor meghívja az **UpdateData()** metódust.

```csharp
void ButtonUpdate_Click(object sender, RoutedEventArgs e)
{
		
	adapter.UpdateData();
	//Próbáljuk a DataGrid színét visszaállítani
	adatok.Background=Brushes.White;
			
}
```
Célszerű lehet jelölni, hogy mely sorok voltak változtatva

```csharp
public void ColorTheRow(object sender, DataGridRowEditEndingEventArgs e)
{
	e.Row.Background=Brushes.Aqua;
}
```
