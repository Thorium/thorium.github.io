
# F# Azure Hands-on #

Sorry only Finnish as this event is in Finnish.
Meanwhile, you can use [this](http://www.tryfsharp.org/Learn).

Harjoituksia on kolmea tasoa, ja ne voi tehdä tason sisällä haluamassaan järjestyksessä.

### Alustavat harjoitukset ###

Näistä on hyvä aloittaa, jos aihe ei ole ennestään tuttu.

- [**Azure Worker -roolin luonti**](1-AzureWorkerRole/AzureWorkerRoleFin.html)
	- Tässä harjoituksessa luodaan worker-role, jota käytetään pohjana muissa harjoituksissa.
- [**F# lyhyt oppimäärä**](1-FSharpBasics/FSharpBasicsFin.html)
	- Tässä harjoituksessa käydään läpi perusosia kielen syntaksista. 

### Edistyneemmät harjoitukset ###

Nämä ovat kaikki vain hyvin kevyitä alustuksia laajempiin aiheisiin.

- [**Informaatiorikas ohjelmointikieli**](2-DataUsage/DataUsageFin.html)
	- F#:ssa on simppeli käyttää mitä tahansa tietolähdettä ja alkaa koodata suoraa sitä vastaan, kääntäjä kääntää tietomallin osaksi kieltä ja IDE:ä.
- [**Azure Blob- ja Table Storage**](2-AzureStorage/AzureStorage.html)
	- Azuren tietovarastojen käyttö F#-kielestä.
- [**Actor-malli ja Agent-malli**](2-AgentModel/AgentModelFin.html)
	- Erlang-kielestä tunnettu malli toteuttaa ohjelman tila niin, että se on hajautettu: ei lukkoja, vaan message-passingia. Ohjelmaa ei ikinä voi ”pysäyttää ja katsoa missä tilassa se on”, vaan tila on kokoajan muuttuva käsite ja eri katsojalle näyttää erilaiselta.
- [**Domain-mallinnus, DSL-kieli**](2-DomainModel/DomainModelFin.html)
	- Omia custom-”yleiskäyttöinen” workflow/sääntö/prosessi/ -moottori (tai framework) -viritelmiä on maailma täynnä. Silti ne eivät palvele tarkoitustaan. 
	- Primitiivien valinta, kompositio, syntaksi
- [**Kevyt WWW-palvelin käyttäen OWIN-rajapintaa ja SignalR-viestitystä**](2-AzureOwinWww/AzureOwinWwwFin.html)

### Itsenäistä koodausta ###

Käyttäen edellisten harjoitusten tuloksia, sinulla on eväät toteuttaa mitä vain. Tässä peruslähtökohtia/mallitoteutuksia muutamiin aiheisiin:

- [**F#, pilvipalvelut ja yhteisöpelit**](3-SocialGame\SocialGameFin.html)
- [**Osake/raaka-aine-kauppasofta**](3-TradingSoftware\TradingSoftwareFin.html)

