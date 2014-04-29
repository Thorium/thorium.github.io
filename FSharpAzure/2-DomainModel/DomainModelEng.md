# Domain-modelling, DSL-language #

Domain-model means data model (=structure) for developers, to guide problem solving to some direction. 

Domain Specific Language (DSL) refers in this practices context only to Embedded / Internal style language, where independent functional (combinator-)library will be integrated to part of F#-language. We won't go through Fowler's OO-based DSL-book.

## The Phases of Modelling ##
Modelling can be divided to three phases

1. **Selection/modelling of the primitives**
	- The basic items of the system. 
	- Only the most essential information
2. **Modelling the composition**
	- How to combine primitives so that they fit seamlessly together
	- Combinators 
3. **Syntax**
	- Decorations, so that pieces are nice to use.

## 1. & 2. Modelling the Primitives and the Composition ##

Functionalities (/contract) can be modelled as data or as computation. Regardless of this, the same tooling is in use.

### The Tooling of Modelling ###

Modelling should be done using three available tools:

- **Tuples**
	- "Product type", can be thought as AND-operation
- **Discriminated unions**
	- "Sum type", can be thought as OR-operation
- **Functions**
	- Transition between the states
	- Modelling of an operation

In theory this Boolean-algebra is enough to model everything in information technology. The cleanest domain is achieved using these (i.e. avoid object-oriented-structures), but in practice you sometimes want to use also these:

- **Record**, if there is a lot of bundled data (e.g. "ERP-object")
- **List**, if you don't want to list each single item
- Discriminated union can sometimes *afterwards* be replaced with an interface, if all the members are not known, i.e. some kind of plug-in -architecture is wanted.

### Domain-model Modelling (Contract as Data) ###

Let's take an example, simple stocks exchange trading.

How would you model the trades?

- I want to buy 100 Microsoft stocks.
- I want to buy 500 Nokia stocks.
- I want to sell 300 Google stocks.

Primitives would be the single trades. Composition is combining those to some kind of abstract syntax tree.

Usually the best practice is to model functionality (commands, verbs) rather than basic-objects (nouns). The model could look for example like this:

    [lang=fsharp]
    module ``OptionTrade case 1`` =

        //Primitives:
        type OptionTrade = 
        | Buy of string*decimal // Buy "MSFT" 100 (amount)
        | Sell of string*decimal
        //Composition combinators:
        | ContractUntil of System.DateTime*OptionTrade
        | ContractAfter of System.DateTime*OptionTrade
        | Combine of OptionTrade*OptionTrade //or OptionTrade list

Or like this:

    [lang=fsharp]
    module ``OptionTrade case 2`` =

        //Primitives:
        type OperationKind = Buy | Sell
        type DateTimeKind = Until | After
        type OptionTrade = 
        | Operation of OperationKind*string*decimal
        //Composition combinators:
        | Contract of DateTimeKind*System.DateTime*OptionTrade
        | Combine of OptionTrade*OptionTrade

The main difference between these two is that when these are later used, how easy is it to access the parameter data:

    [lang=fsharp]
    module ``OptionTrade case 1 usage`` =
        open ``OptionTrade case 1``
        
        let create = 
            Combine(
                Combine(
                    Buy("MSFT",100m),
                    Buy("NOK",500m)),
                Sell("GOOG",300m))

        let rec purge = 
            function
                | Buy (name,amount) -> "" //...
                | Sell (name,amount) -> "" //...
                | ContractUntil (dt,opt) -> 
                     if dt<=System.DateTime.Now then purge opt else "" //...
                | ContractAfter (dt,opt) ->
                     if dt>=System.DateTime.Now then purge opt else "" //...
                | Combine (a,b) -> purge a + purge b

    module ``OptionTrade case 2 usage`` =
        open ``OptionTrade case 2``
                
        let create = 
            Combine(
                Combine(
                    Operation(Buy,"MSFT",100m),
                    Operation(Buy,"NOK",500m)),
                Operation(Sell,"GOOG",300m))
    
        let rec purge = 
            function
                | Operation (kind,name,amount) -> // you could use when or...
                    "" // use common name and amount functionality and then 
                       // match kind with buy/sell when needed
                | Contract (dtk, dt,opt) -> "" // match dt with...purge(opt)
                | Combine (a,b) -> purge a + purge b

Usually the better solution is the model 2, if the parameters describe the same data content, like in here. Then the operation is easier to change from one form to other, e.g. Buy -> Sell. Rather make more types than few mega-types.

This is the easiest and most clean way to model, and this way you will get a clear data model. The downside is that operations are solid (/"hard-coded"), so you can't compose your own structures to do whatever, like a library: the functionality has to obey the data model functionality. Usually this is enough, when your goal is to build simple little system that has clear boundaries.

When C# code so often leads to object-mapping-code from type to another, comes [F# Object Expressions](http://msdn.microsoft.com/en-us/library/dd233237.aspx) to rescue, you can create new instances without boilerplate-code:

    [lang=fsharp]
    type MyInvoice = { 
        Sum : decimal;
        Tax : decimal;
        //... 
        Property01 : string;
        Property02 : string;
        Property03 : string;
        //...
        Property99 : string;
    }
    let myInstance = { Sum=10m; Tax=10m; Property01="a"; Property02="b"; Property03="c"; Property99="zzz" }
    // Don't have to define all copied propertis like in LINQ-select you would have to:
    let myNewCopy = { myInstance with Property01="Hello!" }


### Modelling Contract as Computation ###

For a normal program this is too heavy concept (if you aren't familiar with call-cc and Haskell-libraries), but one good option, if it is a must to try to construct some kind of custom work-flow/process/rule -engine or framework. The difference to the previous one is that the user can refer this as a library and make more custom composition-operations. In that case this can be thought as a combinator-library or a DSL-language.

The model may be modelled as an operation/function `('a -> M<'b>)`, where "a" is "the program state", i.e. against what is the combination rules made, and M describes some kind of capsule/monad (for example a list), and "b" is the result type. The capsule is optional and also "a" and "b" don't have to be generics, if more precise business-oriented types are possible to define. At first, let's give a name to the function, just to avoid drowning into arrows:

    [lang=fsharp]
    type OptionTrade<'a,'b> =
    | Operation of ('a -> List<'b>)

Now you may create a function that compose the functionality, e.g. with list:
    
    [lang=fsharp]
    let combine (Operation f1) (Operation f2) = 
        Operation(fun a -> [f1(a); f2(a)])
    
    // OptionTrade -> OptionTrade -> OptionTrade

and define yet another function, the execution itself:

    [lang=fsharp]
    let eval (Operation f) a = f(a)

The user may now create whatever kind of Operation-style functions, that implement the syntax `(a -> List<b>)` and these can be combined as the user likes.
For example:

    [lang=fsharp]
    let buy name amount = Operation(fun a -> [(name, amount)])

Finally the user calls the execution of the program.

The code logic is not separately developed, but the library already executes the code logics. So the composition-functions in the source code form the abstract syntax tree:
  
    [lang=fsharp]
    let myCombination = 
        combine
            (buy "MSFT" 100m)
            (buy "NOK" 500m)
    let myDone = eval myCombination "now!"


Here the code is overly generic. Sell-operation is left as an exercise.
Practical code sample (where is also the sell-operation and a pile of others) can be downloaded from the [internet](http://archive.msdn.microsoft.com/realworldfp/Release/ProjectReleases.aspx?ReleaseId=3674), code samples of the book ["Real-World Functional Programming"](http://www.manning.com/petricek/), chapter 15.

#### Standard-operations ####

When different functionalities are combined (even between different libraries), is is often handy to use the familiar standard-operations like:

- map `('T -> 'R) -> M<'T> -> M<'R>`
- bind `('T -> M<'R>) -> M<'T> -> M<'R>`
- return `'T -> M<'T>`

For example, map would be like this (with one and two parameters):

    [lang=fsharp]
    //Map with one parameters, just basic composition:
    let map f (Operation f1) = Operation(fun a -> f(f1(a)))
    //Map with f having two parameters:
    let map2 f (Operation f1) (Operation f2) = Operation(fun a -> f (f1 a) (f2 a))

Avoid side-effects. Operation functionality can be figured out from the type syntax.

## 3. Syntax ##

F# has multiple features to make your functionality more convenient to use: Custom operators, Builder-syntax, Quotations (like C# -expression-tree), Query-syntax, ...
Let's take a look at the first two:

### Custom Operators ###

You can make your own [operators](http://msdn.microsoft.com/en-us/library/dd233204.aspx), like a normal function, but writing the operator to parenthesis. When you use operator, the first parameter will come before the operator. For example this "funny trick":

    [lang=fsharp]
    let (+) x y = x-y
    let minusOne = 5+6

...but this will have really nice and useful applications in custom DSL-languages, for example the previous combine-function (we don't need to list parameters due to partial application):

    [lang=fsharp]
    let (&) = combine

    let myCombination2 = 
        buy "MSFT" 100m &
        buy "NOK" 500m &
        sell "GOOG" 300m

    let myDone2 = eval myCombination2 "now!"

Overloading operators can be done also as usual member-functions to types. In F# you can also do extension methods (and extension properties!):

    [lang=fsharp]
    type System.String with
        member x.yell = x + "!"
    // "hello".yell

### Builder-syntax ###

You may build your own computational expressions: You just have to pick your state/side-effect that you want to encapsulate. Inside this the programming is with syntax `myContext{ ... }` where "myContext" is almost any word.

    [lang=fsharp]
    // Return class like Async<T> or IEnumerable<T>:
    type MyReturnClass(n:int) =
        member x.Value = n

    type MyContextBuilder() =        
        member t.Return(x) = MyReturnClass(x)
        member t.Bind(x:MyReturnClass, rest) = 
            printfn "Binded %d" x.Value 
            rest(x.Value)

    let context = MyContextBuilder()

    let test =
        context{
            let! a = MyReturnClass(3) //"let!" calls builder's Bind
            let! b = MyReturnClass(5) //"let!" calls builder's Bind

            //Inside the monad you program like usual F#:
            let mult = a * b   
            let sum = mult + 1 

            return sum //"return" calls builder's Return(x)
        }

- Inside the context, the bang (exclamation mark) -commands are ("syntactic sugar") calls to the corresponding methods of the builder-"interface".
- "Interface" doesn't have to be fully covered, just the methods you like. The system is based on continuation: call-cc and reify.
- Detailed description of the interface and how it works is available:
    - [Interface](http://msdn.microsoft.com/en-us/library/dd233182.aspx)
    - [Additional information](http://blogs.msdn.com/b/dsyme/archive/2007/09/22/some-details-on-f-computation-expressions-aka-monadic-or-workflow-syntax.aspx)


## Exercises ##

### Exercise 1 ###

This is some kind of fictional loan-application-process: [pdf](loanappprocess.pdf).

Here is a quick & dirty version of copy & pasting the texts to data types:

    [lang=fsharp]
    type Terms = string
    type Loan = 
    | Offer of decimal*DateTime
    | Approved of decimal*DateTime

    type ``Loan application state`` =
    | ``Status Received``
    | ``Has Client`` of bool
    | ``Has Credit`` of bool
    | ``Offer terms are ok`` of Terms*bool
    | ``Loan rejected``
    | ``Manual offer terms`` of Terms
    | ``Offer created`` of Terms*Loan
    | ``Offer approved`` of Terms*Loan*bool
    | ``Offer closed`` of Terms*Loan
    | ``Money transferred`` of Loan
    | ``Loan fully paid`` of bool*Loan
    | ``Report created`` of Loan

- How would you optimize these data types?
- Write also a few fictional methods to use this process and transfer the loan application from a process state to another.
- The data not involved to the process is better to encapsulate later as a new tuple beside than inside the process.

### Exercise 2 ###

Think what would be a bad domain-model?

### Exercise 3 ###

The chapter "Modelling Contract as Computation" did left the sell-operation away. Sell could either be a negation or a new custom type of `type OperationKind = Buy | Sell`.

Because there is only one sell-operation and combine-function takes several parameters, there is a little need for code changes, e.g. wrapper-operation to the list and maybe a custom operator for the wrapper:
		
    [lang=fsharp]
    let ``return`` (Operation f1) = Operation(fun a -> [f1(a)])

Implement the sell-operation and try to get it working in the original example.

This is how you can declare your own functions:

    [lang=fsharp]
    //Example use, with list-parameter, one parameters:
    let doubleTradeAmount = map (fun al -> [fst(al |> List.head),snd(al |> List.head)*2m])
    let goneDouble = doubleTradeAmount (buy "MSFT" 100m)
    eval goneDouble ("not-used-initial-state",0m)

As you can see, the list-context is useless here and causes just extra List.head-calls. You can try to remove the list from the computations.

Create a custom function using map2-function.


## Links / Sources ##

Tomas Petricek - Domain Specific Languages in F# - [Video](http://vimeo.com/60546839), [Slides](http://www.slideshare.net/tomaspfb/dsls) 

Simon Peyton-Jones - [Composing contracts: an adventure in financial engineering](http://research.microsoft.com/en-us/um/people/simonpj/papers/financial-contracts/contracts-icfp.htm) 

Lab49 - [The Algebra of Data, and the Calculus of Mutation](http://blog.lab49.com/archives/3011)

Philip Wadler - [Theorems for free!](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf)

[Back to the menu](../ReadmeEng.html)

