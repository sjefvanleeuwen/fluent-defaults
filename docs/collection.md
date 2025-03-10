---
layout: default
title: "Collections"
---

# Async Defaulters

The `FluentDefaults` library supports default values for collections using the `ForEach` method to apply the same rule to multiple items in a collection.

## ForEach().DefaultFor().Is()

The `ForEach` method is used to iterate over each item in a specified collection property of an object. It allows you to apply rules or actions to each item within the collection. The `DefaultFor` method is used to specify a property of the items within the collection that you want to set a default value for. The `Is` method is used to assign the default value to the specified property. It is called after `DefaultFor` to set the value that should be used if the property is not already set. You can pass a fixed value or a function (eg `() => Guid.NewGuid()`)

### Example with fixed value

```csharp
internal sealed class CollectionCustomerWithDefaultForDefaulter : AbstractDefaulter<CollectionCustomer>
{
    int _number = 0;

    internal CollectionCustomerWithDefaultForDefaulter()
    {
        ForEach(x => x.Addresses).DefaultFor(x => x.Street).Is("- unknown street -");
        ForEach(x => x.Addresses2).DefaultFor(x => x.Id).Is(GetNumber);
        ForEach(x => x.Addresses1).DefaultFor(x => x.Street).Is((Customer x) => GetSome(x));
        ForEach(x => x.Addresses1).DefaultFor(x => x.City).Is((Address x) => GetSome(x));
    }

    private string GetSome(Customer x) => $"{x.Number5} street";
    private string GetSome(Address x) => $"City in {x.Region}";
    private int GetNumber() => ++_number;
}

public class CollectionCustomer
{
    public CollectionAddress[] Addresses { get; set; } = [new CollectionAddress()];
}

public class CollectionAddress
{
    public string? Street { get; set; }
    public string? City { get; set; }
}
```

## ForEach().SetDefaulter()

It is also possible to set a defaulter for each item in a collection using the `SetDefaulter` (optional with a reference to the instance) method.

### Example

```csharp
using FluentDefaults;

public class CollectionAddressDefaulter : AbstractDefaulter<CollectionAddress>
{
    public CollectionAddressDefaulter()
    {
        DefaultFor(x => x.Street).Is("- unknown street -");
    }
}

public class CollectionCustomerDefaulter : AbstractDefaulter<CollectionCustomer>
{
    public CollectionCustomerDefaulter()
    {
        ForEach(x => x.Addresses).SetDefaulter(new CollectionAddressDefaulter());
    }
}
```

You can then apply the default values to an instance of the `CollectionCustomer` class:

```csharp
var customer = new CollectionCustomer();
var defaulter = new CollectionCustomerDefaulter();

defaulter.Apply(customer);

Console.WriteLine(customer.Addresses1[0].Street); // Output: '- unknown street -'
```

> **Warning:**
> If deferred execution is detected, a `DeferredExecutionException` will be thrown:
> 
> This is the check:
> ```csharp
> if (!(currentValue is ICollection || currentValue is Array))
> {
>     throw new DeferredExecutionException("Deferred execution detected. Please ensure the collection is materialized.");
> }
> ```
> To avoid this issue, ensure the collection is materialized (e.g., using `.ToList()` or `.ToArray()`) before applying operations.