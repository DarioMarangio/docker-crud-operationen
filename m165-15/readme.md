# Mongo crud operationen

## neues projekt erstellen

````
dotnet new console --name [projektname]

````
## Nugetpakage installiren


````
cd [projektname]

dotnet add package MongoDB.Driver

starten mit dotnet run 

````
## starten des containers
```
docker run --name m165 -p:27017:27017 -d frm1971/m165-15
```
## anbindung an der db

````
using MongoDB.Bson;
using MongoDB.Driver;

MongoClient client = new MongoClient("mongodb://localhost:27017");

````

## create a class with the attributes and get set

````
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

public class Movie
{
    [BsonId]
    public ObjectId Id { get; set; } = ObjectId.GenerateNewId();

    public string Title { get; set; } = "";
    public int Year { get; set; }
    public string Summary { get; set; } = "";
    public List<string> Actors { get; set; } = new List<string>();
}

````

## Get the database collection
````

var m165Db = client.GetDatabase("[databasename]");
var Collection = m165Db.GetCollection<[class]>("[collection_name]");

````


## list all databases

````
var databaseNames = client.ListDatabaseNames().ToList();
Console.WriteLine("Databases: " + string.Join("," , databaseNames));
Console.WriteLine("");

````

## list collection from a database

````
var Db = client.GetDatabase("[database_name]");
var collections = Db.ListCollectionNames().ToList();
Console.WriteLine("Aufgabe B: ----------------------------------------------");
Console.WriteLine("Collections: " + string.Join(",", collections));
Console.WriteLine("");

````

## find one (firstorDefault)
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/findOne
````
var findFilterone = Builders<[class]>.Filter.Eq(f => f.[attribute], [Value]);
var findResultFirstOrDefault = Collection.Find(findFilterone).FirstOrDefault();
Console.WriteLine(" (FirstOrDefault): " + findResultFirstOrDefault);
Console.WriteLine("");
````

## find many 
 https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/findMany
````
var findFiltermany = Builders<[class]>.Filter.AnyEq(f => f.[attribute], "[Value]"); //when a string ""
var findResultsmany = Collection.Find(findFiltermany).ToList();
Console.WriteLine("Findmany (Liste): ");
foreach (var item in findResultsmany)
{
    Console.WriteLine("- " + item);
}
Console.WriteLine("");
````


## find all
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/findMany/
````
var findFilterAll = Builders<[class]>.Filter.Empty;
var findResultsall = Collection.Find(findFilterAll).ToList();

Console.WriteLine("Findall (Liste): ");
foreach (var item in findResultsall)
{
    Console.WriteLine("- " + item);
}
Console.WriteLine("");
````

## insert one
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/insertOne/
````
var myMovie = new [class]();
myMovie.Title = "The Da Vinci Code";
myMovie.Year = 2006;
myMovie.Summary = "So dunkel ist der Betrug an der Menschheit";
myMovie.Actors = new List<string>(){"Tom Hanks","Audrey Tautou"};

moviesCollection.InsertOne(myMovie);

var filterInsertedItem = Builders<[class]>.Filter.Eq(f => f.Id, myMovie.Id);
var insertedone = moviesCollection.Find(filterInsertedItem).Single();


Console.WriteLine("One Inserted: " + insertedone + " " + insertedone.Id);
Console.WriteLine("");


````

## insert many
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/insertMany
````
var newMovies = new List<Movie>();

var myMovie1 = new Movie();
myMovie1.Title = "Ocean's Eleven";
myMovie1.Actors = new List<string>(){"George Clooney", "Brad Pitt", "Julia Roberts"};
myMovie1.Summary = "Bist du drin oder draussen?";
myMovie1.Year = 2001;

newMovies.Add(myMovie1);

var myMovie2 = new Movie();
myMovie2.Title = "Ocean's Twelve";
myMovie2.Actors = new List<string>(){"George Clooney", "Brad Pitt", "Julia Roberts", "Andy Garcia"};
myMovie2.Summary = "Die Elf sind jetzt Zwölf.";
myMovie2.Year = 2004;

newMovies.Add(myMovie2);

moviesCollection.InsertMany(newMovies);

var filterNewInsertedmany = Builders<[class]>.Filter.In(m => m.Id, new[]{myMovie1.Id, myMovie2.Id});
var insertedmany = moviesCollection.Find(filterNewInsertedmany).ToList();


Console.WriteLine(" many Inserted:");

foreach (var singleinsert in insertedmany)
{
    Console.WriteLine("- " + singleinsert + " " + singleinsert.Id);
}
Console.WriteLine("");

````


## update one
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/updateOne
````
var filterupdateone = Builders<[class]>.Filter.Eq(f => f.[attribute], [oldValue]);
var updateone = Builders<[class]>.Update.Set(d => d.[attribute], [new_value]);
var resultone = Collection.updateone(filterupdateone, updateone);
Console.WriteLine("Update '[oldvalue]' -> '[new_value]' " ); 
Console.WriteLine("");
````

## update many

https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/updateMany
````
var updateFiltermany = Builders<[class]>.Filter.Eq(f => f.[attribute], "[oldvalue]");
var updatemany = Builders<[class]>.Update
    .Set(d => d.[attribute], "[new_value]"); //atributes needs to be the same

var resultmany = Collection.UpdateMany(updateFiltermany, updatemany);
Console.WriteLine("Update '[oldvalue]' -> '[new_value]' (Anzahl): " + resultmany.ModifiedCount); 
Console.WriteLine("");

````

## Delete one
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/deleteOne

````
var filterDeleteOne = Builders<[class]>.Filter.Eq(f => f.[attribute], [value]);
var deleteResultOme = Collection.DeleteMany(filterDeleteOne);
Console.WriteLine("Delete many (filter) [value] (Anzahl): " ); 
Console.WriteLine("");
````

## Delete many
https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/deleteMany
````
var deleteFilterMany = Builders<[class]>.Filter.Lte(f => f.[attribute], [value]);
var deleteResultMany = Collection.DeleteMany(deleteFilterMany);
Console.WriteLine("Delete many (filter) [value] (Anzahl): " + deleteResultMany.DeletedCount); 
Console.WriteLine("");
````

## agregate 
````
var aggregateResult = moviesCollection.Aggregate()
    .Match(m => m.Year >= 2000)
    .Group( m => m.Year, g => new{ Jahr = g.Key, Anzahl=g.Count()})
    .SortBy(m => m.Jahr)
    .ToList();

Console.WriteLine("Aufgabe I (Zusatzaufgabe): ------------------------------");
Console.WriteLine("Filme pro Jahr ab 2000");
foreach (var item in aggregateResult)
{
    Console.WriteLine("- " + item.Jahr + " " + item.Anzahl);
}
Console.WriteLine("");

````
## list in Json
````
var allDocuments = Db.GetCollection<BsonDocument>("[collection_name]");
foreach (var item in allDocuments.Find(f => true).ToList())
{
    Console.WriteLine(item.ToString());
}
````