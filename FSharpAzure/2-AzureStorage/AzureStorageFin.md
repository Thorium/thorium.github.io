
# Azure Blob- ja Table Storage #

Käytetään edellä tehtyä WorkerRole-projektia. Käytämme myös Windows Azure Storage -kirjastoa. (Aiemmassa versiossa tätä ohjetta käytettiin Fog-kirjastoa, mutta se ei toiminut hyvin uusimpien Azure SDK:iden kanssa.)

Ensimmäisenä on lisättävä connectionstringit Azuren storageille. Tämä tapahtuu näin:

Avaa deployment-projektin **ServiceDefinition.csdef** ja lisää sinne settingit: **TableStorageConnectionString** ja **BlobStorageConnectionString**. Nämä voivat emulator-ympäristössä olla tyhjät. 

Tiedostojen sisällössä attribuutti `name="FSharpAzure"` viittaa solutionin nimeen ja `schemaVersion="2013-10.2.2"` käytettyyn AzureSDK:iin, tässä 2.2, mutta 2.3 olisi `schemaVersion="2014-01.2.3"`ja 2.4 olisi `schemaVersion="2014-06.2.4"`.

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

Näissä on huomattava se, että NuGet on voinut hakea sinulle eri version komponentista kuin mitä tässä esimerkissä. Joten jos Visual Studio alleviivaa rivin punaisella ja valittaa, että tiedostoa ei löydy, niin tämä voi johtua siitä, että sinulla on eri polussa uudempi versio siitä.
Toinen F#:ssa tyypillinen tapa on lisätä projektiin yksi tiedosto tyyppiä Script File (*.fsx), joka suoritetaan vain interactive-tyylisessä scriptaus-ajoissa, mutta jää itsestään käännöksen ulkopuolelle. Nämä #r:t toimivat myös siellä, ja projektin .fs-tiedoston voi ladata käskyillä: 

    [lang=fsharp]
    #load "MyLogics.fs"
    open MyLogics

## Välimuisti ja asetusten haku ##

Tässä koodi, joka toimii sekä Blob:in että Table:n yhteydessä.
Avaa logiikka"luokka" (=tiedosto) ja lisää siihen seuraava koodi:

    [lang=fsharp]
    module MyLogics
    
    open System
    open Microsoft.WindowsAzure.ServiceRuntime
    open Microsoft.WindowsAzure.Storage

    open System.Collections.Concurrent
    let dict = ConcurrentDictionary() //for caching

    let fetchSetting = RoleEnvironment.GetConfigurationSettingValue >> CloudStorageAccount.Parse

Käytetään memoize:a välimuistitukseen. Memoize/memoization poikkeaa normaalista cache-välimuistista siten, että välimuistitetaankin funktio, eikä vain sen parametreja.
 
## Azure Blob Storage ##

Blob-storage on simppeli tietovarasto esim. tiedostoja varten. Lisää edellisen perään seuraava toiminnallisuus:

    [lang=fsharp]
    let blobConnection (container:string) = 
        let account = fetchSetting "BlobStorageConnectionString"
        let client = account.CreateCloudBlobClient()
        client.GetContainerReference(container.ToLower())

    let uploadBlobToContainer containerName blobName (item:string) = 
        let memoize f = fun x -> dict.GetOrAdd(Some x, lazy (f x)).Force()
        let container = memoize(fun cont -> blobConnection cont) containerName
        async {
            let! ok = container.CreateIfNotExistsAsync() |> Async.AwaitTask
            let blob = container.GetBlockBlobReference(blobName)
            let enc = System.Text.Encoding.ASCII.GetBytes(item)
            use ms = new System.IO.MemoryStream(enc, 0, enc.Length)
            do! blob.UploadFromStreamAsync ms |> Async.AwaitIAsyncResult |> Async.Ignore
        }
    
    let addToBlob = uploadBlobToContainer "testContainer" "testBlob"

    open System
    open FSharp.Data
    let demoData =
        let ``scientist of the day `` = 
            FreebaseData.GetDataContext().Commons.Computers.``Computer Scientists``
            |> Seq.skip DateTime.Now.DayOfYear //
            |> Seq.head
        "Scientist of the day: " + ``scientist of the day ``.Name

    
Nyt voit lisätä kutsun tähän WorkerRole.fs:n wr.OnStart() -metodissa (ennen kutsua base.OnStart()):

    [lang=fsharp]
    MyLogics.demoData |> MyLogics.addToBlob |> Async.RunSynchronously

Kun ajat softan (F5), niin **Server** Explorer:iin (ei Solution Explorer) on (refresh:in jälkeen) ilmestynyt seuraava blobi, jonka voit tupla-klikata auki, jonka blob-listasta voit taas tupla-klikata itse tiedot auki:

![](1-ServerExplorer.png)

## Azure Table Storage ##

Azure Table Storage on NoSQL-henkinen tietovarasto.

Ohessa koodiesimerkki sen käyttöön:

    [lang=fsharp]
    let ``Azure dvd table`` = "Dvd"

    [<Measure>]
    type stars

    open Microsoft.WindowsAzure.Storage.Table

    type MyDvdEntity(partitionKey, rowKey, name, rating) = 
        inherit TableEntity(partitionKey, rowKey)
        new(name, rating) = MyDvdEntity("defaultPartition", System.Guid.NewGuid().ToString(), name, rating)
        new() = MyDvdEntity("", 0<stars>)
        member val Name = name with get, set
        member val Rating = rating with get, set

    let tableConnection tableName = 
        let account = fetchSetting "TableStorageConnectionString"
        let client = account.CreateCloudTableClient()
        client.GetTableReference(tableName)

    let doAction tableName operation = 
        let memoize f = fun x -> dict.GetOrAdd(Some x, lazy (f x)).Force()
        let table = memoize(fun tb -> tableConnection tb) tableName
        async {
            let! created = table.CreateIfNotExistsAsync() |> Async.AwaitTask
            return! table.ExecuteAsync(operation) |> Async.AwaitTask
        }

    let addDvd name rating = 
        let dvd = MyDvdEntity(
                    PartitionKey = "myPartition",
                    RowKey = System.Guid.NewGuid().ToString(),
                    Name = name,
                    Rating = rating
                  )
        dvd, dvd
        |> TableOperation.Insert //Insert, Delete, Replace, etc.
        |> doAction ``Azure dvd table``
        |> Async.RunSynchronously

    let updateDvd dvd = 
        dvd |> TableOperation.Replace |> doAction ``Azure dvd table`` |> Async.RunSynchronously

    let deleteDvd dvd = 
        dvd |> TableOperation.Delete |> doAction ``Azure dvd table`` |> Async.RunSynchronously

...ja vastaavasti tätä kutsutaan WorkerRole.fs-tiedostosta, kutsu OnStart-metodiin, ennen base.OnStart:ia, esim:

    [lang=fsharp]
    let dvd, result = MyLogics.addDvd "Godfather" 4<stars>
    dvd.Rating <- 5<MyLogics.stars>
    let result2 = MyLogics.updateDvd dvd
	
Varsinainen Table Storage:n [kyselykieli on varsin suppea](http://msdn.microsoft.com/en-us/library/windowsazure/dd135725.aspx), eikä uusi Table Service Layer tue LINQ:a, joten kyselyt pitää tehdä joko TableOperation.Retrieve-metodin kautta tai rakentamalla erikseen TableQuery ja suorittamalla se ExecuteQuery-metodilla. 

[Takaisin valikkoon](../Readme.html)