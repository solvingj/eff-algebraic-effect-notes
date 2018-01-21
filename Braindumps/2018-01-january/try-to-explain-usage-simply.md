## Starting from Simple Example  

Here's an example from the C# implementation where Eff is used to model reading data from Asp.Net's `AppSettings`. This is a very common "effectful task", basically identical to reading environment variables, so in theory the pattern would be virtually identical. 

https://github.com/nessos/Eff/blob/master/src/examples/Eff.Examples.Config/Program.cs

 	
From this example, it seems the way to implement Eff is to create a bunch of related `Handler` classes and `Effect` classes, each of which encapsulates a small subset of related "effectful work" in your program. These classes need to extend `Handler` and `Effect` interfaces, and implement/override the single fundamental method of a `Handler`: the `Handle()` method.  The "novel pattern" is: 

1. Create an instance of a custom `Handler<T>` class
2. Create an instance of the related custom `Effect<T>` with the same type `T`. 
3. Call the `Handle()` method, of the `Handler<T>` instance, passing it the instance of `Effect<T>`
4. Crucially, the required and manual last step of every `Handle()` method needs to be to call the built-in `SetResult()` method of the `Effect` instance.  
5. Crucially, you don't return anything from Handle (either `void` or `Task`).

So, lets imagine some other very common system operations.  I can imagine a nuget package called `FileIOEffect`.  If it's thorough, it will try to encapsulate every operation you might want do with FileIO.  This would include a `ReadFileEffect`, `WriteFileEffect`, and perhaps a `ReadAllToString` effect.  Each with slightly different member variables to capture whatever "result type" makes sense.  Then, it would need a `FileIOHandler`.  The interesting thing is that the `FileIOHandler` is that you needs to use a `switch` statement to pattern match on all the possible related `Effect` subtypes, and perform the appropriate logic inside the `case` statements.  I imagine the `case` bodies will often be extracted to static method in the handler class.  Again, the last part is also intereseting, you won't be returning values from these functions, you have to remember to use `SetResult<T>(whatever_you_want_returned)`. 

## Finding Familiarity  

This whole formalized pattern taken together is unique, but there are familiar pieces here.  In one respect, it has elements of the "Execute around" pattern. In another respect, it's shares some elements with the `out parameters` used in C#.   It also has an element of the visitor pattern based on the effect subtypes.  It seems like a kind of clever strategy and convention, but unlike the majority of other "clever strategies" I've seen, this one comes based on a complete algebra, and a massive body of successful research and trial. In C#, the implementation uses a lot of interface and inheritence.  I guess in FP languages, they would use higher-kinded types to achieve the same thing, but I'm not sure.  [Seeking Other Perspectives Here] (https://github.com/solvingj/eff-algebraic-effect-notes/issues/new))\

## Developer Experience - Compiler Considerations  

There are several schools of thought around coding without return values, but it's certainly a departure for many mainstream languages and we do lose a compile-time saftey net here which seems to be a real consideration.  IDE's and compilers currently all throw an error until you return an appropriate variable, but you'll get no warning if you forget to do `SetResult()`.  We would need a compiler error looking for a valid `SetResult()` call inside all `Handle()` methods for all  classes that derive from `Handler<T>`.  This would require a VS extension, if it's even possible. It probably is, since VS now warns if you don't set a value on an `out` variable in C#, but that's a language feature, so who knows.  This is definitely the type of thing that motivates people say "the language isn't designed for that pattern".  It's also definitely a case where the pattern has to prove it's value before the tooling teams will spend the time accomodating it. 