
# OWIN-interface and SignalR-messaging #

[OWIN](http://owin.org/) is an interface, which allows you easily to register components, like:

- WWW-file-server
- REST-Web-interface 
- [SignalR](http://signalr.net/)-communication
- Authentication
- Etc. (in the future, even IIS)

OWIN and SignalR need some infrastructure ceremony to run... 
Let's use the WorkerRole-project and file server component Microsoft.Owin.StaticFiles.

### Technology Choice ###

This practice uses SignalR, which achieves two-way communication. Another typical way would be CRUD-style REST API, which is demonstrated in an OWIN-F#-[code example](https://github.com/dmohl/FsOnTheWeb-Workshop/blob/master/Azure%20Lab/AzureFSharpOwinComplete/WebApiRole/GuitarsApi.fs).

This practice uses WorkerRole and OWIN. Another quite applicable solution would be creating an (C#/VB.NET) ASPNET Web Role, where you can reference F#-library-project. Then you would get some of the IIS-infrastructure on top of that, but may run to some possible background-threading-problems.

## Adding a Start-up-block ##

Select the WorkerRole-project, right mouse click over that, and select Add -> New Item... -> Source File, and add a file MyStartUp.fs to the project, with content:

	[lang=fsharp]
    module MyStartUp

    open Owin
    open Microsoft.AspNet.SignalR

    let hubConfig = HubConfiguration(EnableDetailedErrors = true, EnableJavaScriptProxies = true)

    type MyWebStartup() =
        member x.Configuration(app:Owin.IAppBuilder) =
            //OWIN Component registrations here...

            //SignalR:
            app.MapSignalR(hubConfig) |> ignore            

            //Static files server (Note: default FileSystem is current directory!)
            let fileServerOptions = Microsoft.Owin.StaticFiles.FileServerOptions()
            fileServerOptions.DefaultFilesOptions.DefaultFileNames.Add("index.html")
            //fileServerOptions.FileSystem <- Microsoft.Owin.FileSystems.PhysicalFileSystem(@"c:\wwwroot\")
            app.UseFileServer fileServerOptions
            |> ignore

            ()

    [<assembly: Microsoft.Owin.OwinStartup(typeof<MyWebStartup>)>]
    do()

Move this file in Solution Explorer to the top of WorkerRole.fs, so it will be executed before WorkerRole-file.

### Index.html ###

The code above defines the index.html to be the default start-up-file of the WWW-server.
The default path is the project output-path. So, add to project a TextFile called index.html. From the properties of the index.html modify the "Build Action" to value "Content" and the "Copy to Output Folder" to value "Copy if newer".

![](1-SolutionExplorer.png)

Index.html will contain some kind of content and a pile of JavaScript: SignalR-JavaSctipt-client-side components. But to start with, just write there some kind of test-greeting.
 

## Configuration and the Start-Up ##

Deployment-project has **ServerDefinition.csdef**-file, which contains server settings. At first you may change the WorkerRole-tag attribute vmsize-setting from "Small" to "ExtraSmall". Inside WorkerRole-tag is Imports. The side of Imports, copy a new tag called **Endpoints**. The file should look something like this (of course, depending on your project's name, etc.):

	[lang=xml]
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceDefinition name="FSharpAzure"
	xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" 
	schemaVersion="2013-10.2.2">
	  <WorkerRole name="WorkerRole1" vmsize="ExtraSmall">
	    <Imports>
	      <Import moduleName="Diagnostics" />
	    </Imports>
	    <Endpoints>
	      <InputEndpoint name="Endpoint1" protocol="http" port="80" localPort="80" />
	    </Endpoints>
	  </WorkerRole>
	</ServiceDefinition>

Next, open WorkerRole.fs file. Add inside the class, before methods, this webApp-variable, which is used to free-up resources, and also override for OnStop-method:

	[lang=fsharp]
    let mutable webApp = Unchecked.defaultof<IDisposable>

    override wr.OnStop() =
        if webApp <> Unchecked.defaultof<IDisposable> then
            webApp.Dispose()
        base.OnStop()

Then open wr.OnStart() method and copy before the call base.OnStart() this small source code:

	[lang=fsharp]
    let endpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints.["Endpoint1"]
    let baseUri = sprintf "%s://%A" endpoint.Protocol endpoint.IPEndpoint
    log ("Starting OWIN at " + baseUri) "Information"
    let options = Microsoft.Owin.Hosting.StartOptions()
    options.Urls.Add(baseUri)
    webApp <- WebApp.Start<MyStartUp.MyWebStartup>(options)

This will get the configuration from the ServerDefinition-file, print it to the screen, and start the server. Now when you start the server, you can visit the Azure Compute Emulator UI console (this had separate instructions already in the WorkerRole-practice) to see the address and port where the server did start, and then open that in the web browser.

![](2-StaticFileServer.png)

## SignalR ##

SignalR is dynamic JavaScript based library, which uses C#-dynamic-operator. This is the reason that you have **to add a reference to Microsoft.CSharp** to your WorkerRole-project. (The familiar way, from Solution Explorer over References-folder, right-click and: Add Reference -> Assemblies -> Framework).

In addition, dynamic won't work as such, so add a new file Dynamic.fs and move it to first one, before other fs-files, and copy  [this](https://raw.githubusercontent.com/Thorium/CatVsDog/master/OwinWorkerRole/Dynamic.fs) to its content, the code has been taken from [Fssnip](http://www.fssnip.net/raw/2U)-site.

Let's add a small game as a program, you can test this in the interactive-window:

	[lang=fsharp]
    #if INTERACTIVE
    #r "../packages/FSharp.Data.2.0.7/lib/net40/FSharp.Data.dll"
    #endif    
    open System
    open FSharp.Data
    
    let bank = WorldBankData.GetDataContext()
    let countries = bank.Regions.World.Countries
                    |> Seq.filter(fun i -> i.CapitalCity <> "") 
                    |> Seq.map(fun i -> i.Name, i.CapitalCity)
    let corrects = countries |> Seq.toArray
    let rnd = Random()
    let questionItem() = corrects.[rnd.Next(corrects.Length)]

    let generateQuiz() =
        let correct, capitalCity = questionItem()
        capitalCity, [correct; fst(questionItem()); fst(questionItem())] |> List.sort

    let checkAnswer country capitalCity =
        corrects |> Array.exists(fun (cou, cap) -> cou = country && cap = capitalCity)

Let's also add some helper methods, and one class to model the shape of JSON-traffic:

	[lang=fsharp]
    let preGenerated = [0..500] |> List.map(fun _ -> generateQuiz())
    let takeNth n = preGenerated |> Seq.skip n |> Seq.head

    type QuizItem(quizId)=
        let item = takeNth quizId
        member x.QuizId = quizId
        member x.Capital = item |> fst
        member x.Countries = item |> snd


Then we need a class that will inherit the SignalR-base-class. Let it look like this:

	[lang=fsharp]
    open Microsoft.AspNet.SignalR
    open Dynamic

    type CountryHub() =
        inherit Hub()

        override this.OnConnected() =
            base.OnConnected() |> ignore
            QuizItem(rnd.Next(500)) |> this.Clients.Caller?newQuiz

        member this.GuessCountry(quizzId : int, country : string):unit =
            let capital = takeNth quizzId |> fst
            match checkAnswer country capital with
            | true ->
                this.Clients.Caller?informResult("Correct!") |> ignore
                QuizItem(rnd.Next(500)) |> this.Clients.Caller?newQuiz |> ignore
                //this.Clients.All?informResult("Correct found")
            | false ->
                this.Clients.Caller?informResult("Wrong! Try again!") |> ignore


- GuessCountry-method is called from the JavaScript-client.
- newQuiz and informResult are functions that are defined in the JavaScript, they are called from here. Question mark is the dynamic operator, which comes from the Dynamics.fs-code.

The NuGet setup package has added to WorkerRole-project a Scripts-folder. Because SignalR uses jQuery and SignalR's own JavaScript-client, go to change the "Copy to Output Directory" to value "Copy if newer" to these two files: jquery-1.6.4.min.js and jquery.signalR-2.0.3.min.js

- The file version numbers may change depending on what files the NuGet will fetch. They have to correspond the references of the Index.html-file.

One more thing, you need the content to the Index.html-file:


	[lang=html]
	<!DOCTYPE html>
	<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
	<head>
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	    <meta charset="UTF-8" />
	    <title>SignalR+F-Sharp</title>
	
	    <!--
	    <script type="text/javascript"
	        src="https://code.jquery.com/jquery-1.6.4.min.js"></script>
	    <script type="text/javascript"
	        src="https://ajax.aspnetcdn.com/ajax/signalr/jquery.signalr-2.0.3.min.js"></script>
	    -->
	    <script type="text/javascript" src="Scripts/jquery-1.6.4.min.js"></script>
	    <script type="text/javascript" src="Scripts/jquery.signalR-2.0.3.min.js"></script>
	    <script type="text/javascript" src="/signalr/hubs"></script>
	    <script type="text/javascript">
	        var currentId;
	        var corrects = 0;
	        var wrongs = 0;
	        $(document).ready(function () {
	            $("#correctDisplay").text(0);
	
	            //SignalR Hub:
	            var myHub;
	            $.connection.hub.url = "/signalr";
	            myHub = $.connection.countryHub; //Hub class
	            if (!myHub) console.log("hub not found");
	
	            myHub.client.newQuiz = function (data) {
	                currentId = data.QuizId;
	                question.innerHTML = "Which country has the capital city of " + data.Capital +"?";
	                $("#btn1").val(data.Countries[0]);
	                $("#btn2").val(data.Countries[1]);
	                $("#btn3").val(data.Countries[2]);
	            };
	
	            myHub.client.informResult = function (data) {
	                if (data == "Correct!") $("#correctDisplay").text(++corrects);
	                else alert(data);
	            };
	
	            $.connection.hub.logging = true;
	            $.connection.hub.start().
	                done(function () {
	                    $("#btn1").click(function () {
	                        myHub.server.guessCountry(currentId, $("#btn1").val());
	                        return false;
	                    });
	                    $("#btn2").click(function () {
	                        myHub.server.guessCountry(currentId, $("#btn2").val());
	                        return false;
	                    });
	                    $("#btn3").click(function () {
	                        myHub.server.guessCountry(currentId, $("#btn3").val());
	                        return false;
	                    });
	                });
	        });
	    </script>
	    <style>
	        .bgFrame {
	            background-color: #ffffff;
	            border: thin solid #000000;
	            padding: 30px;
	            margin: 30px;
	            width: 600px;
	            height: 200px;
	            vertical-align: middle;
	            text-align: center;
	        }
	    </style>
	</head>
	
	<body style="background-color: #46c0b8">
	    <div class="bgFrame">
	        <h2>Country Quiz</h2><br />
	        <form method="post" action="Index.html">
	            <div id="question"></div><br />
	            <input type="button" id="btn1" />
	            <input type="button" id="btn2" />
	            <input type="button" id="btn3" />
	            <br /><br /><br />
	            Corrects: <b id="correctDisplay"></b>
	        </form>
	    </div>
	</body>
	</html>


So this is a basic HTML/JavaScript-file. Now, just run the project... (F5)

[Back to the menu](../ReadmeEng.html)