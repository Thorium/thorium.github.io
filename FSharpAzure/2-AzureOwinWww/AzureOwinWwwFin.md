
# OWIN-rajapinta ja SignalR-viestitys. #

[OWIN](http://owin.org/) on rajapinta, johon voi helposti rekisteröidä komponentteja:

- WWW-tiedosto-palvelin
- REST-Web-rajapintaa 
- [SignalR](http://signalr.net/)-kommunikaatio
- Autentikointi
- Yms. (tulevaisuudessa jopa IIS)


OWIN ja SignalR vaativat käynnistyäkseen vähän infrastruktuuri-seremoniaa. 
Käytetään WorkerRole-projektia ja tiedostopalvelimena komponenttia Microsoft.Owin.StaticFiles.

## Käynnistyskoodilohkon lisääminen ##

Valitse WorkerRole-projekti, paina sen päällä hiiren oikeaa nappia, ja valitse Add -> New Item... -> Source File, ja lisää projektille tiedosto MyStartUp.fs

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

Siirrä kyseinen tiedosto Solution Explorerissa WorkerRole.fs-tiedoston päälle, jotta sen suoritus tapahtuu ennen WorkerRole-tiedostoa.

### Index.html ###

Koodissa lisätään index.html www-palvelun oletustiedostoksi.
Oletuspolku palvelulle on projektin output-path. Joten lisää projektiin TextFile nimeltään index.html ja vaihda sen propertiesista "Build Action" arvoon "Content" ja "Copy to Output Folder" arvoon "Copy if newer".

![](1-SolutionExplorer.png)

Index.html tulee sisältämään jonkinlaisen sisällön ja kansan JavaScriptiä: SignalR-javascript-puolen komponentit. Mutta hommaa voi testata, niin voit nyt aluksi vaan kirjottaa sinne jonkinlaisen tervehdyksen.
 

## Konfiguraatio ja käynnistys ##

Deployment-projektissa on **ServerDefinition.csdef**-tiedosto, jossa on palvelimen asetuksia. Aluksi voit muuttaa WorkerRole-tagin attribuutin vmsize-asetuksen asennosta "Small" asentoon "ExtraSmall". WorkerRole-tagin sisässä on Imports. Kopioi Imports:in rinnalle uusi tagi **Endpoints**. Tiedosto näyttää jokseenkin tältä (riippuen tietysti projektisi nimestä, yms.):

	[lang=xml]
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceDefinition name="FSharpAzure"
	xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2013-10.2.2">
	  <WorkerRole name="WorkerRole1" vmsize="ExtraSmall">
	    <Imports>
	      <Import moduleName="Diagnostics" />
	    </Imports>
	    <Endpoints>
	      <InputEndpoint name="Endpoint1" protocol="http" port="80" localPort="80" />
	    </Endpoints>
	  </WorkerRole>
	</ServiceDefinition>

Seuraavaksi avaa WorkerRole.fs:n tiedosto wr.OnStart() -metodi ja kopioi ennen base.OnStart():ia tapahtumaan seuraava koodinpätkä:

	[lang=fsharp]
    let endpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints.["Endpoint1"]
    let baseUri = sprintf "%s://%A" endpoint.Protocol endpoint.IPEndpoint
    log ("Starting OWIN at " + baseUri) "Information"
    let options = StartOptions()
    options.Urls.Add(baseUri)
    webApp <- WebApp.Start<MyStartUp.MyWebStartup>(options)

Tämä hakee konfiguraation ServerDefinition-tiedostosta, tulostaa sen ruudulle, ja käynnistää palvelimen. Nyt kun käynnistät palvelimen, niin voit käydä Azuren Compute Emulator UI:n konsolista katsomassa (tähän oli ohjeet jo WorkerRole-harjoituksessa) mihin osoitteeseen palvelin käynnistyi, ja surffata siihen selaimella.

![](2-StaticFileServer.png)

## Signal-R ##

Tämän osion materiaali ei ole vielä valmis...

[Takaisin valikkoon](../Readme.html)