---
title: "Don't fall for TaskCompletionSource traps"
excerpt: "Dealing with unhandled task exception"
header:
 teaser:
tags: 
  - csharp
  - .net
  - dotnet
  - TaskCompletionSource
---

### Wrapping callback hell with TaskCompletionSource

Ever wanted to turn callback style async code to awaitable form?
You might used TaskCompletionSource for it.

```csharp
  class Program
    {
        static void Main(string[] args)
        {
            Run();
            Console.ReadLine();
        }

        static async Task Run()
        {
            CallbackStyleAsyncMethod((result) => Console.WriteLine(result)); //callback style
            var asyncResult = await CallbackStyleAsyncMethodWrappedAsync(); //async/await style
            Console.WriteLine(asyncResult);
        }


        static Task<string> CallbackStyleAsyncMethodWrappedAsync()
        {
            var tsc = new TaskCompletionSource<string>();
            CallbackStyleAsyncMethod((res) => tsc.SetResult(res));
            return tsc.Task;
        }


        static void CallbackStyleAsyncMethod(Action<string> onFinish)
        {
            Task.Delay(5000).ContinueWith((_) => onFinish("I'm finished"));
        }
    }
```

There is one problem with above code that happens very often.
In case of exception it won't be catched in try/catch block. 
This exception won't be even printed in console windows unless It's handled by TaskScheduler.UnobservedTaskException Event or DomainUnhandledException Event.

```csharp
 try
            {
                var tsc = new TaskCompletionSource<string>();
                CallbackStyleAsyncMethod((res) => tsc.SetResult(new string[] { "a" }[1])); // runtime exception
                return tsc.Task;
            }
            catch (Exception ex)
            {
                return Task.FromResult(""); // won't be catched
            }
```

Solution is trivial but easy to forget

```csharp
  static Task<string> CallbackStyleAsyncMethodWrappedAsync()
        {
            try
            {
                var tsc = new TaskCompletionSource<string>();
                CallbackStyleAsyncMethod((res) =>
                {
                    try
                    {
                        tsc.SetResult(new string[] { "a" }[1]); // runtime exception
                    }
                    catch (Exception ex)
                    {
                        tsc.SetException(ex); // exception is handled here
                    }
                });
                return tsc.Task;
            }
            catch (Exception ex)
            {
                return Task.FromResult(""); // will be catched
            }
        }
```

Cheers!
