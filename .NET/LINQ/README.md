# LINQ

LINQ (Language Integrated Query) is a set of extension functions to the `IEnumerable` (also referred to as "operators") in the .Net framework that allow you to query data from a variety of data sources where a data source is a type that implements the `IEnumerable`/`IEnumerable<T>` or `IQueryable`/`IQueryable<T>` interfaces.

## IEnumerable/IEnumerable\<T>

The `IEnumerable`/`IEnumerable<T>` interfaces are the base interfaces for all collections that can be enumerated. It has a method `GetEnumerator` and `GetEnumerator<T>` in the case of the generic interface that returns an `IEnumerator`/`IEnumerator<T>` object which can be used to iterate through the collection as it has methods `MoveNext` and `Reset` and a property `Current` that returns the current element in the collection.

## IQueryable/IQueryable\<T>

The `IQueryable`/`IQueryable<T>` interfaces are also IEnumerables but they have properties and methods that allow you to build a query that can be executed on the data source from the C# `Expression` provided in the LINQ query.

## Expressions

An `Expression` is a tree-like data structure that represents code in a form that can be executed at runtime. It is used to represent code as data and is used in LINQ to build queries that can be executed on the data source. The `Expression` class is in the `System.Linq.Expressions` namespace and is used to represent code as data. It has properties and methods that allow you to build a query that can be executed on the data source.

Linq extension methods to the `IEnumerable` take it's parameters as delegates and executes those delegates on the sequence since it's an in-memory data with normal c# types. On the other hand, Linq extension methods to the `IQueryable` take it's parameters as `Expression` and using the implementation of the `IQueryable` interface methods the `Expression` is translated into the query language of the external data source, for example Executing this query

```csharp
var result = students.Where(student => student.Age > 18)
                    .Select(student => student.Name);
```

if `students` is an `IEnumerable` the `Where` and `Select` methods in this case take it's parameters as delegates and executed the delegate on the data in the `IEnumerable` to generate the result,  but if `students` is an implementation for `IQueryable` for SQL for example, the `Where` and `Select` parameters in this case are `Expression`s. the `Expression`s will be used to translate the query into SQL, the native query language of the external data source and will send it to be executed on the data source and get the returned result from it.

## LINQ Operators

LINQ operators are extension methods of the `IEnumerable<T>` and `IQueryable<T>` interfaces. They are used to perform operations on sequences of data. The following are some of the most commonly used LINQ operators:

### Filtering Operators

- `Where` : Filters a sequence of values based on a predicate.
- `OfType` : Filters a sequence of values based on if they can be casted to a specified type.

### Projection Operators

- `Select` : Projects values that are based on a transform function.

> ```csharp
> // will return IEnumerable<string> names from the IEnumerable<Student>
> var query = students.Select(student => student.Name);
> ```

- `SelectMany` : Projects values that are based on a transform function and then flattens them into one sequence.

> ```csharp
>List<List<int>> lists = new List<List<int>> {
>   new List<int> {1, 2, 3},
>   new List<int> {12},
>   new List<int> {5, 6, 5, 7},
>   new List<int> {10, 10, 10, 12}
>};
>// Will return { 1, 2, 3, 12, 5, 6, 7, 10, 12 }
>IEnumerable<int> result = lists.SelectMany(list => list.Distinct());
> ```

- `Zip` : Merges two sequences by using a specified function.

> ```csharp
> var letters= new string[] { "A", "B", "C", "D", "E" };
> var numbers= new int[] { 1, 2, 3 };
> var q = letters.Zip(numbers, (l, n) => l + n.ToString());
> // Will return { "A1", "B2", "C3" }
> ```

### Set Operators

- `Distinct` : Returns distinct elements from a sequence.
- `DistinctBy` : Returns distinct elements from a sequence based on a key selector.
- `Except` : Returns the set difference of two sequences.
- `ExceptBy` : Returns the set difference of two sequences based on a key selector (the two sequences can be of heterogenous types).
- `Intersect` : Returns the set intersection of two sequences.
- `IntersectBy` : Returns the set intersection of two sequences based on a key selector (the two sequences can be of heterogenous types).
- `Union` : Returns the set union of two sequences.
- `UnionBy` : Returns the set union of two sequences based on a key selector.

### Sorting Operators

- `OrderBy` : Sorts the elements of a sequence in ascending order.
- `OrderByDescending` : Sorts the elements of a sequence in descending order.
- `ThenBy` : Performs a subsequent ordering of the elements in a sequence in ascending order.
- `ThenByDescending` : Performs a subsequent ordering of the elements in a sequence in descending order.
- `Reverse` : Inverts the order of the elements in a sequence.

### Quantifier Operators

- `All` : Determines whether all elements of a sequence satisfy a condition.
- `Any` : Determines whether any element of a sequence satisfies a condition.
- `Contains` : Determines whether a sequence contains a specified element.

### Partitioning Operators

- `Take` : Returns a specified number of contiguous elements from the start of a sequence.
- `TakeWhile` : Returns elements from a sequence as long as a specified condition is true.
- `Skip` : Bypasses a specified number of elements in a sequence and then returns the remaining elements.
- `SkipWhile` : Bypasses elements in a sequence as long as a specified condition is true and then returns the remaining elements.
- `Chunk` : Splits a sequence into chunks of a specified size.

### Join Operators

- `Join` : Correlates the elements of two sequences based on matching keys and returns a pair of 1st sequence and 2nd sequence objects that correlate to the result selector.
- `GroupJoin` : Correlates the elements of two sequences based on matching keys and returns every element of the first sequence with a list of it's correlating elements in the second sequence to the result selector.

### Grouping Operators

- `GroupBy` : Groups the elements of a sequence based on a key selector function and returns a collection of elements that are grouped by the key and returns a list of `IGrouping<TKey, TElement>`.

- `ToLookup` : Groups the elements of a sequence based on a key selector function and returns a collection of elements that are grouped by the key and returns a `ILookup<TKey, TElement>` which is a one-to-many dictionary.

### Aggregation Operators

- `Count` : Returns the number of elements in a sequence.
- `Sum` : Computes the sum of the sequence of numeric values.
- `Min` : Returns the minimum value in a sequence.
- `Max` : Returns the maximum value in a sequence.
- `Average` : Computes the average of the sequence of numeric values.
- `Concat` : Concatenates two sequences.
- `Aggregate` : Applies an accumulator function over a sequence.

### Element Operators

- `First` : Returns the first element of a sequence.
- `FirstOrDefault` : Returns the first element of a sequence, or a default value if the sequence contains no elements.
- `Last` : Returns the last element of a sequence.
- `LastOrDefault` : Returns the last element of a sequence, or a default value if the sequence contains no elements.
- `Single` : Returns the only element of a sequence.
- `SingleOrDefault` : Returns the only element of a sequence, or a default value if the sequence contains no elements or more than one element.
- `ElementAt` : Returns the element at a specified index in a sequence.
- `ElementAtOrDefault` : Returns the element at a specified index in a sequence or a default value if the index is out of range.
- `DefaultIfEmpty` : Returns the elements of the specified sequence or the type parameter's default value in a singleton collection if the sequence is empty.

### Generation Operators

- `Enumerable.Empty<T>` : Returns an empty `IEnumerable<T>`.
- `Enumerable.Range` :  Generates a sequence of integral numbers within a specified range.
- `Enumerable.Repeat` : Generates a sequence that contains one repeated value.

### Conversion Operators

- `ToArray` : Converts a sequence to an array.
- `ToList` : Converts a sequence to a list.
- `ToDictionary` : Converts a sequence to a dictionary based on a key selector delegate.
- `ToLookup` : Converts a sequence to a lookup based on a key selector delegate.
- `Cast` : Casts the elements of a sequence to the specified type.
- `AsEnumerable` : Returns the input sequence typed as `IEnumerable<T>`.
- `AsQueryable` : Returns the input sequence typed as `IQueryable<T>`.

## Query Execution

LINQ queries aren't usually executed once defined. Instead, they are executed when the result is enumerated.

```csharp
var query = students.Where(student => student.Age > 18)
                    .Select(student => student.Name);

foreach (var name in query)
    Console.WriteLine(name);
```

in the previous code, the query is stored and not executed until an enumeration is executed on it like the `foreach` loop. In the `foreach` loop the next value is returned from the `IEnumerator.MoveNext()` of the `IQueryable` one by one.

This means we can define a query and after enumerating it if the data source changes and then we enumerate it again we will see different result reflecting the changes. the enumeration result will always reflect the latest change of the data source.

LINQ queries execution is either deferred like the previous example (most cases) or can be immediate.

### Immediate Execution

The query is executed immediately when it's defined and the result is returned. when using some operators like `ToList`, `ToArray`, `ToDictionary` on the query it's executed immediately and the result is returned. this is called eager execution.

### Deferred Execution

LINQ queries that return `IEnumerable` or `IOrderedEnumerable` are not executed immediately, it's execution is deferred until it's enumerated. this is called lazy execution.

Deferred Execution operator can be additionally classified into two types

#### Streaming

Streaming operators do not have to read all the source data before they yield elements. At the time of execution, a streaming operator performs its operation on each source element as it is read and yields the element if appropriate. A streaming operator continues to read source elements until a result element can be produced. This means that more than one source element might be read to produce one result element.

#### Non-Streaming

Non-streaming operators have to read all the source data before they yield elements. At the time of execution, a non-streaming operator reads all the source elements, performs its operation, and yields all the result elements. This means that all source elements are read before any result elements are produced.

You can see the [operator classification table](https://learn.microsoft.com/en-us/dotnet/csharp/linq/query-a-collection-of-objects#classification-table) to check the class of each operator.

_ _ _

### *That's it, now you know.*
