# Misc

## Setting up PostgreSQL extensions

The provider allows you to specify PostgreSQL extensions that should be set up in your database.
Simply use HasPostgresExtension in your context's OnModelCreating:

```c#
protected override void OnModelCreating(ModelBuilder modelBuilder) {
    modelBuilder.HasPostgresExtension("hstore");
}
```

## Optimistic Concurrency and Concurrency Tokens

Entity Framework supports the concept of optimistic concurrency - a property on your entity is designated as a concurrency token, and EF detects concurrent modifications by checking whether that token has changed since the entity was read. You can read more about this in the [EF docs](https://docs.microsoft.com/en-us/ef/core/modeling/concurrency).

Although applications can update concurrency tokens themselves, we frequently rely on the database automatically updating a column on update - a "last modified" timestamp, an SQL Server `rowversion`, etc. Unfortunately PostgreSQL doesn't have such auto-updating columns - but there is one feature that can be used for concurrency token. All PostgreSQL tables have a set of [implicit and hidden system columns](https://www.postgresql.org/docs/current/static/ddl-system-columns.htm://www.postgresql.org/docs/current/static/ddl-system-columns.html), among which `xmin` holds the ID of the latest updating transaction. Since this value automatically gets updated every time the row is changed, it is ideal for use as a concurrency token.

To enable this feature on an entity, insert the following code into your models' `OnModelCreating` method:

```c#
modelBuilder.Entity<MyEntity>().ForNpgsqlUseXminAsConcurrencyToken();
```

## PostgreSQL Index Methods

PostgreSQL supports a number of _index methods_, or _types_. These are specified at index creation time via the `USING _method_` clause, see the [PostgreSQL docs for `CREAT
E INDEX`](https://www.postgresql.org/docs/current/static/sql-createindex.html) and [this page](https://www.postgresql.org/docs/current/static/indexes-types.html) for info o
n the different types.

The Npgsql EF Core provider allows you to specify the index method to be used by specifying `ForNpgsqlHasMethod()` on your index in your context's OnModelCreating:
```c#
protected override void OnModelCreating(ModelBuilder modelBuilder) {
    modelBuilder.Entity<Blog>()
        .HasIndex(b => b.Url)
        .ForNpgsqlHasMethod("gin");
}
```

## Using a database template

When creating a new database,
[PostgreSQL allows specifying another "template database"](http://www.postgresql.org/docs/current/static/manage-ag-templatedbs.html)
which will be copied as the basis for the new one. This can be useful for including database entities which aren't managed by Entity Framework. You can trigger this by using HasDatabaseTemplate in your context's `OnModelCreating`:

```c#
modelBuilder.HasDatabaseTemplate("my_template_db");
```
