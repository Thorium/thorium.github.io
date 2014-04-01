# Agent-malli ja Actor-malli #

Puhuttaessa Actor-mallista termi "Actor" viittaa "toimijaan" tai "tekijään", ei "näyttelijään". Puhuttaessa Agent-mallista termi "Agent" viittaa myös "toimijaan" tai "edustajaan", mutta ei "salaiseen agenttiin".

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

Actor-malli on teoria, joka käyttää “Actor”eita rinnakkaislaskennan alkioina. Actor voi prosessoida tietoa, varastoida tietoa tai kommunikoida muiden actoreiden kanssa. Actor voi jopa ottaa vastaan viestissä itselleen uuden toiminnallisuuden.

Actor-malli on lähtöisin Erlang-kielestä, mutta nykyään vaihtoehtoja alkaa löytyä jo monille eri kielille.

Tausta-ajatusta voisi verrata versiohallintaan: ohjelmaa ei ikinä voi ”pysäyttää ja katsoa missä tilassa se on”, vaan tila on kokoajan muuttuva käsite ja eri katsojalle näyttää erilaiselta.


## Agent-malli ##

Agent-malli on käytännössä sama asia, ehkä sillä pienellä erolla, että agent ottaa vastaan pyyntöjä, joten rajapinta on kiinteämpi. F#:ssa Agent-malli on kielessä suoraa tuettuna ja kapseloituna luokkaan nimeltä MailboxProcessor. Usein käytännössä tämä konkretisoituu siten, että isäntä vaan kyselee asioita, ja taustasäie sykkii parsien tapahtumahistoriaa joukko-operaatioilla.

Edellinen esimerkki lähetti viestikanavana merkkijonoja. Kätevämpää on kuitenkin lähettää discriminated union komentoja ja niiden parametreja.

Staattinen MailboxProcessor.Start on sama kuin kutsuisi ensin new MailboxProcessor() ja sitten sille .Start(). Agentille on helppo rakentaa myös timeoutit, virheenkäsittely ja peruutus-operaattiot.

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


Jos haluat testailla nopeuksia ja asynkronisuutta, saat päälle ajanoton interactiveen kirjoittamalla:
 
    #time "on";;


## Harjoitustehtävä ##

Lisää "Person agent methods"-metodiin uusi metodi, joka palauttaa vain yli 18 vuotiaat ihmiset.

Jos kiinnostus riittää, voit myös:
- Vaihtaa tämän asynkroniseksi ja rakentaa toisen mailboxin, joka lähettää viestejä tälle.
- Kapseloida mailboxin oman tyyppin sisälle, jotta sitä voisi käyttää C#-kirjastosta.

[Takaisin valikkoon](../Readme.html)
