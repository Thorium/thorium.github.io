# F# lyhyt oppimäärä #

F# on selkeä ohjelmointikieli, joka ohjaa lähtökohtaisesti käyttäjäänsä funktionaalisiin ratkaisuihin. Tätä kautta ohjelman tila mallintuu oikein, ja ohjelmat säikeistyvät hyvin ja niillä on totuttua parempi ylläpidettävyys.

> "Functional programming combines the flexibility and power of abstract mathematics with the intuitive clarity of abstract mathematics." - XKCD

Vaikka lyhyt on kaunista, ja F# on tarvittaessa hyvinkin abstrakti kieli, niin tärkeintä ei ole saada aikaan lyhyin syntaksi, vaan tilanteeseen sopivin/selkein. On kielen voimakkuutta, että asiat voidaan tehdä monella tavalla, mutta hyvyyttä, että kieli ohjaa intuitiivisesti parhaaseen ratkaisuun.

Ehkä pieni harppaus kokeellisesta yritys-erehdys-ohjelmistoylläpidosta vähän matemaattisempaan suuntaan ei olisi pahasta?

Koodia oppii lukemaan nopeasti, mutta kivuton kirjoittaminen vaatii aikansa: uuden opettelu vaatii malttia. F#-kääntäjä on tiukempi kuin C#, sen kanssa saa alussa vähän hakata päätä seinään, mutta kun koodi menee kääntäjästä läpi, niin todennäköisemmin ohjelma ei kaadu ajonaikaisesti. Toisaalta se tuo onnistumisentunteita, ja saa C#-veteraaninkin huomaamaan, että koodaaminen voi olla kivaa.

## F-Sharp-projektin luonti Visual Studiossa ##

Voit käyttää Azure-Worker-rolea (josta erilliset ohjeet), tai tehdä kokonaan uuden projektin. Oletuksena projektista kääntyy dll, samoin kuin C#-projektista, ja dll:t ovat yhteensopivia.

Jos haluat tehdä uuden projektin Visual Studiossa, niin se menee näin:
File -> New -> Project... -> Other Languages -> Visual F# ->F# Library

## C# ja F# yhteispeli (ja miksei myös VB.NET) ##

C# on parhaimmillaan kun tarvitaan valmiit wizzardit riittävät, esim. XAML-kehityksessä. F#:sta voi referoida ja käyttää suoraa C#-dll:iä. Samassa Visual Studio "solutionissa" voi olla sekä F# että C# projekteja, mutta muutosten näkyminen toisesta toiseen vaatii referoitavan projektin kääntämistä.

Lisää kokeeksi C#-projekti (kirjasto, console-app, testiprojekti, tms) samaan "solutioniin" F#-projektin rinnalle, ja lisää viittaus:
Solution Explorerista C#-projektin References-kansion päällä hiiren oikeaa nappia ja:
Add reference... -> Solution -> rasti F#-projektin ruutuun ja ok.

Tässä malliluokka F#-sisällöksi (näitä käydään tarkemmin läpi alempana):

    [lang=fsharp]
    namespace MyLibrary
    
    type MyClass1() = 
        member this.X = "F#"
    
    module MyModule =
    
        let StaticItems = 42
    
        let MyList = ["a";"b";"c"] |> List.toSeq
    
        let MyAddFive x = x+5;
    
        let MyGenericFunction x = "Hello " + x.ToString();
    
        type UserData1() = 
            member val Email = "" with get, set

...ja C#-projektin luokasta niitä voi (projektin käännön jälkeen) käyttää ihan suoraa:


    [lang=csharp]
	var class1 = new MyLibrary.MyClass1();
	var fromFs = class1.X;
	
	int Item = MyLibrary.MyModule.StaticItems;
	
	IEnumerable<string> response = MyLibrary.MyModule.MyList;
	
	int seven = MyLibrary.MyModule.MyAddFive(2);
	
	var res = MyLibrary.MyModule.MyGenericFunction<int>(2);
	
	var class2 = new MyLibrary.MyModule.UserData1();
	class2.Email = "no@spam.com";
        
Näin vanhalla C#-koodaajalla on turvaverkko, johon voi luottaa ennenkuin F# on täysin hallinnassa.
Samaan tapaan toimisi myös referointi VB.NET-komponenttiin.

Huomattavat rajat:

- F#:ssa yleensä kirjoitetaan vakioiden/muuttujien/funktioiden nimet pienellä. 
    - Tosin se on vain käytäntö, jotta erottaa nämä tyypeistä/luokista.
- F#-perus-lista on oma linkitetty listansa. 
    - Seq on tyypitetty IEnumerable.
- F#-perus-funktio on myös omansa
    - F#:sta voi kutsua C#-metodia joka ottaa C# Func-parametrin, F#-funktio kelpaa tähän suuntaan.
    - Mutta jos F#:sta palauttaa oletus-funktion metodista ulos, niin C#-kehittäjä vähän hämmästyy.
    - F#:sta voi erikseen luoda ja palauttaa ulos C#:pin Func-tietotyyppin, mutta sen käyttö F#:pin sisällä ei ole luonnollista.
        - C#:ssa Func ja Action ovat eri luokat ja void ei ole tietotyyppi. 
        - F# void on nimeltään unit ja se on tietotyyppi. 

Näillä rajoilla on merkitystä lähinnä, jos ennalta tiedetään, että kehitetään F#:sta komponenttia C#-kehityksen käyttöön.

Sitten itse kieleen:

## Hello world ##

Käyttäen .NET-luokkia, "Hello World"-ohjelman voi tehdä esim. näin:

    [lang=fsharp]
    open System
    let x = "Hello World"
    Console.WriteLine x

Lähtökohtaisesti F# ohjaa siihen, että et voi käyttää "muuttujia" (/vakioita) uudestaan, eli kun x on kerran määritelty, niin käytä mieluummin seuraavalla kerralla muuttujaa x2 kuin x. (Tosin ehkä paremmalla nimeämiskäytännöllä.) Oikeastihan tarvitset muuttujia vain ylläpitämään luuppien tilaa. Lähtökohtaisesti F# ohjaa käyttämään rekursiota luuppien sijaan, mutta tästä lisää myöhemmin.

Vertailuoperaatio on vain yksi yhtäkuinmerkki.
Kommentit ovat // tai (* ja *). Private ja public näkyvyydet ovat käytettävissä.

Voit myös antaa kohteille vielä selkeämpiä välilyönnillisiä nimiä (tosin huona puolena suomalainen näppäimistö-layout:in hankaluus):

    [lang=fsharp]
    let ``Hello World Container`` = "Hello World"

## Interactive ##

Olettaen, että Resharper ei ole ihan sotkenut pikanäppäimiäsi:

Painamalla Ctrl+Alt+F saat esille interactive-ikkunan ("REPL-loop"), jossa voit suorittaa koodia lennosta. Voit kirjoittaa koodia ikkunaan suoraa, ja suorittaa sen kirjoittamalla ennen enterin painallusta kaksi puolipistettä, esim: 

    [lang=fsharp]
    let z = 4;;

Tai sitten voit suoraa projektitiedostosta maalata kasan koodia, ja lähettää sen interactive-ikkunaan (joko hiiren oikealla napilla ja "Execute in interactive", tai näppäinyhdistelmällä Alt+Enter.

Yleensä kannattaa ensin antaa haluamansa alkuarvot parametreille ja sitten lähettää haluamansa koodinpala interactiveen.

F#:lle on myös hyvät yksikkötesti- ja BDD-frameworkit, mutta interactiven ansiosta niitä ei tarvita kokeelliseen kehitykseen, vaan lähinnä ylläpidon varmistamiseen.

## Tyypitys ##

Tyyppijärjestelmähän on tarkoitettu siihen, että kääntäjä palvelee koodaajaa, eikä toisinpäin. Oletuksena F# löytää tyypit itsekin, mutta joskus käyttäjä haluaa tyypittää käsin:

    [lang=fsharp]
    let f1 x = x+4.5m 
    let f2 (x:decimal) = x+4.5m //argument type
    let f3 x :decimal = x+4.5m  //function type

Geneeriset tyypit (klassinen T, T1, T2, jne) merkitään tyyppijärjestelmässä heittomerkillä: 'a, 'b, 'c, jne

## Putkitus ##

F#:ssa on operaatio "|>", joka siirtää metodin viimeisen parametrin ensimmäiseksi:

    [lang=fsharp] 
    open System
    "Hello World" |> Console.WriteLine

Tätä voit ajatella vähän kuin C#-extension-metodina: jos metodit ovat verbejä, on usein luonnollista aloittaa lause substanttiivilla, kielestä tulee luonnollisempaa: 

"koira haukkuu ihmiselle" vs. "haukkuu(koira, ihmiselle)"

## Funktiot ##

F#:ssa ei tarvita sulkuja funktioparametreissa ja pilkku on varattu Tuple-luokkien käyttöön, joten parametrillisen funktion voit tehdä näin:

    [lang=fsharp]
    let f x y = x+y

Funktion tyyppisyntaksi on sama kuin C#-kielen Func<...>-luokan parametrit, esim: Func<int, int, string> on funktio joka ottaa kaksi numeroa ja palauttaa merkkijonon. F#:ssa parametrien välissä on nuoli, eli vastaavan funktion tyyppisyntaksi on

    int -> int -> string

Funktion tyyppisyntaksista voi hyvin pitkälle päätellä mitä funktio tekee.

Parametriton funktio määriteltäisiin suluilla, tätä tarvitset, jos et halua suorittaa evaluointia heti. return-komentoa ei yleensä tarvi, koska funktio palauttaa viimeisen käskyn arvon ulos joka tapauksessa. Funktiot voivat olla sisäkkäin.

    [lang=fsharp]
    let sayHello () = 
        let hello = 
            let ello = "ello"
            "h"+ello
        let world = 
            "world"
        hello + " " + world

Lambda-funktion voi tehdään fun-sanalla ja nuolella.


## Listat ##

F#:ssa on LINQ:a monipuolisemmat  listojenkäsittelykirjastot, mutta voit halutessasi käyttää myös LINQ-kirjastoa, jos sen jo osaat.

F#:ssa näkyy usein käytettävän kolmea eri tyyppiä listoja, joilla on kaikilla suurinpiirtein samat käsittelykirjastot. Karrikoidusti tyypit menevät näin:

- List
    - Ylläpidettävin koodi.
    - Immutable: alkiot eivät muutu
    - Kaksi kaksoispistettä voidaan käyttää tunnistamaan ensimmäinen ja loput alkiot
    - At-merkkiä "@" voi käyttää yhdistämään kaksi listaa.
- Array
    - Tehokkain suoritus. Vastaa C#-arrayta.
- Seq
    - Yhteensopivin. Tämä vastaa C# IEnumerablea.

Listoja voi muuttaa muodosta toiseen: List.toArray, List.toSeq, Seq.toList, Array.toSeq, jne. Pari  koodiesimerkkiä:

    [lang=fsharp]
    let emptyList = []

    let listOfLists = [[1;2;3];[4;5;6]]

    let myList = 
        [1..10] |> List.filter(fun i -> i%2=0)
    let mySeq =
        [|1..10|] |> Seq.filter(fun i -> i%2=0)

    let merged = [1;2;3] @ [4;5;6]

    let head::tail = ["h";"t1";"t2";"t3"]

    let ``A and H to Z`` = 'A'::['H'..'Z']

    open System.Linq
    let threeFirst = 
        mySeq.Take(3)
        |> Seq.toArray
        |> Array.map(fun i -> i*3)


Funktioita voi myös yhdistellä (function composition) ">>" -merkillä, tosin tätä näkee harvemmin.

    [lang=fsharp]
    let composed = 
        List.filter(fun i -> i%2=0) >> List.map(fun i -> i+5)
    [6; 5; 3; 2; 4] |> composed

Listan tyypin voi halutessaan merkitä joko C#-tyylisesti tai OCaml-tyylisesti:

    [lang=fsharp]
    let emptyList2:list<bigint> = []
    let emptyList3:bigint list = []

## Tuple ##

Listojen lisäksi toinen yleinen rakenne on Tuple: lista on n-kappaletta yhtä samaa tyyppiä, tuple on yksi kappale n:ää eri tyyppiä. C#:ssa on myös Tuple, ja tämä luokka on sama.

Voit toki tehdä Tuplen C#-tapaan System.Tuple.Create(1, "a", 1.0), mutta tähän on myös helpompi tapa, pelkkä pilkku. Tuplen voi myös takaisin purkaa osiinsa, joko fst- ja snd-käskyillä, tai sitten määrittelemällä "temp-muuttujat" yhtäkuin-merkin vasemmalla puolella:

    [lang=fsharp]
    let tuple = 1, "a", 1.0
    let a, b, c = tuple

Tyyppisyntaksissa tuple merkitään tähdellä, esim: int*string.

## Luokat ##

F# on myös erinomainen oliokieli.

Olio, jolla on yksi property, email, jossa on getteri ja setteri, ja oletusarvo sille on tyhjä merkkijono, tehtäisiin näin:

    [lang=fsharp]
    type UserData1() = 
       member val Email = "" with get, set

	let myInstance1 = UserData1()

Kun luot uuden instanssin oliosta, niin new-sana on vapaaehtoinen.

Olio, jolla on yksi konstruktorissa asetettava readonly-property, email, ja lisäksi tyhjä konstruktori, joka asettaa tyhjän arvon tehtäisiin näin:

    [lang=fsharp]
    type UserData2(email) =
       new() = UserData2("")
       member x.Email = email

	

Perintä-syntaksi ei ole yhtä näppärä kuin C#:ssa, koska vahvaan tyyppitarkistukseen rajapinnat pitää aina periyttää eksplisiittisesti:

    [lang=fsharp]
    type Base() =
        let DoSomething() = ()
        abstract member Method : unit -> unit
        default x.Method() = DoSomething()

    type ``My inherited class with dispose``() =
        inherit Base()
        override x.Method() =
            base.Method()

        interface IDisposable with
            override x.Dispose() = 
                //Dispose resources...
                GC.SuppressFinalize(x)

## Discriminated unioin ##

Voit tehdä "joko-tai"-tyyppejä:

    [lang=fsharp]
    type myEnum =
    | A
    | B

    type CalendarEvent =
    | ExactTime of DateTime
    | MonthAndYear of string*int //month*year

    type Tree =
    | Leaf of int
    | Node of Tree*Tree

Käyttötarkoituksena esimerkiksi ohjelman tilanhallinta, jo kääntäjätasolla, ilman sotkuista ajonaikaista if-logiikkaa. Nämä ovat erittäin käteviä yhdessä pattern matchingin kanssa. 

## Astetta erikoisemmat tyypit ##

Tyyppi voi olla myös vain alias, tai "joko-tai" voi esiintyä myös yksinään:

    [lang=fsharp]
    type aliasItem<'a,'b> = System.Collections.Generic.KeyValuePair<'a,'b>

    type OrderId = | Id of System.Guid

Tyyppi voi olla tietue, record:

    [lang=fsharp]
    type Address = { 
        Street: string; 
        Zip: int; 
        City: string; 
    }
    let myAddrr = { Street = "Juvank."; Zip = 33710; City = "Tre"; }

...tai sillä voidaan Measure-attribuutin kanssa määritellä vahva vain-käännösaikainen tyypitys. Vähän kuin oma luokka kapseloimaan vain yksi laskennallinen arvo (mutta näyttämään se aina ulos), jotta eri asiat tai mittayksiköt eivät varmasti mene sekaisin:
 
    [lang=fsharp]
    [<Measure>]
    type EUR

    [<Measure>]
    type LTC

    let rate =
        let ratePlain = 9.45m //fetch from internet...
        ratePlain * 1m<EUR/LTC>

    let myMoneyInEur (x:decimal<LTC>) = x*rate 

    let converted = myMoneyInEur 7m<LTC>

Helpoin tapa muuttaa perus-.NET-tyyppi mitaksi on kertoa se yhdellä mitalla. Esteettinen haittapuoli on, että tämä tuo "pienempi kuin" ja "suurempi kuin" merkkejä koodiin.

## Pattern matching ##

Pattern-matching on tavallaan selkeä switch/case tai if-elseif-kuvio, jossa ehto ei voi vaihtua kesken kaiken, mutta eri case:illa ei ole constant-vaatimusta. Eri patternteita on useita. 

Lisäksi funktio voi suoraa olla pattern-funktio tai voidaan käyttää ns. active-patterneita selkeyttämään koodia. Tässä sama toiminnallisuus usealla eri kooditavalla:

    [lang=fsharp]
    let MyFunction1 a =
       if a = "A" then 1
       elif a = "B" then 2
       else 3

    let MyFunction2 a = 
       match a with
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction3 = fun a -> 
       match a with
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction4 = function
       | "A" -> 1
       | "B" -> 2
       | _ -> 3

    let MyFunction5 a = 
       match a with
       | x when x = "A" -> 1
       | x when x = "B" -> 2
       | x -> 3


    let MyFunction6 a = 

        let (|FirstVowel|FirstConsonant|Other|) p =
            match p with
            | "A" -> FirstVowel
            | "B" -> FirstConsonant
            | _ -> Other

        match a with
        | FirstVowel -> 1
        | FirstConsonant -> 2
        | Other -> 3
 

Usein, jos mahdollista, on parempi jättää oletus-tapaus kokonaan pois, koska silloin kääntäjä saa kiinni virheet, jos tuntemattomia tapauksia luodaan jälkikäteen koodiin  lisää.

## Nullittomuus ja option-type ##

C#:ssa NULL on sekä "special case" (Fowler: PoEAA) että un-assigned-muuttuja. Kaksi rooila yhdellä asialla tekee sen käytöstä sotkuista.

Lähtökohtaisesti F#:ssa NULL ei ole käytössä. Sen sijaan on käytössä option-type, asialle jota ei ole tiedossa:

    [lang=fsharp]
    let echo x = "You have to work " + x.ToString() + " hours."

    let ``working hours`` =
        let day = (int System.DateTime.Now.DayOfWeek)
        match day with
        | 0 | 6 -> None
        | _ -> Some(7.5)
    
    let echoHours1 = 
        match ``working hours`` with
        | None -> ""
        | Some h -> echo h

    let echoHours2 = 
        ``working hours`` 
        |> Option.map(fun h -> echo h)

    let echoHours3 = 
        ``working hours`` 
        |> Option.map(echo)


## Rekursiivisuus ##

Rekursiiviseen funktioon kirjoitetaan rec. Usein itse rekursio kapseloidaan toisen funktion sisään, jotta voidaan peittää "turhat" alkuarvot. Usein myös käytetään muotoa, jossa viimeisenä parametrina menee "akkumulaattori", johon kerätään tulos. Tämä siksi, että kääntäjä optimoi häntärekursiolla stäkin pois, ts. ei tule StackOverflowException:ia, vaikka läpikäytävä rekursio olisi iso.

    [lang=fsharp]
    let sumItems myList =
        let rec calculate li acc =
            match li with
            |[] -> acc
            |h::t -> calculate t (h+acc)
        calculate myList 0

Tämä tyypillinen funktio löytyy tietysti myös suoraa List-kirjastosta, mutta jos tarvitaan enemmän custom-toiminnallisuutta, niin silloin oma rekursio on joskus paikallaan.


## Computation Expressions ##

Nämä ovat eräänlaisia kapseleita kapseloimaan toiminnallisuuden sivuvaikutuksia. Valmiita ovat mm. seq (listoille) ja async (asynkroniselle koodille), mutta näitä voi tehdä itse lisää.

Asynkronisuuden sivuvaikutus on odottaminen. Async.RunSynchronously nyt ei itsessään ole kovin hyödyllinen, mutta Async:illa on paljon muitakin vaihtoehtoja.

Asian ydin on se, että kun katsotaan koodia kohdassa sum = x + y, niin se näyttää hyvin samalta kuin perus F#-koodi muutenkin, vaikka ollaan sivuvaikutuksen piirissä. Ja itse sivuvaikutus, async, ilmenee koodista eksplisiittisesti. Huutomerkki-syntaksi tarkoittaa kutsua "computation expression"-rajapintaan.

    [lang=fsharp]
    let asyncOperation = Async.Sleep 3000
    
    let waitAndReturn = 
        async {
            do! asyncOperation
            return 3.0
        }
    
    let waitAndReturn2 = 
        async {
            let! x = waitAndReturn
            let y = 7.0
            let sum = x + y
            return sum
        }
    
    let result = Async.RunSynchronously waitAndReturn2

Jos verrataan tätä C#:piin, niin seq { ... } vastaava on LINQ. Mutta LINQ on tavallaan oma kielensä, joka koodaajan on opeteltava erikseen. Ja koodaaja ei tiedä tekeekö LINQ:a IEnumerablea, IQueryablea, IObservablea, vai mitä vasten... Vähän naiivi ja C#-henkinen, mutta toimiva esimerkki seq-expressionista:

    [lang=fsharp]
    open System.Linq
    let slice mySeq = 
        let chunkSize = 5
        let rec seqSlice (s:int seq) =
            seq{
                yield s.Take chunkSize
                yield! seqSlice (s.Skip chunkSize)
            }
        seqSlice mySeq
    
"Computation Expressions" on kauniimpi nimi asialle, josta käytetään myös ilmaisua "Monad".

## Harjoitustehtäviä ##

- Luo rekursiivinen ohjelma joka listaa 1000 ensimmäistä fibonacci-lukua (1, 1, 2, 3, 5, 8, 13, ...).
- Luo C#-yksikkötesti tai konsoliaplikaatio kutsumaan edellistä.


## Linkit ##

Jos tämä oli liian hapokasta, niin kannattaa katsoa läpi [tämä materiaali](https://bitbucket.org/ilmirajat/fsharptraining.fi/src/05a105289a51212202c76794a704721503c55e41/FSharpTraining/).
Jos haluat opetella listakirjastojen toiminnallisuuksia, niin koita tehdä [Project Euler](https://projecteuler.net/problems) -tehtäviä. Jos haluat itsenäisen pienen koodiesimerkin jostain aiheesta, niin [fssnip.net](http://www.fssnip.net/pages/AllTags) on hyvä saitti lähteä etsimään.


[Takaisin valikkoon](../Readme.html)