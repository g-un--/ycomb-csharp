# ycomb-csharp
Y combinator in C#

Just a toy program.

Can be useful for creating a pipeline of recursive functions

## Usage

Compute fibonacci with caching without altering original fibonacci function

```

var fibonacci = new YComb<int, int>.Lambda(rec => n => n < 2 ? n : rec(n - 1) + rec(n - 2));

var log = new YComb<int, int>.Lambda(rec => n =>
{
    Console.WriteLine($"Calling fibonacci for: {n}");
    return rec(n);
});
var storage = new Dictionary<int, int>();
var cache = new YComb<int, int>.Lambda(rec => n =>
{
    var hasItem = storage.TryGetValue(n, out var cached);
    if (!hasItem)
    {
        var result = rec(n);
        storage[n] = result;
        cached = result;
    }
    return cached;
});

var fib = YComb<int, int>.Fix(fibonacci, log, cache);
var result = fib(10);
Console.WriteLine($"Result: {result}");

public static class YComb<T, R>
{
    delegate Func<T, R> Y(Lambda x);

    delegate Func<T, R> Rec(Rec rec);

    public delegate Func<T, R> Lambda(Func<T, R> rec);

    public static Func<T, R> Fix(params Lambda[] fs)
    {
        var f = fs.Aggregate((fx, decorator) => x => decorator(fx(x)));
        return new Y(f => new Rec(x => v => f(x(x))(v))(new Rec(x => v => f(x(x))(v))))(f);
    }
}

```
