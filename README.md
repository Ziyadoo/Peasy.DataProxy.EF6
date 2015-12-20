![peasy](https://www.dropbox.com/s/2yajr2x9yevvzbm/peasy3.png?dl=0&raw=1)

# Peasy.DataProxy.EF6

Peasy.DataProxy.EF6 provides the [EF6DataProxyBase](https://github.com/peasy/Peasy.DataProxy.EF6/blob/master/Peasy.DataProxy.EF6/EF6DataProxyBase.cs) class.  EF6DataProxyBase is an abstract class that implements [IDataProxy](https://github.com/ahanusa/Peasy.NET/wiki/Data-Proxy), and can be used to very quickly and easily provide a data proxy that communicates with a database using Entity Framework [code first](https://msdn.microsoft.com/en-us/data/jj193542.aspx) or [database first](https://msdn.microsoft.com/en-us/data/jj206878.aspx) classes.

###Where can I get it?

First, install NuGet. Then create a project for your Entity Framework data proxy implementations to live.  Finally, install Peasy.DataProxy.EF6 from the package manager console:

``` PM> Install-Package Peasy.DataProxy.EF6 ```

You can also download and add the Peasy.DataProxy.EF6 project to your solution and set references where applicable

### Creating a concrete EF6 data proxy

To create an EF6 repository, you must inherit from [EF6DataProxyBase](https://github.com/peasy/Peasy.DataProxy.EF6/blob/master/Peasy.DataProxy.EF6/EF6DataProxyBase.cs).  There is one contractual obligation to fullfill.

1.) Override [GetDbContext](https://github.com/peasy/Peasy.DataProxy.EF6/blob/master/Peasy.DataProxy.EF6/EF6DataProxyBase.cs#L25) - this method should return your custom [DbContext](https://msdn.microsoft.com/en-us/library/system.data.entity.dbcontext(v=vs.113).aspx) that all methods in the proxy will consume.

Here is a sample implementation

```c#
// Your DbContext ...
public class OrdersDotComContext : DbContext
{
    public DbSet<Category> Categories { get; set; }
    public DbSet<Customer> Customers { get; set; }
}

public class OrderDataProxy : EF6DataProxyBase<Order, Order, long>
{
    protected override DbContext GetDbContext()
    {
        return new OrdersDotComContext();
    }
}

```

In this example, we create an EF6 order repository.  The first thing to note is that we supply ```Order``` as our DTO and TEntity generic constraints, respectively.  By contractual agreement, these types must implement [```IDomainObject<T>```](https://github.com/peasy/Peasy.NET/blob/master/Peasy.Core/IDomainObject.cs).  The ```long``` specifies the key type that will be used for all of our arguments to the [IDataProxy](https://github.com/peasy/Peasy.NET/wiki/Data-Proxy) methods.

As part of our contractual obligation, we override ```GetDbContext``` to provide the context in which the data proxy will consume to work with Entity Framework.

Note that in this example, we use the Order class that will become arguments and return types for our data proxy methods.  We could also specify 2 different classes as well, which might be the case if you need to supply EF with a different class, which will be the case if you use a [database first](https://msdn.microsoft.com/en-us/data/jj206878.aspx) approach. 

An implementation might look like this:

```c#
public class OrderDataProxy : EF6DataProxyBase<Order, tOrder, long>
{
    protected override DbContext GetDbContext()
    {
        return new OrdersDotComContext();
    }
}
```

In this example, we provide Order and tOrder as the first 2 generic arguments, where Order is your DTO and tOrder reprensents a EF6 class generated by database first.  It should be noted that Order and tOrder must implement [```IDomainObject<T>```](https://github.com/peasy/Peasy.NET/blob/master/Peasy.Core/IDomainObject.cs).

In the case that you use a database first approach, you will want to create partial classes for your entities that implement IDomainObject<T> in a separate class file(s) so that they aren't clobbered when you regenerate them.

EF6DataProxyBase relies on [Automapper](https://github.com/AutoMapper/AutoMapper) to map DTOs to entities and as a result, if you supply generic arguments of differing types, you must [provide mappings](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started) in your application to account for this.  You may also use a mapping technology of your [choice]().

By simply inheriting from EF6DataProxyBase and overriding GetDbContext, you have a full-blown peasy EF6 data proxy that performs CRUD operations against a database table.

### Execution hooks

### Mapping Logic

By default, EF6DataProxyBase is configured to use [Automapper](https://github.com/AutoMapper/AutoMapper) to map DTOs to Entities.  However, if you use a different mapping tool by creating a facade (wrapper) class that implements [Peasy.DataProxy.EF6.IMapper](https://github.com/peasy/Peasy.DataProxy.EF6/blob/master/Peasy.DataProxy.EF6/IMapper.cs) and supplying it to the overloaded constructor of EF6DataProxyBase.

Here is a sample implementation of a custom mapper using [Mapster](https://github.com/eswann/Mapster)

```c#
public class MapsterHelper : IMapper
{
    public TDestination Map<TSource, TDestination>(TSource source)
    {
        return TypeAdapter.Adapt<TDestination>(source);
    }
	
    public TDestination Map<TSource, TDestination>(TSource source, TDestination destination)
    {
        return TypeAdapter.Adapt<TSource, TDestination>(source, destination);
    }
}
```

And wiring it up...

```c#
public class OrderDataProxy : EF6DataProxyBase<Order, tOrder, long>
{
    public OrderDataProxy() : base(new MapsterHelper())
    {
    }
        
    protected override DbContext GetDbContext()
    {
        return new OrdersDotComContext();
    }
}
```

Now the OrderDataProxy will use Mapster instead of the preconfigured AutoMapper.
