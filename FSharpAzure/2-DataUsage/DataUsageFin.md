
# Informaatiorikas ohjelmointikieli #

Nykyään F#-kielessä kääntäjä osaa kääntää eri tietolähteitä osaksi ohjelmointikielen tyyppijärjestelmää niin, että esim. koodaajalla toimii intellisense (kirjoitusaikainen syntaksitarkistus). 

Näitä "TypeProvider" tietolähteitä on olemassa mm. seuraavia:

- **JSON**-datalähteille (ja **XML**-tietolähteille)
	- Perus-web-serviceiden schemat. Vähän sama kuin, että generoisi Service-referenssejä, mutta kääntäjä tekee sen puolestasi.
- **CSV**-tiedostoille
	- On-line-dataa tarjotaan paljon myös CSV-muodossa. 
- **FreeBase**-nettitietokannalle
	- Freebase on tietokanta, johon on koostettu tietoa useista eri tietolähteistä. Täältä ei saa helposti tietorakenteita, eikä reaaliaikaista dataa, mutta kylläkin sekalaisia data-listoja itsessään.
- **WorldBankille**
	- Maailmanpankki on YK:n alla toimiva järjestö, josta saa tietoa eri maista. 
- **SQL**-palvelimelle
	- Vähän sama kuin generoisi EF-tietomallin, mutta jälleen, kääntäjä hoitaa kaiken, et voi mennä itse sörkkimään väliin, mikä on hyvä juttu.
- **Azure Marketplace**
	- Jos esim. haluat ostaa domain-mallisi muualta ja itse keskittyä vain liiketoimintalogiikkaan.
	- [http://datamarket.azure.com/browse/data?price=free](http://datamarket.azure.com/browse/data?price=free)
- Ja näitä voi tehdä itse lisää...

Testataan muutamia, käyttäen [FSharp.Data](http://fsharp.github.io/FSharp.Data/)-kirjastoa. Voit käyttää aiemmin tehtyä WorkerRole-projektia. (Tarvitset nettiyhteyden ja näitä käytettäessä Visual Studio saattaa kysyä lupaa netin käyttämiseen, vastaa "Enable".)

## FreeBase TypeProvider ##

Voit käyttää interactivea testaamaan alla olevaa koodia:

	[lang=fsharp]
    #if INTERACTIVE
    #r "../packages/FSharp.Data.2.0.4/lib/net40/FSharp.Data.dll"
    #endif

    open FSharp.Data
    let data = FreebaseData.GetDataContext()
    let Helsinki10 = 
        data.Commons.Business.``Stock exchanges``.Individuals.OMXH.``Companies traded``
        |> Seq.take 10
        |> Seq.map (fun company -> company.``Ticker symbol``)

    let iterated = Helsinki10 |> Seq.iter System.Console.WriteLine

Testaa myös muita data. ja data.Commons -alta löytyviä settejä. Huomaa Visual Studion Intellisense samalla kun kehität:

![](1-Freebase.png)

## CSV TypeProvider ##


Edellisestä saatiin siis lista Helsingin pörssin osakkeista.

CSV-type-provider toimii TypeProviderille tyypillisellä tavalla:


1. Tarvitsemme kovakoodatun Scheman, tiedon muodon. Tämä on yleensä jonkinlainen otos vanhaa dataa, johon viitataan kehityskoodissa.
	- Lisää projektiin uusi tiedosto, TextFile1.csv
	- Tallenna sen sisällöksi tämä:

		    Date,Open,High,Low,Close,Volume,Adj Close
		    2012-01-27,29.45,29.53,29.17,29.23,44187700,29.23
		    2012-01-26,29.61,29.70,29.40,29.50,49102800,29.50
		    2012-01-25,29.07,29.65,29.07,29.56,59231700,29.56
		    2012-01-24,29.47,29.57,29.18,29.34,51703300,29.34
    

2. Tarvitsemme reaaliaikaisen datanlähteen
	- Käytämme [Yahoo Finance](http://ichart.finance.yahoo.com/)-palvelua. Vastaavia nettipalveluita on muitakin, mutta tätä muutkin käyttävät.

Jatketaan edellisen logiikan perään: nyt voit hakea dataa oheisella koodilla:

	[lang=fsharp]
    let symbol = "NOK" // symbols |> Seq.head
    type Stocks = CsvProvider<"TextFile1.csv">
    let quotes = Stocks.Load("http://ichart.finance.yahoo.com/table.csv?s=" + symbol)
    let ``Last day in USD`` =  quotes.Rows |> Seq.head


Valitettavasti Yahoon USD-EUR-kurssi ei ollut ajan tasalla, joten se on haettava jostain muualta.

Jos kirjoitit tiedostonimen TextFile1.csv oikein, niin tässäkin intellisense osaa tyypittää sarakkeet oikein ja jopa tietomuodot (System.DateTime, decimal, jne):

![](2-Csv.png)

## Harjoitustehtävät ##

#### Harjoitustehtävä 1 ####

Tee ohjelma, joka kyselee "kumpi oli presidentti":
> Kumpi oli presidentti, Richard Burton vai Gerald Ford?

Ohjelman toimintaperiaate:

1. Hae Freebase:sta (Commons.Government alta) lista USA:n presidenteistä. Vain nimet.
	- Sijoita tilapäismuuttujaan niin, ettei kyselyitä tule liian usein.
2. Hae Freebase:sta (Commons.Film alta) lista näyttelijöistä. Vain nimet. Ota vain 50 ensimmäistä.
	- Sijoita tilapäismuuttujaan niin, ettei kyselyitä tule liian usein.
3. Kysely-toiminto:
	- Arvo kummastakin listasta yksi nimi ja esitä ne satunnaisjärjestyksessä kumpi-kysymyksen muodossa.
4. Tarkistus-toiminto:
	- Ota parametrina nimi ja tarkista löytyykö annettu nimi presidentti-listalta.


#### Harjoitustehtävä 2 ####
Maailma on täynnä avoimia API-rajapintoja:
 
- Suomalaisia:
	- [Reittiopas](http://developer.reittiopas.fi/pages/en/http-get-interface-version-2.php), 
	- [VR](http://www.vr.fi/fi/index/palvelut/avoin_data/Junatkartalla-rajapinta.html),
	- [Ontologia API](http://onki.fi/api/v2/http/#reposearch) 
	- [Ninchat API](https://github.com/ninchat/ninchat-api/blob/v1/api.md)
	- Avointen palveluiden [listaus](http://www.suomi.fi/suomifi/tyohuone/yhteiset_palvelut/avoin_data/).
- Kansainvälisiä:
	- Luultavasti kaikki isot yritykset kuten YouTube, Amazon, jne.
	- [Finanssi-feed-lista](http://en.wikipedia.org/wiki/List_of_financial_data_feeds)
	- BTC: [Quandl](http://www.quandl.com/api/v1/datasets/BITCOIN/MTGOXUSD.csv?&trim_start=2010-07-17&trim_end=2013-07-08&sort_order=desc), [BTC-e](https://btc-e.com/api/2/btc_usd/trades), [LocalBitcoins](https://localbitcoins.com/api-docs/)

Rekisteröidy käyttäjäksi johonkin, ja tutki API-rajapintaa [JSON-TypeProviderin](http://fsharp.github.io/FSharp.Data/library/JsonProvider.html) avulla.

Jos tarvitset usein SQL-palvelinta, voit myös koittaa käyttää SqlEntityConnection-TypeProvideria.


[Takaisin valikkoon](../Readme.html)