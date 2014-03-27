
# Azure Worker -roolin luonti #

Oletetaan, että koneellasi on Visual Studio 2012 tai Visual Studio 2013 ja siihen asennettuna oletuksena mukana tuleva F#-kielen tuki. Lisäksi olet asentanut myös [Windows Azure SDK](http://www.windowsazure.com/en-us/downloads/):n.

Avaa Visual Studio ja valitse valikosta: File -> New -> Project

Kirjoita hakukenttään **azure**

Voit valita "Windows Azure Cloud Service". Älä hämäänny kielivalinnasta, voit valita kumman tahansa. Keksi projektillesi nimi ja paina ok.

![](http://thorium.github.io/FSharpAzure/AzureWorkerRole/1-NewProject.png)

Aukeaa uusi dialogi, valitse siitä **Visual F#**

Valitse Worker Role ja **lisää se nuolesta listaan**, vasta tämän jälkeen paina ok. (Jos vain suoraa ok, tulee tyhjä "solution".)

![](http://thorium.github.io/FSharpAzure/AzureWorkerRole/2-WorkerRole.png)

Nyt sinulla pitäisi olla luotu uusi "solution" ja siinä kaksi projektia:

 1. Valitsemasi niminen projekti (pilvikuvakkeella), joka on ns. deployment-projekti: Jos valitset tämän ja painaisit hiiren oikeaa nappia, tulee valikko, jossa on mm. "Publish"-nappi, josta projekti julkaistaan Windows Azureen.
 2. WorkerRole-projekti (F#-kuvakkeella), joka on itse lähdekoodiprojekti. Tänne on oletuksena tehty WorkerRole.fs-niminen koodiluokka. Kyseinen luokka ei tee muuta, kuin käynnistyessään kirjoittaa konsolille 10 sekunnin välein tekstin "Working".

![](http://thorium.github.io/FSharpAzure/AzureWorkerRole/3-SolutionExplorer.png)

Hyvä, sitten vain ohjelma käyntiin. Mutta tässä vaiheessa on vielä turha lähettää ohjelmaa nettiin: sen voi ensiksi ajaa emulaattorilla. (Emulaattori toimii "aika realistisesti", eli kannattaa kuitenkin aika-ajoin joskus julkaista myös Azureen, eikä sokeasti luottaa emulaattoriin.)

Emulaattorin saa käyntiin, kun joko valitsee valikosta Debug -> Start debugging, tai painaa **F5**. Ohjelma lataa hetken ja emulaattorin kuvake ilmestyy Windowsin oikean alakulman SysTray-palkkiin. Paina emulaattorin **kuvakkeen päällä hiiren oikeaa nappia**, ja valitse "Show Compute emulator UI".

![](http://thorium.github.io/FSharpAzure/AzureWorkerRole/4-Systray.png)

Nyt aukeaa emulaattorin käyttöliittymä. Avaa vasemman puolen listasta WorkerRole ja valitse se, jolloin saat konsolin näkyville ruudulle. Konsolin yläpalkkia painaessa pomppaa konsoli isoksi näytölle. Konsolissa pitäisi näkyä aluksi sekalaista tekstiä, mutta hetken päästä siihen pitäisi alkaa tulostua säännönmukaisesti teksti "Working".

![](http://thorium.github.io/FSharpAzure/AzureWorkerRole/5-ComputeEmulator.png)

Jos näin on, onnittelut, ympäristösi on kunnossa...

3/27/2014 11:15:12 PM 