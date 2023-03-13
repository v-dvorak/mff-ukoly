# Rekapitulace zadání

Máme figurku kulhavého koně, která se vyznačuje tím, že v sudém tahu táhne jako šachový kůň a lichém tahu jako král. Kůň existuje na nějaké šachovnici $N\times N$, na které se pro něj snažíme najít nejkratší cestu mezi dvěma zadanými body.

# Reprezentace šachovnice a tahů

## Šachovnice

Indexovat v řešení budeme od nuly a budeme předpokládat, že celou šachovnici (všechny její body) máme uložené v dvourozměrném indexovatelném poli, jestliže známe index pole, zjistíme jeho obsah v $O(1)$.

(V RAMu lze tuto 2D strukturu implementovat jako seznam $N^2$ prvků a používat jednodchou funkci na přeměnu dvou souřadnic na jednu.)

Navíc o každém tomto poli budeme vědět, ze kterého jiného pole jsme se dostali. Struktura "Policko" tedy vypada nejak takto:

```C#
class Policko {
    bool nenavstiveny = true;
    Policko = rodic;
}
```

Na začátku je hodnota rodiče nastavena na `Null` a až při příadném navštívení ji nastavíme na rodiče. Indexy prvku známe z 2D pole.

## Tahy

Máme vyřešeno indexování polí, takže můžeme použít seznamy vektorů, abychom se z jednoho pole dostali na jeho povolené sousedy.

$$
\begin{array}{|c|c|c|c|c|}
\hline
 & \times & & \times &\\\hline
\times &  \square& \square&\square &  \times \\\hline
&\square & \circ &\square &  \\\hline
\times &\square & \square& \square& \times  \\\hline
& \times& &\times &  \\\hline
\end{array}
$$

Na improvizovaném obrázku vidíme, kam se kůň ($\circ$) může pohnout v sudém ($\times$) a lichém ($\square$) tahu. Z toho odvodíme seznamu vektorů, které budeme v algoritmu dále používat pouze pod jejich jménem:

```C#
kral_tahy = {(1, 0), (1, 1), (0, 1), (-1, 1), (-1, 0), (-1, -1), (0, -1), (1, -1)};
kun_kun = {(1, 2), (2, 1), (2, -1), (1, -2), (-1, -2), (-2, -1), (-2, 1), (-1, 2)};

```

# Algoritmus

## BFS

Pro nalezení nejkratší možné cesty mezi dvěma zadanými body použijeme algoritmus BFS s jemnou úpravou. V sudém kroku volá vektory koně, v lichém krále (krok je přechod mezi hladinami) a po nalezení doposud nenavštíveného políčka do něj uloží odkaz na jeho rodiče.

## Souřadnice cesty

Pokud najdeme nějakou cestu, tak se vydáme zpátky po předcích cílového pole, až konečně najdeme pole startovní. Každé pole, které navštívíme, je součastí cesty mezi startem a cílem.

## Důkaz korektnosti

Z obecné implementace BFS plyne, že opravdu najde nejkratší cestu, naše modifikace toto tvrzení nijak neovlivňuje (jenom "chodíme divně").

Jestliže do cíle pomocí BFS nedojdeme, pak cesta mezi dvěma body neexistuje. Pakliže cestu najdeme, existuje rodič cílového bodu, skrz který jsme do cíle došli. Každé políčko má maximálně jednoho rodiče, přičemž políčka v první navštívené hladině mají za rodiče start. Zpětně tak můžeme dojít od cíle, přes jeho rodiče, přes jeho rodiče ... až do startu. Tato pole můžeme postupně vypsat, nebo uložit, a odpovídají námi hledané cestě.

# Kód algoritmu

V něm použijeme ještě výše nezmíněnou frontu.

```C#
kun_tahy;
kral_tahy;
sachovnice; // 2D pole NxN policek, viz definice tridy vyse
start = a, b
cil = c, d
fronta1;
fronta2;
krok = 0;
fronta1 <- start

dokud fronta1 není prázdná:
    dokud fronta1 není prázdná:
        krok <- krok + 1
        v <- dequeue fronta1
        pokud krok % 2 == 0:
            vektory <- kun_tahy
        jinak:
            vektory <- kral_tahy
        pro každý vektor u z vektory:
            w <- u + v
            pokud w je na šachovnici a ještě není navštívený:
                w.nenavstiveny <- false
                w.rodic <- v
                fronta2 <- w
    // díky dvěma frontám neřešíme přechody mezi vrstvami -> fronta1 dojde, jdeme na dalši vrstvu
    fronta1, fronta2 = fronta2, fronta1
```