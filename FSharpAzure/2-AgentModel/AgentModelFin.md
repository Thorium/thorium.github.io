# Agent-malli ja Actor-malli #

Puhuttaessa Actor-mallista termi "Actor" viittaa "toimijaan" tai "tekijään", ei "näyttelijään". Puhuttaessa Agent-mallista termi "Agent" viittaa myös "toimijaan" tai "edustajaan", mutta ei "salaiseen agenttiin".

Jos haluat testailla nopeuksia, saat päälle ajanoton interactiveen kirjoittamalla:
 
    #time "on";;

## Rinnakkaissuorituksesta ##

Rinnakkaista suorittamista on kolmea tyyppiä: 

- Parallel Execution
	- Ajetaan samaa koodipolkua yhtaikaa monessa säikeessä.
	- Sopii rinnakkaisiin joukko-operaatioihin itsenäisille alkioille
	- Sopii tarvittaessa CPU-resursseja paljon ja kun niitä on myös käytettävissä.
	- .NET-ympäristössä: Task Parallel Library (TPL), .AsParallel()
- Concurrency.  
	- Ajetaan eri koodipolkuja yhtaikaa.
	- Sopii silloin kun odotettavat resurssit ovat jotain muuta kuin CPU: Esim I/O tai se, että "odotetaan muita, odotetaan synkronointia".
	- Perustuu siihen, että luovutetaan oma säie pois, kun sitä ei tarvita: Kaikkien muidenkin on tehtävä niin!
	- .NET-ympäristössä: Async-await
	- Hyödynnetään yleisesti viestipohjaisissa ("Message Passing") arkkitehtuureissa
- Lukitaan säikeet ja tila
	- Ei ajeta rinnakkain
	- Ei synkronointikysymyksiä
	- .NET: lock
	- Skaalautuvuus- ja suorituskykyongelmat

Asiaa havainnollistamaan, tässä kolme pientä ohjelmaa, jotka lähinnä kelaavat listan läpi ja nukkuvat joka alkiolla hetken:

    [lang=fsharp]
    #if INTERACTIVE
    #time "on"
    #endif

    open System
    open System.Threading
    open System.Threading.Tasks

    let Execution() = 
        let iteri() = 
            [|1..1000|] |> Array.map(
                fun i -> 
                    Thread.Sleep 200 //Lock the thread
                    match i % 250 with
                    | 0 -> Console.WriteLine("Item: " + i.ToString())
                    | _ -> ()
            )
        Console.WriteLine("Starting...")
        let complete = iteri()
        Console.WriteLine("Done")

    let ParallelExecution() = //TPL
        let iteri() = 
            [|1..1000|] |> Array.Parallel.map(
                fun i -> 
                    Thread.Sleep 200
                    match i % 250 with
                    | 0 -> Console.WriteLine("Item: " + i.ToString())
                    | _ -> ()
            )
        Console.WriteLine("Starting...")
        let complete = iteri()
        Console.WriteLine("Done")

    let AsyncParallelExecution() = //Async-await
        let iteri() = 
            [|1..1000|] |> Array.map(
                fun i -> 
                    async {
                        do! Async.Sleep 200
                        return 
                            match i % 250 with
                            | 0 -> Console.WriteLine("Item: " + i.ToString())
                            | _ -> ()
                    }
            )
        Console.WriteLine("Starting...")
        let complete = iteri() |> Async.Parallel |> Async.RunSynchronously
        Console.WriteLine("Done")

Ensimmäinen on hidas. Seuraava on huomattavasti nopeampi. Mutta sille käy niin, että koneesta loppuvat threadit kesken. Kolmas on selvästi nopein, koska se vapauttaa säikeet suorittaessaan (ei-cpu-riippuvaista) async-operaatiota.

Tässä vastaavat suoritusajat (Interactivessa, i7-kannettavalla):

	> Execution();;
	Starting...
	Item: 250
	Item: 500
	Item: 750
	Item: 1000
	Done
	Real: 00:03:20.582, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()
	> 
	
	> ParallelExecution();;
	Starting...
	Item: 250
	Item: 500
	Item: 1000
	Item: 750
	Done
	Real: 00:00:22.266, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()
	
	> AsyncParallelExecution();;
	Starting...
	Item: 1000
	Item: 750
	Item: 500
	Item: 250
	Done
	Real: 00:00:00.204, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
	val it : unit = ()

## Viestijonot ##

Erilaisia MQ-jono-tuotteita on useita. Mitäpä jos yksinkertainen viestijonotoiminnallisuus olisi integroitu suoraa ohjelmointikieleen?

Testaa oheinen koodi interactiven avulla:

    [lang=fsharp]
    let myQueue = 
	    new MailboxProcessor<string>(fun myHandler -> 
	        let rec readNextMessage() =
	            async {
	                let! result = myHandler.Receive()
	                System.Console.WriteLine("Handled: " + result)
	                do! readNextMessage()
	            }
	        readNextMessage())

	myQueue.Post("Hello 1 !")
	myQueue.Post("Hello 2 !")
	
	myQueue.CurrentQueueLength // 2
	
	myQueue.Start()
	
	myQueue.CurrentQueueLength // 0
	
	myQueue.Post("Hello 3 !")


## Actor-malli ##

Mitä jos "oliot" tilan kapseloijina korvattaisiin viestipohjaisella ratkaisulla?

Actor-malli on teoria, joka käyttää “Actor”eita rinnakkaislaskennan alkioina. Actor voi prosessoida tietoa, varastoida tietoa tai kommunikoida muiden actoreiden kanssa. Actor voi jopa ottaa vastaan viestissä itsellensä uuden toiminnallisuuden.

Actor-malli on lähtöisin Erlang-kielestä, mutta nykyään vaihtoehtoja alkaa löytyä jo monille eri kielille.

Tausta-ajatusta voisi verrata versiohallintaan: ohjelmaa ei ikinä voi ”pysäyttää ja katsoa missä tilassa se on”, vaan tila on kokoajan muuttuva käsite ja eri katsojalle näyttää erilaiselta.


## Agent-malli ##


Agent-malli on käytännössä sama asia kuin Actor-malli, ehkä sillä pienellä erolla, että agent ottaa vastaan pyyntöjä, joten rajapinta on kiinteämpi. F#:ssa Agent-malli on kielessä suoraa tuettuna ja kapseloituna luokkaan nimeltä MailboxProcessor. Usein käytännössä tämä konkretisoituu siten, että isäntä vaan kyselee asioita, ja taustasäie sykkii parsien tapahtumahistoriaa joukko-operaatioilla.

Edellinen esimerkki lähetti viestikanavana merkkijonoja. Kätevämpää on kuitenkin lähettää discriminated union komentoja ja niiden parametreja.

Staattinen MailboxProcessor.Start on sama kuin kutsuisi ensin new MailboxProcessor() ja sitten sille .Start(). Agentille on helppo rakentaa myös timeoutit, virheenkäsittely ja peruutus-operaatiot.

Testaa oheista koodia:

    [lang=fsharp]
    type ``Person agent methods`` =
    | AddPerson of string * int //Name*Age
    | GetPeople of AsyncReplyChannel<(string*int) list>

    let myQueue2 = 
        MailboxProcessor<``Person agent methods``>.Start(fun myHandler -> 
            let rec readNextMessage people =
                async {
                    let! result = myHandler.Receive()
                    match result with
                    | AddPerson(n,s) ->
                        return! readNextMessage ((n,s)::people)
                    | GetPeople(response) ->
                        let p = people
                        response.Reply(p)
                        return! readNextMessage people
                }
            readNextMessage [])

    let addPerson =  AddPerson >> myQueue2.Post 
    ("Matti", 30) |> addPerson
    ("Maija", 7) |> addPerson 

    let res = myQueue2.PostAndReply(GetPeople)

PostAndReply ohella on myös AsyncPostAndReply

### Tarvitaan lisää... ###

Tämä oli vasta yksi agent, joka ei itsessään vielä oikein eroa .NET ConcurrentDictionary<K, T>:stä muuta kuin siten, että ohjelmointirajapinta on business-loogisia metodeita, eikä CRUD-operaatioita. Mutta varsinaiset hyödyt aletaan saada vasta kun agentteja on useita:

Olion periyttämisen (tai enum-tyyppiproperyjen) sijaan käytetäänkin sitä, että agentit keskustelevat keskenään.



![](fig1.jpg)

*(kuvan lähde: [Writing Concurrent Applications Using F# Agents](http://www.developerfusion.com/article/140677/writing-concurrent-applications-using-f-agents/))*


## Harjoitustehtävä ##

#### Harjoitus 1 ####
Lisää "Person agent methods"-metodiin uusi metodi, joka palauttaa vain yli 18 vuotiaat ihmiset.

#### Harjoitus 2 ####
Muuta "Person agent methods" agentti olemaan "prosessoimattomat ihmiset" ja tee rinnalle toinen agentti, "prosessoidut ihmiset":

- Tee pieni business-logiikka ja kommunikaatio, miten agentti prosessoimaton ihminen siirtyy prosessoiduksi. 
- Vaihda rajapinta asynkroniseksi (AsyncPostAndReply)
- Voit myös kapseloida agentin tyypin sisälle.

[Takaisin valikkoon](../Readme.html)
