# Domain-mallinnus, DSL-kieli #

Domain-mallilla tarkoitetaan (kehittäjille suunnattua) tietomallia (=rakennetta), ohjaamaan ongelmanratkaisua tietyille raiteille. 

Domain Specific Language (DSL) viittaa tässä vain Embedded / Internal tyyliseen kieleen, jossa itsenäinen funktionaalinen (kombinaattori-)kirjasto upotetaan osaksi F#-kieltä. Emme käy läpi Fowler:in OO-pohjaista DSL-kirjaa.

## Mallinnuksen vaiheet ##
Mallinnus koostuu kolmesta vaiheesta

1. **Valitaan/mallinnetaan alkiot (primitives)**
	- Järjestelmän perus-yksiköiden tilat. 
	- Vain keskeisin/oleellisin tieto
2. **Mallinnetaan kompositio (yhdistäminen)**
	- Miten primitiivejä yhdistellään niin, että ne sopivat saumattomasti yhteen
	- Kombinaattorit 
3. **Syntaksi**
	- Lisäkoristelut, jotta palikoita on kiva käyttää.

## 1. & 2. Mallinnetaan alkiot ja niiden kompositio ##

Toiminnallisuudet voidaan lähteä mallintamaan joko tietona (contract as data) tai laskentaoperaatioina (contract as computation). Riippumatta tästä, käytössä on samat työkalut.

### Mallinnuksen työkalut ###

Mallinnus kannattaa tehdä käyttäen kolmea työkalua:

- **Tuplet**
	- "Product type", voidaan ajatella AND-operaationa
- **Discriminated union**
	- "Sum type", voidaan ajatella OR-operaationa
- **Funktiot**
	- Siirtymä tilojen välillä
	- Toiminnon mallinnus

Teoriassa boolean-operaatiot riittävät tietotekniikassa mallintamaan kaiken. Selkein domain tuleekin käyttäen näitä (ts. lähtökohtaisesti vältä oliorakenteita), mutta käytännössä halutaan ehkä käyttää apuna vielä seuraavia:

- **Record**, jos yhteen-niputettavaa tietoa on paljon (esim. "ERP-olio")
- **Lista**, jos yksittäisiä kohteita ei haluta listata
- Discriminated union voidaan *jälkikäteen* vaihtaa rajapinnaksi, jos kaikki sen jäsenet eivät ole tiedossa, ts. halutaan plugin-arkkitehtuuri.

### Domain-mallin mallintaminen (contract as data) ###

Otetaan esimerkkinä yksinkertainen osake/pörssikauppa.

Kuinka mallinnetaan:

- Haluan ostaa Microsoftin osakkeita 100kpl
- Haluan ostaa Nokian osakkeita 500kpl
- Haluan myydä Googlen osakkeita 300kpl

Alkiot (primitiivit) olisivat yksittäisiä kauppoja, ja kompositio on sitä, että primitiiveistä yhdistellään eräänlainen abstrakti syntaksipuu.

Usein kannattaa lähteä mallintamaan järjestelmän toimintoja (commandeja, verbejä), eikä niinkään "perus-olioita" (substantiiveja). Malli voisi näyttää esimerkiksi tältä:

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

Tai tältä:

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

Näiden kahden merkittävin ero on siinä, että sitten kun näitä käytetään, niin kuinka helposti päästään käsiksi parametritietoihin:

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

Kannattaa valita vaihtoehto 2, jos käytettävät parametrit kuvaavat samaa tietosisältöä, kuten tässä. Silloin operaatio on helpompi muokata muodosta toiseen, esim. vaihtaa Buy -> Sell. Tee mieluummin useita tyyppejä kuin mega-tyyppejä. 

Tämä on helpoin ja selkein tapa mallintaa, näin saat selkeän tietomallin. Haittapuolena on, että operaatiot ovat kiinteitä (/"kovakoodattu"), eli et voi kirjastomaisesti yhdistää omia rakenteita tekemään mitä tahansa, vaan toiminnallisuuden on kunnioitettava tietomallin toiminnallisuuksia. Usein tämä riittää, kun tavoitteena on rakentaa yksittäinen järjestelmää, jolla on selkeät rajat.

Siinä missä C# usein johtaa olio-mappaus-koodiin tyypistä toiseen, auttavat [F# Object Expressionit](http://msdn.microsoft.com/en-us/library/dd233237.aspx), joilla voi luoda instansseja (structeista, olioista, interfaceista) ilman boilerplate-koodia:

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


### Tietomallin mallintaminen laskentaoperaatioina (contract as computation) ###

Tämä on perus-ohjelmaan usein turhan monimutkainen konsepti (ellei call-cc ja Haskell-kirjastot ole entuudestaan tuttuja), mutta hyvä vaihtoehto, jos on pakko yrittää rakentaa jonkinlainen oma workflow/prosessi/sääntö -moottori tai framework. Erona edelliseen on se, että käyttäjä voi referoida tämän kirjastona ja itse tehdä komposito-operaatioita lisää. Tällöin kyseessä alkaa olla jo kombinaattorikirjasto tai DSL-kieli.

Tietomallia voidaan mallintaa toimintona/funktiona `('a -> M<'b>)`, jossa a on "ohjelman tila", ts. mitä vastaan kombinaatiosääntöjä tehdään, ja M kuvaa jonkinlaista kapselia/monadia (vaikkapa lista), ja b taas on tuloksen tyyppi. Kapseli on vapaaehtoinen ja myöskään a ja b ei tarvitse olla geneerisiä, jos tarkemmat tyypit on business-mielessä mahdollista määrittää. Aluksi annetaan funktiolle nimi, jotta ei hukuta nuoliin:

    [lang=fsharp]
    type OptionTrade<'a,'b> =
    | Operation of ('a -> List<'b>)

Nyt voidaan luoda funktio, joka yhdistää kaksi toiminnallisuutta, esim. listalla:
    
    [lang=fsharp]
    let combine (Operation f1) (Operation f2) = 
        Operation(fun a -> [f1(a); f2(a)])
    
    // OptionTrade -> OptionTrade -> OptionTrade

ja määritellään vielä toinen funktio, itse suoritus:

    [lang=fsharp]
    let eval (Operation f) a = f(a)

Nyt käyttäjä voi tehdä mitä tahansa Operation-tyylisiä funktioita, jotka toteuttavat syntaksin `(a -> List<b>)` ja näitä voidaan yhdistellä käyttäjän mielen mukaan. 
Esim:

    [lang=fsharp]
    let buy name amount = Operation(fun a -> [(name, amount)])

Lopuksi käyttäjä pyytää suorittamaan ajon.

Koodin logiikkaa ei erikseen tehdä domainin rinnalle, vaan kirjasto jo suorittaa koodin logiikan. Eli abstrakti-syntaksi-puu muodostuukin kooste-funktioista lähdekoodissa:
  
    [lang=fsharp]
    let myCombination = 
        combine
            (buy "MSFT" 100m)
            (buy "NOK" 500m)
    let myDone = eval myCombination "now!"


Tässä tehtiin koodista turhan geneeristä ja myynti-(sell)-operaatio jätetään harjoitustehtäväksi. Käytännön tarkempi lähdekoodiesimerkki (jossa myös myynti on toteutettu, ja kasa muita operaatioita) on saatavilla [netistä](http://archive.msdn.microsoft.com/realworldfp/Release/ProjectReleases.aspx?ReleaseId=3674), kirjan ["Real-World Functional Programming"](http://www.manning.com/petricek/) koodiesimerkeistä, kappale 15.

#### Standardi-operaatiot ####

Yhdisteltäessä eri toiminnallisuuksia (jopa eri kirjastojen välillä), on usein kätevää käyttää tuttuja standardi-operaatiota kuten esim:

- map `('T -> 'R) -> M<'T> -> M<'R>`
- bind `('T -> M<'R>) -> M<'T> -> M<'R>`
- return `'T -> M<'T>`

Esim map menisi näin (yhdellä ja kahdella parametrilla):

    [lang=fsharp]
    //Map with one parameters, just basic composition:
    let map f (Operation f1) = Operation(fun a -> f(f1(a)))
    //Map with f having two parameters:
    let map2 f (Operation f1) (Operation f2) = Operation(fun a -> f (f1 a) (f2 a))

    

Vältä sivuvaikutuksia. Operaation tietotyypistä voi jo päätellä mitä itse operaatio tekee.

## 3. Syntaksi ##

Jotta toiminnallisuuksiasi olisi parempi käyttää, niin F#:ssa on lukuisia hienouksia: omat custom-operaattorit, Builder-syntaksi, Quotations (kuin C# -expression-puu), Query-syntaksi, ...
Tässä esiteltynä tarkemmin kaksi:

### Custom-operaattorit ###

F#:ssa voit tehdä omat [operaattorisi](http://msdn.microsoft.com/en-us/library/dd233204.aspx), kuten normaali funktio, mutta kirjoittamalla operaattorin sulkuihin. Käytettäessä operaattorin ensimmäinen parametri tulee ennen operaattoria. Esimerkkinä seuraava "hauska jekku":

    [lang=fsharp]
    let (+) x y = x-y
    let minusOne = 5+6

mutta tälle on myös erittäin mukavia käyttötarkoituksia omissa DSL-kielissä, tästä esimerkkinä edellisen kohda combine-funktio (partial application:in takia ei tarvitse listata parametreja):

    [lang=fsharp]
    let (&) = combine

    let myCombination2 = 
        buy "MSFT" 100m &
        buy "NOK" 500m &
        sell "GOOG" 300m

    let myDone2 = eval myCombination2 "now!"

Ylikuormitettavat operaattorit voi myös määrittää tavallisina member-funktioina tyypille. F#:ssa voit tehdä myös extension-metodeja (ja extension-propertyjä!):

    [lang=fsharp]
    type System.String with
        member x.yell = x + "!"
    // "hello".yell
    
### Builder-syntaksi ###

Omia computational expressioneita (/monadeita) voi rakentaa lennosta: Keksit vain tilan/sivuvaikutuksen, jonka kapseloit. Tämän sisällä ohjelmoidaan `konteksti{ ... }` -syntaksilla (jossa "konteksti" voi olla melkein mikä tahansa valitsemasi sana).

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

- Kontekstin sisällä huutomerkki-käskyt ("syntaktisokeria") ohjaavat builder-"rajapinnan" vastaaviin metodeihin.
- "Rajapinnasta" ei tarvitse täyttää kuin valitsemasi metodit. Homma perustuu continuationiin (tarkemmin: call-cc) ja reify:yn.
- Tarkempi kuvaus rajapinnan metodeista ja sisäisestä toiminnasta löytyy netistä.
    - [Rajapinta](http://msdn.microsoft.com/en-us/library/dd233182.aspx)
    - [Lisäinfoa](http://blogs.msdn.com/b/dsyme/archive/2007/09/22/some-details-on-f-computation-expressions-aka-monadic-or-workflow-syntax.aspx)

Tätä voidaan käyttää kapseloimaan jokin asiaan liittyvä tila, osakekauppaesimerkissä se voisi olla vaikkapa pankkitilin rahamäärä, joka on käytössä kauppoihin.

## Harjoitustehtävät ##

### Harjoitustehtävä 1 ###

Ohessa on piirretty kuvitteellinen lainanhakuprosessi: [pdf](loanappprocess.pdf).

Siitä copy & pastella on hutaistu tekstit oheisiin tietotyyppeihin:

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

- Miten optimoisit näitä tietotyyppejä?
- Kirjoita myös muutama kuvitteellinen metodi käyttämään tätä kuvattua prosessia ja kuljettamaan lainahakemusta prosessin vaiheesta toiseen. 
- Prosessiin kuulumaton data kannattaa kapseloida mieluummin uutena tuplena myöhemmin rinnalle, kuin prosessin sisään.

### Harjoitustehtävä 2 ###

Mieti mikä olisi huono domain-malli?

### Harjoitustehtävä 3 ###

Kappaleessa "Tietomallin mallintaminen laskentaoperaatioina" oli jätetty myynti-operaatio pois. Myynti voisi olla joko vain negaatio, tai sitten lisätään taas rinnalle oma `type OperationKind = Buy | Sell`.

Koska myyntejä on vain yksi, ja combine ottaa kaksi parametria, niin tarvitaan vähän koodimuutoksia, esim. wrapperi käytettyyn listaan, ja ehkä tälle oma custom-operaattori:
		
    [lang=fsharp]
    let ``return`` (Operation f1) = Operation(fun a -> [f1(a)])

Toteuta myyntioperaatio ja koita saada se toimimaan alkuperäisessä esimerkissä.

Tässä esimerkki miten tehdään omia funktioita:

    [lang=fsharp]
    //Example use, with list-parameter, one parameters:
    let doubleTradeAmount = map (fun al -> [fst(al |> List.head),snd(al |> List.head)*2m])
    let goneDouble = doubleTradeAmount (buy "MSFT" 100m)
    eval goneDouble ("not-used-initial-state",0m)

Kuten huomataan, niin tässä kontekstissa turhasta listakapselista on nyt vaivaa (turha List.head). Voit koittaa ottaa listan pois.

Tee jokin oma operaatio, joka käyttää map2-funktiota.


## Linkit / Lähteet ##

Tomas Petricek - Domain Specific Languages in F# - [Video](http://vimeo.com/60546839), [Slides](http://www.slideshare.net/tomaspfb/dsls) 

Simon Peyton-Jones - [Composing contracts: an adventure in financial engineering](http://research.microsoft.com/en-us/um/people/simonpj/papers/financial-contracts/contracts-icfp.htm) 

Lab49 - [The Algebra of Data, and the Calculus of Mutation](http://blog.lab49.com/archives/3011)

Philip Wadler - [Theorems for free!](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf)

[Takaisin valikkoon](../Readme.html)

