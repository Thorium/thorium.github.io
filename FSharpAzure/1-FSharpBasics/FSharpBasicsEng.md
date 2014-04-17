# F# in Brief #

F# is a clear programming language, which, by default, drives user towards functional programming solutions. This way the program state modelling goes correctly and programs are easily multi-threaded and they have good maintainability.

> "Functional programming combines the flexibility and power of abstract mathematics with the intuitive clarity of abstract mathematics." - XKCD

Although small is beautiful, and F# can be very abstract language, it is not about to get the most succinct syntax. It is about the best (and most clear) syntax fitting for the situation. The power of a language is to be able to do things in many ways, but goodness to intuitively drive the user to the best solution.

Maybe a small step from experimental trial-and-error-program-maintenance to a bit more mathematical way wouldn't be so bad?

You may learn to read the code easily, but painless writing of the programs will take its time: learning new things will take some patience. F#-compiler is stricter than C# compiler, so you have to hit your head to the wall at first, but when the code compiles, it will more probably not fail runtime. On the other hand it will give some achievement-feeling and make C#-veteran to feel that coding can be fun.

## Creating F-Sharp-project in the Visual Studio ##

You can either use Azure-Worker-role (which has separate instructions), or make a new project. By default F# project will compile as dll, like C#-project, and those are compatible which each other.

If you want to make a new project in Visual Studio:
File -> New -> Project... -> Other Languages -> Visual F# ->F# Library

## C# with F# (and why not also VB.NET) ##

The strengths of C# is the ready-made wizards when they are enough, like in XAML-development. From F# you may reference and use C#-dlls directly. The same Visual Studio solution may have both F# and C# projects, but the visibility of the changes needs a compilation of the referenced project.

Try to add C#-project (library, console-app, test project, or something) to the same solution with F# project and add a reference:
From Solution Explorer, C#-project, References-folder: right click and:
Add reference... -> Solution -> check mark F#-project and OK.

Here is a sample content for F#-file (with more details below):

    [lang=fsharp]
    namespace MyLibrary
    
    type MyClass1() = 
        member this.X = "F#"
    
    module MyModule =
    
        let StaticItems = 42
    
        let MyList = ["a";"b";"c"] |> List.toSeq
    
        let MyAddFive x = x+5;
    
        let MyGenericFunction x = "Hello " + x.ToString();
    
        type UserData1() = 
            member val Email = "" with get, set

...and those can be used (after compilation) from the C#-project's class directly:


    [lang=csharp]
	var class1 = new MyLibrary.MyClass1();
	var fromFs = class1.X;
	
	int Item = MyLibrary.MyModule.StaticItems;
	
	IEnumerable<string> response = MyLibrary.MyModule.MyList;
	
	int seven = MyLibrary.MyModule.MyAddFive(2);
	
	var res = MyLibrary.MyModule.MyGenericFunction<int>(2);
	
	var class2 = new MyLibrary.MyModule.UserData1();
	class2.Email = "no@spam.com";
        
This will be a safety net for old C#-developer before F# is totally under belt.
Likewise would work referencing to VB.NET-component.

Notable limitations:

- Property/variable/function names are usually written in lower case in F#. 
    - But this is only a practice to separate those from types/classes.
- F#-default-list is a linked list of F#-core. 
    - Seq is a strongly typed IEnumerable.
- Also F#-default-function is a function of F#-core. 
    - From F# you may call C#-method that takes C# Func-parameter, F#-function is OK in this way.
    - But if you return a function from F#, then the C# developer will get confused.
    - You may explicitly return C# Func -data type from F#, but using C# Func inside F# is not natural.
        - Func and Action are separate classes in C#, and "void" is not a type. 
        - F# void is called unit and it is a data type. 

These limitations will matter, if you know in the first place, that you will develop F#-component to use from C#-development.

Then the language itself:

## Hello world ##

With .NET classes, "Hello World"-program can be created e.g. this way:

    [lang=fsharp]
    open System
    let x = "Hello World"
    Console.WriteLine x

F# will drive you to the direction that you don't re-use your "variables" (/values) by default. So that when x is once defined, rather use x2 next time (...with better naming conventions). Actually you need variables only to maintain mutable state of loops. F# will drive you to use recursion instead of loops by default. More about this later...

When comparing equality, just one equals sign is used.
Comments are // or (* and *). Private and public scopes are in use.

You may also give values long names with spaces like these (but international keyboard layouts will make these a bit more difficult to use):

    [lang=fsharp]
    let ``Hello World Container`` = "Hello World"

## Interactive ##

Assuming that Resharper has not totally messed-up your keyboard short-cuts:

Press Ctrl+Alt+F to get the interactive-window ("REPL-loop"), where you may execute code on-the-fly. You may write code directly to this window and execute it by writing two semicolons before the last enter-key, for example: 

    [lang=fsharp]
    let z = 4;;

Or you can also select multiple lines of code from your F#-file and then send it to the interactive-window (from right mouse click menu, "Execute in interactive", or) with keyboard short-cut Alt+Enter

Usually the best practice is to define initial values for parameters and then send the wanted code-block to interactive.

There are some good unit testing and BDD -frameworks that can be used with F#. But interactive is better for experimental-development so these frameworks are mostly needed to ensure maintainability.

## Type system ##

Type system is for compiler to serve the developer, not for developer to serve the compiler. F# will do a good job to resolve types by itself, but sometimes the developer wants to define types manually:

    [lang=fsharp]
    let f1 x = x+4.5m 
    let f2 (x:decimal) = x+4.5m //argument type
    let f3 x :decimal = x+4.5m  //function type

Generic types (the classic T, T1, T2, ...) have the type syntax of: 'a, 'b, 'c, ...

## Pipes ##

F# has operation "|>", which moves the last parameter of a method to first:

    [lang=fsharp] 
    open System
    "Hello World" |> Console.WriteLine

This can be thought a bit like C# extension methods: if functions are verbs, it is usually more natural to start a sentence with substantive, the language is more natural:

"a dog barks to a human" versus "barks(a dog, to a human)"

## Functions ##

You don't need brackets in function parameters, and comma is reserved for Tuple-class, so you may define a function with parameters like this:

    [lang=fsharp]
    let f x y = x+y

The function type syntax is like C#-language Func<...>-class parameters, e.g.: Func<int, int, string> is a function that takes input two numbers and returns a string. In F#, there is an arrow between parameters, so the corresponding function type syntax is

    int -> int -> string

	
You can often figure out what the function does just by the function type syntax.

Parameter-less function are defined with brackets. You need this when you don't want to execute evaluation immediately. "return"-statement is not usually needed, because the function will return the value of the last call anyway. Functions can locate inside of other functions.

    [lang=fsharp]
    let sayHello () = 
        let hello = 
            let ello = "ello"
            "h"+ello
        let world = 
            "world"
        hello + " " + world

Lambda-function is constructed by the fun-keyword and arrow.


## Lists ##

F# has even more diverse list-manipulation-libraries than the LINQ-library, but you may also use the LINQ-library if you already know that well.

There appears to be three different types of lists widely used, which have mostly the same operations in the libraries. Exaggerate by generalizing, it goes like this:

- List
    - Best maintainability in the code.
    - Immutable: items won't change
    - Two colon signs may be used to match/separate the first item and a list of the rest.
    - At-sign "@" may be used to concatenate/merge two lists.
- Array
    - Best efficiency. Corresponds the C#-array.
    - By default, it is an anti-pattern to point directly at the index. It can be done by writing period-sign before square brackets.
- Seq
    - Best compatibility. This corresponds the C# IEnumerable.

Lists can be converted to other forms: List.toArray, List.toSeq, Seq.toList, Array.toSeq, ... Some code examples:

    [lang=fsharp]
    let emptyList = []

    let listOfLists = [[1;2;3];[4;5;6]]

    let myList = 
        [1..10] |> List.filter(fun i -> i%2=0)
    let mySeq =
        [|1..10|] |> Seq.filter(fun i -> i%2=0)

    let merged = [1;2;3] @ [4;5;6]

    let head::tail = ["h";"t1";"t2";"t3"]

    let ``A and H to Z`` = 'A'::['H'..'Z']

    open System.Linq
    let threeFirst = 
        mySeq.Take(3)
        |> Seq.toArray
        |> Array.map(fun i -> i*3)


You can compose function with ">>" -operator. "First do this then this."

    [lang=fsharp]
    let composed = 
        List.filter(fun i -> i%2=0) >> List.map(fun i -> i+5)
    [6; 5; 3; 2; 4] |> composed

Manual typing can be done either C#-way or OCaml-way:

    [lang=fsharp]
    let emptyList2:list<bigint> = []
    let emptyList3:bigint list = []

## Tuple ##

Besides lists, other common structure is a Tuple: List is n-times one same type of items, tuple is one item of each n separate types. C# also have the Tuple-class and this is the same class.

You could create tuple like C#-way System.Tuple.Create(1, "a", 1.0), but there is a lot easier way, just plain comma-sign. Tuple may be dissembled to pieces, either with fst- and snd-operations, or by defining "temp-variables" in the left side of equality-operator:

    [lang=fsharp]
    let tuple = 1, "a", 1.0
    let a, b, c = tuple

Tuple is marked as times-sign "*" in the type syntax, like: int*string.

## Classes ##

F# is also a great Object-Oriented-language.

Object having one property, email, which has a getter and a setter, and the default value for that is an empty string, would be implemented like this:

    [lang=fsharp]
    type UserData1() = 
       member val Email = "" with get, set

	let myInstance1 = UserData1()

When you create a new instance of an object, the new-keyword is optional.

Object, having constructor with parameter of a readonly-property, email, and also empty constructor (setting empty string as default), would be implemented like this:

    [lang=fsharp]
    type UserData2(email) =
       new() = UserData2("")
       member x.Email = email

Inheritance-syntax is not as handy as in C#, because the type checking needs explicit implementation of interfaces:

    [lang=fsharp]
    type Base() =
        let DoSomething() = ()
        abstract member Method : unit -> unit
        default x.Method() = DoSomething()

    type ``My inherited class with dispose``() =
        inherit Base()
        override x.Method() =
            base.Method()

        interface IDisposable with
            override x.Dispose() = 
                //Dispose resources...
                GC.SuppressFinalize(x)

## Discriminated union ##

You can construct "either-or"-types:

    [lang=fsharp]
    type myEnum =
    | A
    | B

    type CalendarEvent =
    | ExactTime of DateTime
    | MonthAndYear of string*int //month*year

    type Tree =
    | Leaf of int
    | Node of Tree*Tree

One usage of this is the program state management, in compile-time, avoiding the runtime if-logic-jungle. Discriminated unions are very handy with pattern matching.

## Some more particular types ##

Type can be only alias, or "either-or" can have only one case:

    [lang=fsharp]
    type aliasItem<'a,'b> = System.Collections.Generic.KeyValuePair<'a,'b>

    type OrderId = | Id of System.Guid

Type may be a record:

    [lang=fsharp]
    type Address = { 
        Street: string; 
        Zip: int; 
        City: string; 
    }
    let myAddrr = { Street = "Juvank."; Zip = 33710; City = "Tre"; }

...or it could define a compile-time-checking with Measure-attribute. A bit like separate class to capsulate only one value (but exposing/displaying that always), so that different things or unit of measures won't ever mix by accident:
 
    [lang=fsharp]
    [<Measure>]
    type EUR

    [<Measure>]
    type LTC

    let rate =
        let ratePlain = 9.45m //fetch from internet...
        ratePlain * 1m<EUR/LTC>

    let myMoneyInEur (x:decimal<LTC>) = x*rate 

    let converted = myMoneyInEur 7m<LTC>

The easiest way to convert basic-.NET-type to a unit-of-measure is multiplying it by one unit. Aesthetic downside is that this will bring "greater than" and "less than" signs to the source code.

## Pattern matching ##

Pattern-matching is a kind of switch/case or if-elseif-pattern done right, where the condition can't change on on-the-fly, but separate cases don't have the restriction of being constants. There exist several patterns. 

Also, function can directly be a pattern-function or it can use so called "active patterns" to make the intention of the code more clear. Here is the same functionality in multiple ways:

    [lang=fsharp]
    let MyFunction1 a =
       if a = "A" then 1
       elif a = "B" then 2
       else 3

    let MyFunction2 a = 
       match a with
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction3 = fun a -> 
       match a with
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction4 = function
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction5 a = 
       match a with
       | x when x = "A" -> 1
       | x when x = "B" -> 2
       | x -> 3


    let MyFunction6 a = 

        let (|FirstVowel|FirstConsonant|Other|) p =
            match p with
            | "A" -> FirstVowel
            | "B" -> FirstConsonant
            | _ -> Other

        match a with
        | FirstVowel -> 1
        | FirstConsonant -> 2
        | Other -> 3
 
Usually, if possible, it is best practice to leave the default-condition away, because then the compiler will catch mistakes, if unknown conditions are added afterwards to the code.

## Option type: World without Null ##

In C#, the NULL has two meanings: It is a "special case" (Fowler: PoEAA) and also it is un-assigned-variable. Two roles of the same construct makes things messy.

NULL is not used in F# per se. The construct to work as unknown value is option-type:

    [lang=fsharp]
    let echo x = "You have to work " + x.ToString() + " hours."

    let ``working hours`` =
        let day = (int System.DateTime.Now.DayOfWeek)
        match day with
        | 0 | 6 -> None
        | _ -> Some(7.5)
    
    let echoHours1 = 
        match ``working hours`` with
        | None -> ""
        | Some h -> echo h

    let echoHours2 = 
        ``working hours`` 
        |> Option.map(fun h -> echo h)

    let echoHours3 = 
        ``working hours`` 
        |> Option.map(echo)


## Recursion ##

Recursive function is stated as keyword rec. The best practice is to encapsulate the recursion itself inside another function, to hide "useless" initial values. One common practice is to use last parameter as "accumulator", to collect the result of the recursion. The reason is that then the compiler can optimize the stack away with the tail-recursion, so you won't get StackOverflowException even if the recursion would be deep.

    [lang=fsharp]
    let sumItems myList =
        let rec calculate li acc =
            match li with
            |[] -> acc
            |h::t -> calculate t (h+acc)
        calculate myList 0

Of course this typical function is also included in the List-library, but if you would need more complex custom-functionality, then writing explicit recursion is sometimes the right solution.


## Computation Expressions ##

These are kind of capsules to encapsulate side-effects of some functionality. Out-of-the-box you get e.g. seq (for sequences) and async (for asynchronous code), but you can create more by yourself.

The side-effect of asynchrony is that you have to wait. Async.RunSynchronously is not very useful by itself, but Async has a lot of other possibilities also.

The main point is that when you look the code sum = x + y below, it seems very typical F#-code, even if we are in the context of the side-effect. And the side-effect itself, the async, is explicitly stated in the code. The bang (exclamation mark, "!"-sign) syntax means a call to the "computation expression"-interface.

    [lang=fsharp]
    let asyncOperation = Async.Sleep 3000
    
    let waitAndReturn = 
        async {
            do! asyncOperation
            return 3.0
        }
    
    let waitAndReturn2 = 
        async {
            let! x = waitAndReturn
            let y = 7.0
            let sum = x + y
            return sum
        }
    
    let result = Async.RunSynchronously waitAndReturn2

If you compare this to C#, the seq { ... } equivalent is LINQ. But LINQ is kind of a language its own, which is a separate learning curve for the developer. And the developer doesn't always know if (s)he is coding LINQ against IEnumerable, IQueryable, IObservable, or what... A bit naive and C#:ish but working example of seq-expression:

    [lang=fsharp]
    open System.Linq
    let slice mySeq = 
        let chunkSize = 5
        let rec seqSlice (s:int seq) =
            seq{
                yield s.Take chunkSize
                yield! seqSlice (s.Skip chunkSize)
            }
        seqSlice mySeq
    
"Computation Expressions" is another name to the thing aka. "Monad".

## Exercises ##

- Create a recursive program that lists 1000 first numbers of Fibonacci-sequence (1, 1, 2, 3, 5, 8, 13, ...).
- Create a C# unit test or console application to call that program.


## Links ##

If this wasn't complete enough, you might want to [check this material](http://www.tryfsharp.org/Learn).
If you want to practise list-manipulation-libraries, you could do some [Project Euler](https://projecteuler.net/problems) -exercises. If you want some separate code examples of one subject, then [fssnip.net](http://www.fssnip.net/pages/AllTags) is a good site to begin with.


[Back to the menu](../ReadmeEng.html)