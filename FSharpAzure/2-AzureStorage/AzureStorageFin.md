
# Azure Blob- ja Table Storage #

Käytetään edellä tehtyä WorkerRole-projektia. Käytämme myös [Fog](http://dmohl.github.io/Fog/)-kirjastoa, jossa on tuki Azuren palveluille: Table Storagelle, Blob Storage, Queue Storage, Serice Bus ja Cache.

Ensimmäisenä on lisättävä connectionstringit Azuren storageille. Tämä tapahtuu näin:

Avaa deployment-projektin **ServiceDefinition.csdef** ja lisää sinne settingit: **TableStorageConnectionString** ja **BlobStorageConnectionString**. Nämä voivat emulator-ympäristössä olla tyhjät.

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

Muokkaa myös tiedostoja **ServiceConfiguration.Cloud.cscfg** ja **ServiceConfiguration.Local.cscfg** sisältämään vastaavat settingit:
	
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

Emulaattorille kelpaa arvo  value="UseDevelopmentStorage=true", mutta tuotannossa tämä connection-string on jotain muuta. Ohjeet siihen löytyy [netistä](http://msdn.microsoft.com/library/azure/ee758697.aspx).

### F-Sharp skripti-tiedostojen käyttö ###

Näiden koodien ajaminen toimii interactive-ympäristöstä tiettyyn pisteeseen asti, mutta itse Azuren kutsukoodien suoritus ei. Voit siis lisätä koodien alkuun:

    [lang=fsharp]
    #if INTERACTIVE
    #r "../packages/Fog.0.1.4.1/Lib/Net40/Fog.dll"
    #r "System.Data.Services.Client.dll"
    #r "Microsoft.WindowsAzure.StorageClient.dll"
    #endif

Toinen F#:ssa tyypillinen tapa on lisätä projektiin yksi tiedosto tyyppiä Script File (*.fsx), joka suoritetaan vain interactive-tyylisessä scriptaus-ajoissa, mutta jää itsestään käännöksen ulkopuolelle. Nämä #r:t toimivat myös siellä, ja projektin .fs-tiedoston voi ladata käskyillä: 

    [lang=fsharp]
    #load "MyLogics.fs"
    open MyLogics

 
## Azure Blob Storage ##

Blob-storage on simppeli tietovarasto esim. tiedostoja varten. Avaa logiikkaluokka ja lisää siihen seuraava koodi:

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

    
Nyt voit lisätä kutsun tähän WorkerRole.fs:n wr.OnStart() -metodissa:

    [lang=fsharp]
    MyLogics.demoData |> MyLogics.addToBlob

Kun ajat softan (F5), niin **Server** Explorer:iin (ei Solution Explorer) on (refresh:in jälkeen) ilmestynyt seuraava blobi, jonka voit tupla-klikata auki, jonka blob-listasta voit taas tupla-klikata itse tiedot auki:

![](1-ServerExplorer.png)

## Azure Table Storage ##

Azure Table Storage on NoSQL-henkinen tietovarasto.

Ohessa koodiesimerkki sen käyttöön:

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

Varsinainen Table Storage:n [kyselykieli on varsin suppea](http://msdn.microsoft.com/en-us/library/windowsazure/dd135725.aspx). Sen käyttöön Fog-kirjasto ei tarjoa kätevää rajapintaa, joten joudut tekemään asian suoraa Azuren omaa rajapintaa vasten: Referoi **Azure-SDK:n Microsoft.WindowsAzure.StorageClient.dll** (joka löytyy oletuksena polusta: C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\v2.2\bin\Microsoft.WindowsAzure.StorageClient.dll ).
Edellisen perään lisättynä, tässä mallikoodi kyselyn tekoon:

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

 

[Takaisin valikkoon](../Readme.html)