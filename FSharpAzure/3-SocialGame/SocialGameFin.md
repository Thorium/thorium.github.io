# F#, pilvipalvelut ja yhteisöpelit #

Referenssinä, yli miljoonan käyttäjän Facebook-peli "[Here be monsters](https://www.facebook.com/HereBeMonstersGame)":

- Käyttää taustalogiikkana F#-agent-mallia
- Hostataan virtuaalikoneissa pilvipalvelussa

Tämän pelin pääarkkitehti on tehnyt aiheesta malli-samplen: [**SharpVille**](https://github.com/theburningmonk/SharpVille) on Farmville-tyylinen peli.

[**CatVsDog**](https://github.com/thorium/CatVsDog/) on samassa hengessä tehty pohja, joka toimii Azure-worker-roolila, käyttäen OWIN-palvelinta, Agent-mallia ja Azuren Table ja Blob -storagea, Facebook/Google-autentikaatiota, SignalR-kommunikaatiota ja HTML5 (Knockout.js) -käyttöliittymää.

Näitä mallina käyttäen voit luoda kohtuullisella työmäärällä oman pelisi.


[Takaisin valikkoon](../Readme.html)