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


