
# Osake-/raaka-aine-kauppasofta #

Koska F# on isojen Lontoon pankkien suosiossa, niin Microsoft on tehnyt myös hyvän [tutoriaalin](http://www.tryfsharp.org/Learn/financial-computing#understanding-european-options) aiheen matemaattiseen puoleen.

Referenssinä tästä esim. [TrayPort](http://www.trayport.com/), joka tekee F#-pohjaista ohjelmistoa energia-treidaukseen. [Jane's Street](https://blogs.janestreet.com/caml-trading-talk-at-cmu/) niminen firma taas tekee OCaml-kielellä osakekauppasoftaa. F# on käytännössä OCaml-kieli Microsoftin laajennuksilla.

Funktionaalisen kielen hyviä puolia on mm. se, että yksittäisiä funktioita voi kätevästi netistä ja suoraa liimata osaksi omaa softaa. [Tässä](http://www.fssnip.net/pages/AllTags) malliesimerkki kirjastosta.


## Random ei riitä? ##

Vähän .NET-oletusta parempi satunnaisgeneraattori löytyy kirjastosta System.Security.Cryptography luokka RNGCryptoServiceProvider.

Myös enemmän satunnaisia satunnaisuuksia saa esim. Nugetista löytyvästä  kirjastosta "[MathNet.Numerics.FSharp](https://www.nuget.org/packages/MathNet.Numerics.FSharp)".

    [lang=fsharp]
    #if INTERACTIVE
    #r "MathNet.Numerics.dll"
    #endif
    open MathNet.Numerics.Random
    open MathNet.Numerics.Statistics
    
    let mcg31m1 = Mcg31m1()
    let palf = Palf()
    let wh2006 = WH2006()
    
    mcg31m1.Next(100)
    palf.Next(100)
    wh2006.Next(100)
    

## Julkinen kaupparajapinta ##

Integraatiorajapinta QuickFix-nimiseen komponenttiin [täällä](https://github.com/joastbg/fsharp-for-quantitative-finance/blob/master/Chapter07/FIX.fs).

[QuickFix](http://www.quickfixengine.org/) on open source C++ implementaatio Fix-protokollalle. 

[Fix-protokolla](http://www.fixtradingcommunity.org/) on avoin yleinen finanssi-markettipuolella käytetty protokolla.
 

[Takaisin valikkoon](../Readme.html)
