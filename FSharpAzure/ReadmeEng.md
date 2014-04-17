
# F# and Azure in Practice #

The event is in Finnish only, but these pages are a translation of the materials used. So this is just a quick translation, not a perfect F#-tutorial. Original [Finnish version](Readme.html).


There are actually 2 levels of exercises and they are not in any particular order.

## Initial Exercises ##

It is better to start here, if you are not familiar with the topic.

- [**Creating an Azure Worker**](1-AzureWorkerRole/AzureWorkerRoleEng.html)
	- Here we create an Azure Worker role, which is used as a template in other exercises.
- [**F# in Brief**](1-FSharpBasics/FSharpBasicsEng.html)
	- Some basics of F#-language syntax. 

## Advanced Exercises ##

These are very short introductions to larger-scale topics.

- [**Information Rich Programming Language**](2-DataUsage/DataUsageEng.html)
	- It is easy to use any data source in F# and start to develop using it. The compiler compiles the data model as part of the language and IDE.
- [**Azure Blob- and Table Storage**](2-AzureStorage/AzureStorageEng.html)
	- Using Azure storages from the F#-language.
- [**Actor-model and Agent-model**](2-AgentModel/AgentModelEng.html)
	- Model to implement the program state so that it is distributed: no locks but message passing. You can't stop the program and check "what state it is in"; the state is constantly changing and will look different from the different point of views.
- [**Domain-modelling, DSL-language**](2-DomainModel/DomainModelEng.html)
	- DSL-language: the king of the workflow/process/rule -engines and frameworks.
	- Selecting the primitives, composition and syntax
- [**OWIN-interface and SignalR-messaging**](2-AzureOwinWww/AzureOwinWwwEng.html)
	- OWIN is a middleware-interface to the server, in which you may register components like www-hosting-service.
	- SignalR will take care of two-way communication channel between the server and the client.

## Independent Exercises ##

Some well-known existing references of F# usage areas are:

- Stock/Commodities Trading 
- Social Gaming
- Cloud Computing

But F# is a general purpose language, so you can code what you want. :-)

----------

*These exercises start from the scratch. Still, if you have network problems or something, you may download an example-VS2013-solution from [here](FSharpAzure.zip).*
