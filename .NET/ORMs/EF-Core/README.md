# EF Core

EF Core is a fully fledged ORM that allows you to access and manipulate data in a database using .Net objects.

## How to use

First you create your database model classes. let's say we have a database for a supermarket in our database we store the data of the products, the customers, the purchases. we will have a table for each of these entities and we will create classes that represent these tables.

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

```csharp
public class Purchase
{
    public int ProductId { get; set; }
    public int CustomerId { get; set; }
    public DateTime TimeStamp { get; set; }
    public int Quantity { get; set; }
}
```

Now we create a class that represents our database. in EF Core this class is called the `DbContext`. This class inherits form `DbContext` and has a `DbSet<T>` property for each table in the database where `T` is the model/type that represents that table.

```csharp
public class SupermarketDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Purchase> Purchases { get; set; }
}
```

now we need to define the relation between these tables and define the properties of there columns, like defining which is the primary key of the table, does it have any foreign keys, any constraints, etc.

one way to do this is by overriding the `OnModelCreating` method of the `DbContext` class this way.

```csharp
public class SupermarketDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Purchase> Purchases { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Purchase>()
            .HasKey(p => new { p.ProductId, p.CustomerId });

        modelBuilder.Entity<Purchase>()
            .HasOne(p => p.Product)
            .WithMany(p => p.Purchases)
            .HasForeignKey(p => p.ProductId);

        modelBuilder.Entity<Purchase>()
            .HasOne(p => p.Customer)
            .WithMany(c => c.Purchases)
            .HasForeignKey(p => p.CustomerId);
    }
}
```

Now to tell this DbContext class which database to connect to we need to override the `OnConfiguring` method of the `DbContext` class and pass it the connection string of the database.

```csharp
public class SupermarketDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Purchase> Purchases { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=Supermarket;Trusted_Connection=True;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Purchase>()
            .HasKey(p => new { p.ProductId, p.CustomerId });

        modelBuilder.Entity<Purchase>()
            .HasOne(p => p.Product)
            .WithMany(p => p.Purchases)
            .HasForeignKey(p => p.ProductId);

        modelBuilder.Entity<Purchase>()
            .HasOne(p => p.Customer)
            .WithMany(c => c.Purchases)
            .HasForeignKey(p => p.CustomerId);
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=Supermarket;Trusted_Connection=True;");
    }
}
```

At this point the our `DbContext` is ready to use and we can start using it to access and manipulate the data in the database.

to start using the `DbContext` we need to create an instance of it and use LINQ methods with it's `DbSet`s to get data from the database and methods like `Remove` and `Add` to add data to it.

```csharp
using (var db = new SupermarketDbContext())
{
    var products = db.Products.Where(p => p.Stock > 0).ToList();
    var customers = db.Customers.ToList();
    var purchases = db.Purchases.ToList();

    var newProduct = new Product { Name = "Milk", Price = 1.5m, Stock = 100 };
    db.Products.Add(newProduct);
    db.SaveChanges();
}
```

All the changes that we make to the `DbContext` are not saved to the database until we call the `SaveChanges` method.

Objects retrieved or "added" to the DbContext are being tracked until the `SaveChanges` method is called. This means that if we change the value of a property of an object that we retrieved from the database and then call the `SaveChanges` method, the value of that property will be updated in the database.

## Migrations

In the previous example we assumed the database is already created and the tables are already created and the relations between them are already defined. but what if we want to create a new database or add a new table to the database or add a new column to a table or change the type of a column or add a new relation between tables.

EF Core allows us to create "migrations" which are a set of instructions that tells EF Core how to change the database schema to match the changes that we made to the `DbContext`.

After creating your `DbContext` and your model classes you can use the `dotnet ef migrations add` command to create a migration or the `Add-Migration` in the PM Console in visual studio.

```bash
Add-Migration InitialCreate
```

This will create a new folder in your project called `Migrations` and inside it a new file with a name that represents the name of the migration and inside this file there will be a class that represents the migration and inside this class there will be a method called `Up` that contains the instructions to change the database schema to match the changes that we made to the `DbContext`.

Then we run the command `Update-Database` in the PM Console in visual studio or `dotnet ef database update` in the terminal to run the Up method and apply the migration to the database.

```bash
Update-Database
```

If we want to undo the changes that we made to the database we can use the command `Remove-Migration` in the PM Console in visual studio or `dotnet ef migrations remove` in the terminal.

```bash
Remove-Migration
```

Creating the database this way is called code-first approach.

## Scaffolding (Reverse Engineering)

This approach is called database-first approach.

If you have an existing database and you want to create a `DbContext` and model classes that represent the tables in the database you can use the `dotnet ef dbcontext scaffold {db provider name}` command and give it the connection string of the database and the provider name or `Scaffold-DbContext` in the PM Console in visual studio.

```bash
Scaffold-DbContext "Server=localhost;Database=Supermarket;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer
```

This will generate a DbContext class with model classes ready to be used to access and manipulate the data in the database.

_ _ _

### *That's it, now you know.*
