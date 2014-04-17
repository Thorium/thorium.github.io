# Actor-model and Agent-model #

When talking about the Actor-model, the term "Actor" refers to verb act as "someone who does something", not "someone who acts like doing". When talking about the Agent-model, the term "Agent" refers also to "someone who does something" or "represents" (like agency), but not "secret agent". Actor-model is not an alternative to domain-model: Actor-model is not actually an abstract data model, but more like a concrete way to implement the program state.

If you want to test speed in interactive, just type:
 
    #time "on";;

## Parallelism ##

There are three types of doing parallelism: 

- Parallel Execution
	- Run the same code path simultaneously in multiple threads.
	- Suits well for parallel set-theory operations for collection of independent items
	- Suits well for CPU-bound resources when a lot of CPU-cycles available.
	- .NET-environment: Task Parallel Library (TPL), .AsParallel()
- Concurrency.  
	- Run separate code paths simultaneously.
	- Suits well when resources are something else than CPU-bound: e.g. I/O or if just "waiting for synchronization".
	- Based on giving away the thread when not using it: Everyone else should also do that!
	- .NET-environment: Async-await
	- Used in message based ("Message Passing") architectures
- Lock the threads and the state
	- No parallelism
	- No synchronization issues
	- .NET: lock
	- Scalability and performance problems

To more concrete example, here are three little programs that enumerate a collection of items through and sleeps a while in every item while enumerating that item:

    [lang=fsharp]
    #if INTERACTIVE
    #time "on"
    #endif

    open System
    open System.Threading
    open System.Threading.Tasks

    let Execution() = 
        let iteri() = 
            [|1..1000|] |> Array.map(
                fun i -> 
                    Thread.Sleep 200 //Lock the thread
                    match i % 250 with
                    | 0 -> Console.WriteLine("Item: " + i.ToString())
                    | _ -> ()
            )
        Console.WriteLine("Starting...")
        let complete = iteri()
        Console.WriteLine("Done")

    let ParallelExecution() = //TPL
        let iteri() = 
            [|1..1000|] |> Array.Parallel.map(
                fun i -> 
                    Thread.Sleep 200
                    match i % 250 with
                    | 0 -> Console.WriteLine("Item: " + i.ToString())
                    | _ -> ()
            )
        Console.WriteLine("Starting...")
        let complete = iteri()
        Console.WriteLine("Done")

    let AsyncParallelExecution() = //Async-await
        let iteri() = 
            [|1..1000|] |> Array.map(
                fun i -> 
                    async {
                        do! Async.Sleep 200
                        return 
                            match i % 250 with
                            | 0 -> Console.WriteLine("Item: " + i.ToString())
                            | _ -> ()
                    }
            )
        Console.WriteLine("Starting...")
        let complete = iteri() |> Async.Parallel |> Async.RunSynchronously
        Console.WriteLine("Done")

The first one is slow. The second one is clearly faster. But it runs out of the threads. The third one is clearly the fastest, because it won't block the threads while executing (non-CPU-bound) async-operation.

Here are corresponding execution times (in interactive, with i7-laptop):

	> Execution();;
	Starting...
	Item: 250
	Item: 500
	Item: 750
	Item: 1000
	Done
	Real: 00:03:20.582, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()
	> 
	
	> ParallelExecution();;
	Starting...
	Item: 250
	Item: 500
	Item: 1000
	Item: 750
	Done
	Real: 00:00:22.266, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()
	
	> AsyncParallelExecution();;
	Starting...
	Item: 1000
	Item: 750
	Item: 500
	Item: 250
	Done
	Real: 00:00:00.204, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()

## Message Queues ##

There are a lot of different kind of MQ-implementation-products. What if some simple message-queue-functionality would be integrated directly to the programming language?

Test this source code with interactive:

    [lang=fsharp]
    let myQueue = 
	    new MailboxProcessor<string>(fun myHandler -> 
	        let rec readNextMessage() =
	            async {
	                let! result = myHandler.Receive()
	                System.Console.WriteLine("Handled: " + result)
	                do! readNextMessage()
	            }
	        readNextMessage())

	myQueue.Post("Hello 1 !")
	myQueue.Post("Hello 2 !")
	
	myQueue.CurrentQueueLength // 2
	
	myQueue.Start()
	
	myQueue.CurrentQueueLength // 0
	
	myQueue.Post("Hello 3 !")


## Actor-model ##

What if "objects" as capturing the program state, would be replaced with message-based solution?

Actor-model is a theory, which uses “Actors” as parallel-computation-items. Actor may process information, store information or communicate with other actors. Actor may even take a message which gives a totally new functionality to that actor.

Actor-model has been implemented to Erlang-language, but lately there have appeared actor-libraries to many programming languages.

The basic idea is comparable to distributed version control system: you can't actually "stop the program and see what state it is in", as the state is constantly-evolving and may look different from a different point of view.


## Agent-model ##


Agent-model is basically the same thing as actor model with small differences. Agent accepts requests, so the interface is more solid. F#-language have built-in support for Agent-model and it is capsulated to the class called MailboxProcessor. Usually this is concretized like this: the master just asks for things and the background thread does the work of parsing the event history with set-theory-operations.

The previous example sent strings to the message-channel. Usually it is a better practice to send discriminated union commands and their parameters.

Static MailboxProcessor.Start is equivalent of creating new MailboxProcessor() and then calling its .Start(). It is easy to construct time-outs, exception handling and cancellation-operations for agents.

Test the following source code:

    [lang=fsharp]
    type ``Person agent methods`` =
    | AddPerson of string * int //Name*Age
    | GetPeople of AsyncReplyChannel<(string*int) list>

    let myQueue2 = 
        MailboxProcessor<``Person agent methods``>.Start(fun myHandler -> 
            let rec readNextMessage people =
                async {
                    let! result = myHandler.Receive()
                    match result with
                    | AddPerson(n,s) ->
                        return! readNextMessage ((n,s)::people)
                    | GetPeople(response) ->
                        let p = people
                        response.Reply(p)
                        return! readNextMessage people
                }
            readNextMessage [])

    let addPerson =  AddPerson >> myQueue2.Post 
    ("Matti", 30) |> addPerson
    ("Maija", 7) |> addPerson 

    let res = myQueue2.PostAndReply(GetPeople)

In addition to PostAndReply there is also AsyncPostAndReply

### We want more... ###

This was just a single agent, which by itself doesn't separate very much from  .NET ConcurrentDictionary<K, T> other than this: The programming interface is business-logical methods and not CRUD-operations. But the real benefits come to life when there are a lot of actors:

Instead of object-inheritance (or enum-type-properties) using agents to communicate with each other.


![](fig1.jpg)

*(image source: [Writing Concurrent Applications Using F# Agents](http://www.developerfusion.com/article/140677/writing-concurrent-applications-using-f-agents/))*


## Exercises ##

#### Exercise 1 ####
Add a new method to "Person agent methods": a method that will return only people over the age of 18 years.

#### Exercise 2 ####
Modify the "Person agent methods" agent to be "unprocessed people" and make also another agent, "processed people":

- Create a small business-logic and communication, how the un-processed people moves to processed people. 
- Change the interface to asynchronous (AsyncPostAndReply)
- You can also encapsulate the agent inside a type.

[Back to the menu](../ReadmeEng.html)