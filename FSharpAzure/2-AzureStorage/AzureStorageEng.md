
# Azure Blob- and Table Storage #

Let's use the WorkerRole from previous tutorials. Let's also use [Fog](http://dmohl.github.io/Fog/)-library, which supports Azure services: Table Storage, Blob Storage, Queue Storage, Service Bus and Cache.

First task is to add connection strings to Azure storage. It is done like this:

Open the deployment-project file **ServiceDefinition.csdef** and add there settings: **TableStorageConnectionString** and **BlobStorageConnectionString**. In the Azure emulator -environment these can be empty. 

File content attribute `name="FSharpAzure"` refers to the name of the solution and  `schemaVersion="2013-10.2.2"` refers to the used AzureSDK, here 2.2, but 2.3 would be  `schemaVersion="2014-01.2.3"`.


	[lang=xml]
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceDefinition name="FSharpAzure" 
	xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition"
	schemaVersion="2013-10.2.2">
	  <WorkerRole name="WorkerRole1" vmsize="ExtraSmall">
	    <Endpoints>
	      <InputEndpoint name="Endpoint1" protocol="http" port="80" localPort="80" />
	    </Endpoints>
	    <Imports>
	      <Import moduleName="Diagnostics" />
	    </Imports>
	    <ConfigurationSettings>
	      <Setting name="TableStorageConnectionString" />
	      <Setting name="BlobStorageConnectionString" />
	    </ConfigurationSettings>    
	  </WorkerRole>
	</ServiceDefinition>

Modify also the files **ServiceConfiguration.Cloud.cscfg** and **ServiceConfiguration.Local.cscfg** to contain the corresponding settings:
	
	[lang=xml]
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceConfiguration serviceName="FSharpAzure" 
	xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration"
	osFamily="4" osVersion="*" schemaVersion="2013-10.2.2">
	  <Role name="WorkerRole1">
	    <Instances count="1" />
	    <ConfigurationSettings>
	      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" 
		  value="UseDevelopmentStorage=true" />
	      <Setting name="TableStorageConnectionString" value="UseDevelopmentStorage=true" />
	      <Setting name="BlobStorageConnectionString" value="UseDevelopmentStorage=true" />
	    </ConfigurationSettings>
	  </Role>
	</ServiceConfiguration>

For emulator, it is OK to use value="UseDevelopmentStorage=true", but in the production this connection-string should be something else. For this, there is tutorial in the [net](http://msdn.microsoft.com/library/azure/ee758697.aspx).

### Using F-Sharp script-files ###

Using these codes in interactive-environment works, but execution of the actual Azure-calls won't. So you may add to beginning of the file:

    [lang=fsharp]
    #if INTERACTIVE
    #r "../packages/Fog.0.1.4.1/Lib/Net40/Fog.dll"
    #r "System.Data.Services.Client.dll"
    #r "Microsoft.WindowsAzure.StorageClient.dll"
    #endif

Note that NuGet may have fetched you different version of the component than in this example. So if Visual Studio underlines the line with red, and says the file doesn't exist, this may be the result of you having newer version of the component, in another path. 
Another commonly used practice in F# is to add one file of type Script File (*.fsx) to the project. This runs only in interactive-style scripting but will be excluded from the actual compilation. These #r:s works also there and project's  .fs-files could be loaded like this: 

    [lang=fsharp]
    #load "MyLogics.fs"
    open MyLogics

 
## Azure Blob Storage ##

Blob-storage is a simple data storage e.g. for files. Open the logics file and add on this code to it:

    [lang=fsharp]
    module MyLogics
    
    open Fog.Storage.Blob
    
    let containerName = "testContainer"
    let blobFileName = "testBlob"
    let blob = GetBlobReference containerName blobFileName
    
    let addToBlob text = 
        text |> blob.UploadText

    open System
    open FSharp.Data
    let demoData =
        let ``scientist of the day `` = 
            FreebaseData.GetDataContext().Commons.Computers.``Computer Scientists``
            |> Seq.skip DateTime.Now.DayOfYear //
            |> Seq.head
        "Scientist of the day: " + ``scientist of the day ``.Name

    
Now you may call this from the WorkerRole.fs file's method wr.OnStart():

    [lang=fsharp]
    MyLogics.demoData |> MyLogics.addToBlob

When you start the program (F5), you may find from **Server** Explorer (not Solution Explorer) that (after refresh) there has been appeared a blob which you can double-click to open, and there is the blob-list that you can again double-click to open the data itself:

![](1-ServerExplorer.png)

## Azure Table Storage ##

Azure Table Storage is NoSQL-minded data storage.

Here is a code sample to use it (tested on AzureSDK 2.2, but the new version seems to have some issues):

    [lang=fsharp]
    let ``Azure dvd table`` = "Dvd"

    [<Measure>]
    type stars

    open System
    open System.Data.Services.Common

    [<DataServiceKey("PartitionKey", "RowKey")>] 
    type MyDvdEntity() = 
        member val PartitionKey = String.Empty with get, set
        member val RowKey = String.Empty with get, set
        member val Timestamp = DateTime.UtcNow with get, set
        member val Name = String.Empty with get, set
        member val Rating = 0<stars> with get, set

    open Fog.Storage.Table

    let addDvd name rating = 
            MyDvdEntity(
                PartitionKey = "myPartition",
                RowKey = System.Guid.NewGuid().ToString("N"),
                Name = name,
                Rating = rating
            ) |> CreateEntity ``Azure dvd table``

    let updateDvd dvd = UpdateEntity ``Azure dvd table`` dvd
    let deleteDvd dvd = DeleteEntity ``Azure dvd table`` dvd

The actual Table Storage [query language is very limited](http://msdn.microsoft.com/en-us/library/windowsazure/dd135725.aspx). Fog doesn't provide handy interface for that, so you have to use the Azure-interface directly: Add reference to **Azure-SDK's Microsoft.WindowsAzure.StorageClient.dll** (which is located by default in the path: C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\v2.2\bin\Microsoft.WindowsAzure.StorageClient.dll ).
After the previous example, you can add this to execute a query:

    [lang=fsharp]
    open Microsoft.WindowsAzure.StorageClient

    let tableClient = BuildTableClient()
    ``Azure dvd table`` |> tableClient.CreateTableIfNotExist |> ignore

    let getUsers() = 
        let context = tableClient.GetDataServiceContext()
        let query = 
            query { for item in context.CreateQuery<MyDvdEntity>(``Azure dvd table``) do
                    select item }
        // query makes IQueryable, so you can use System.Linq if you want.
        query.AsTableServiceQuery().Execute()

 

[Back to the menu](../ReadmeEng.html)