# Hajautustaulu

Hajautustaulu
Hajautustaulu on toteutettu taulukkona, missä jokainen alkio sisältää listan. Listalle tallennetaan (avain,arvo)-pareja. Käyttäjä voi hakea hajautustaulusta arvoja avaimen 
perusteella, ja toisaalta käyttäjä voi lisätä hajautustauluun avain-arvo -pareja. Kukin avain voi esiintyä hajautustaulussa korkeintaan kerran.

Hajautustaulun toiminta perustuu avaimen hajautusarvoon. Kun hajautustauluun lisätään (avain,arvo)-pari, lasketaan avaimeen liittyvä hajautusarvo. Hajautusarvo määrää 
hajautustaulun sisäisen taulukon indeksin, missä olevaan listaan (avain,arvo)-pari lisätään.

Hahmotellaan hajautustaulun toimintaa.

Avain-arvo -pari
Luodaan ensin avain-arvo -paria kuvaava luokka Pari. Haluamme tehdä hajautustaulusta mahdollisimman yleiskäyttöisen, joten avaimen ja arvon tyyppi määrätään ajonaikaisesti. 
Pari sisältää avaimen ja arvon sekä niihin liittyvät get-metodit. Geneeriset tyypit K ja V ovat nimetty sanojen key ja value perusteella.

public class Pari<K, V> {

    private K avain;
    private V arvo;

    public Pari(K avain, V arvo) {
        this.avain = avain;
        this.arvo = arvo;
    }

    public K getAvain() {
        return avain;
    }

    public V getArvo() {
        return arvo;
    }

    public void setArvo(V arvo) {
        this.arvo = arvo;
    }
}
Avain-arvo -parien luominen on suoraviivaista.

Pari<String, Integer> pari = new Pari<>("yksi", 1);
System.out.println(pari.getAvain() + " -> " + pari.getArvo());
Esimerkkitulostus
yksi -> 1

Hajautustaulun luominen
Hajautustaulu sisältää taulukon listoja. Jokainen listan arvo on edellä kuvattu pari, joka sisältää avain-arvo -parin. Hajautustaululla on lisäksi tieto arvojen lukumäärästä. 
Tässä käytössämme on edellä luotu luokka Lista.

public class Hajautustaulu<K, V> {

    private Lista<Pari<K, V>>[] arvot;
    private int arvoja;

    public Hajautustaulu() {
        this.arvot = new Lista[32];
        this.arvoja = 0;
    }
}
Arvon hakeminen
Toteutetaan ensin metodi public V hae(K avain), jota käytetään arvon hakemiseen avaimen perusteella.

Metodissa lasketaan ensin avaimen hajautusarvo ja päätellään sen perusteella hajautustaulun sisäisen taulukon indeksi, mistä arvoja haetaan. Avaimen hajautusarvon laskemiseen 
käytetään jokaisella oliolla olevaa hashCode-metodia. Jakojäännöstä käytetään hajautusarvon hajautustaulun koon sisällä pysymiseen.

Mikäli hajautusarvon ja jakojäänneksen avulla lasketussa indeksissä ei ole listaa, ei indeksiin ole lisätty vielä yhtäkään avain-arvo -paria, eikä avaimelle ole tallennettu 
arvoa. Tällöin palautetaan null-viite. Muussa tapauksessa taulukon indeksissä oleva lista käydään läpi, ja avaimen yhtäsuuruutta vertaillaan jokaiseen listan avain-arvo -parin
avaimeen. Mikäli joku listalla olevista avaimista vastaa avainta, jonka perusteella arvoa haetaan, palautetaan kyseinen arvo. Muulloin avainta (ja siihen liittyvää arvoa) ei
löydy, ja palautetaan arvo null.

public V hae(K avain) {
    int hajautusArvo = Math.abs(avain.hashCode() % this.arvot.length);
    if (this.arvot[hajautusArvo] == null) {
        return null;
    }

    Lista<Pari<K, V>> arvotIndeksissa = this.arvot[hajautusArvo];

    for (int i = 0; i < arvotIndeksissa.koko(); i++) {
        if (arvotIndeksissa.arvo(i).getAvain().equals(avain)) {
            return arvotIndeksissa.arvo(i).getArvo();
        }
    }

    return null;
}
Miksei hajautustaulua toteuteta listana?
Hajautustaulun toimintaperiaate perustuu siihen, että avain-arvo -parit jaetaan hajautusarvon perusteella pieniin joukkoihin. Tällöin avaimen perusteella haettaessa käydään 
läpi vain hyvin pieni joukko avain-arvo -pareja — olettaen toki, että hajautusarvo on järkevä.

Jos hajautusarvo on aina sama — esimerkiksi 1 — vastaa hajautustaulun sisäinen toteutus listaa — kaikki arvot ovat samalla listalla. Jos taas hajautusarvo on hyvin satunnainen, 
arvot hajautetaan mahdollisimman tasaisesti taulukon eri listoille.

Hajautustaulu toimii lisäksi siten, että hajautustaulun käyttämää taulukkoa kasvatetaan mikäli arvojen määrä on tarpeeksi iso (tyypillisesti noin 75% taulukon koosta). 
Tyypillisesti miljoonia avain-arvo -pareja sisältävän hajautustaulun taulukon yhdessä indeksissä on vain muutama avain-arvo -pari. Tämä tarkoittaa käytännössä sitä, että 
avain-arvo -parin olemassaolon selvittämiseen tarvitaan vain hajautusarvon laskeminen sekä muutaman olion tarkastelu — tämä on paljon nopeampaa kuin listan läpikäynti.

Hajautustauluun lisääminen, osa 1
Toteutetaan hajautustauluun lisäämisen käytettävän metodin public void lisaa(K avain, V arvo) ensimmäinen versio. Ensimmäisessä versiossa hajautustaulun sisältämän taulukon 
kokoa ei kasvateta lisäyksen yhteydessä.

Metodi laskee ensin avaimelle hajautusarvon ja päättelee hajautusarvon perusteella hajautustaulun sisäisen taulukon indeksin. Jos taulukon kyseisessä indeksissä ei ole arvoa, 
taulukon indeksiin lisätään lista. Tämän jälkeen taulukon indeksissä oleva lista käydään läpi ja sieltä etsitään avain-arvo -paria, jonka avain vastaa lisättävän avain-arvo 
-parin avainta. Mikäli vastaava avain löytyy, päivitetään olemassaolevan avain-arvo -parin arvo vastaamaan uutta avainta. Muulloin listaan lisätään uusi avain-arvo -pari — 
tällöin myös hajautustaulussa olevien arvojen lukumäärää kasvatetaan yhdellä.

public void lisaa(K avain, V arvo) {
    int hajautusArvo = Math.abs(avain.hashCode() % arvot.length);
    if (arvot[hajautusArvo] == null) {
        arvot[hajautusArvo] = new Lista<>();
    }

    Lista<Pari<K, V>> arvotIndeksissa = arvot[hajautusArvo];

    int indeksi = -1;
    for (int i = 0; i < arvotIndeksissa.koko(); i++) {
        if (arvotIndeksissa.arvo(i).getAvain().equals(avain)) {
            indeksi = i;
            break;
        }
    }

    if (indeksi < 0) {
        arvotIndeksissa.lisaa(new Pari<>(avain, arvo));
        this.arvoja++;
    } else {
        arvotIndeksissa.arvo(indeksi).setArvo(arvo);
    }
}
Metodi on melko monimutkainen. Pilkotaan se pienempiin osiin — ensimmäisen osan vastuulla on avaimeen liittyvän listan hakeminen ja toisen osan vastuulla on avaimen 
indeksin etsiminen listalta.

private Lista<Pari<K, V>> haeAvaimeenLittyvaLista(K avain) {
    int hajautusArvo = Math.abs(avain.hashCode() % arvot.length);
    if (arvot[hajautusArvo] == null) {
        arvot[hajautusArvo] = new Lista<>();
    }

    return arvot[hajautusArvo];
}

private int haeAvaimenIndeksi(Lista<Pari<K, V>> lista, K avain) {
    for (int i = 0; i < lista.koko(); i++) {
        if (lista.arvo(i).getAvain().equals(avain)) {
            return i;
        }
    }

    return -1;
}
Nyt metodi public void lisaa(K avain, V arvo) voidaan toteuttaa hieman selkeämmin.

public void lisaa(K avain, V arvo) {
    Lista<Pari<K, V>> arvotIndeksissa = haeAvaimeenLittyvaLista(avain);
    int indeksi = haeAvaimenIndeksi(arvotIndeksissa, avain);

    if (indeksi < 0) {
        arvotIndeksissa.lisaa(new Pari<>(avain, arvo));
        this.arvoja++;
    } else {
        arvotIndeksissa.arvo(indeksi).setArvo(arvo);
    }
}
Hajautustauluun lisääminen, osa 2
Edellä kuvattu hajautustauluun lisääminen toimii osittain. Toiminnallisuuden suurin puute on se, että taulukon kokoa ei kasvateta kun arvojen määrä kasvaa liian suureksi. 
Lisätään ohjelmaan kasvatustoiminnallisuus, mikä tuplaa hajautustaulun sisäisen taulukon koon. Kasvatustoiminnallisuuden tulee myös sijoittaa jokainen hajautustaulussa olevan 
taulukon arvo uuteen taulukkoon.

Hahmotellaan kasvatustoiminnallisuuden alku. Kasvatustoiminnallisuudessa luodaan uusi taulukko, jonka koko on edelliseen verrattuna kaksinkertainen. Tämän jälkeen alkuperäinen
taulukko käydään indeksi indeksiltä läpi ja olemassaolevat avain-arvo -parit kopioidaan uuteen taulukkoon. Lopulta alkuperäinen taulukko korvataan uudella taulukolla.

Alla on hahmoteltu metodin toimintaa. Kopiointia ei ole vielä toteutettu.

private void kasvata() {
    // luodaan uusi taulukko
    Lista<Pari<K, V>>[] uusi = new Lista[this.arvot.length * 2];

    for (int i = 0; i < this.arvot.length; i++) {
        // kopioidaan vanhan taulukon arvot uuteen

    }

    // korvataan vanha taulukko uudella
    this.arvot = uusi;
}
Hahmotellaan seuraavaksi metodia, joka kopioi alkuperäisen taulukon yhden indeksin sisältämän listan arvot uuteen taulukkoon. Kopioinnin yhteydessä jokaisen kopioitavan 
avain-arvo -parin sijainti taulukossa lasketaan uudelleen — tämä tehdään, sillä taustalla olevan taulukon koko kasvaa ja avain-arvot -parit halutaan sijoittaa taulukkoon 
mahdollisimman tasaisesti.

private void kopioi(Lista<Pari<K, V>>[] uusi, int indeksista) {
    for (int i = 0; i < this.arvot[indeksista].koko(); i++) {
        Pari<K, V> arvo = this.arvot[indeksista].arvo(i);

        int hajautusarvo = Math.abs(arvo.getAvain().hashCode() % uusi.length);
        if(uusi[hajautusarvo] == null) {
            uusi[hajautusarvo] = new Lista<>();
        }

        uusi[hajautusarvo].lisaa(arvo);
    }
}

Nyt kopioi-metodia voidaan kutsua kasvata-metodista.

private void kasvata() {
    // luodaan uusi taulukko
    Lista<Pari<K, V>>[] uusi = new Lista[this.arvot.length * 2];

    for (int i = 0; i < this.arvot.length; i++) {
        // kopioidaan vanhan taulukon arvot uuteen
        kopioi(uusi, i);
    }

    // korvataan vanha taulukko uudella
    this.arvot = uusi;
}
Lisätään lopuksi kasvatustoiminnallisuus osaksi lisäystoiminnallisuutta. Hajautustaulun kokoa kasvatetaan aina jos hajautustaulussa olevien avain-arvo -parien määrä on yli 
75% taulukon koosta.

public void lisaa(K avain, V arvo) {
    Lista<Pari<K, V>> arvotIndeksissa = haeAvaimeenLittyvaLista(avain);
    int indeksi = haeAvaimenIndeksi(arvotIndeksissa, avain);

    if (indeksi < 0) {
        arvotIndeksissa.lisaa(new Pari<>(avain, arvo));
        this.arvoja++;
    } else {
        arvotIndeksissa.arvo(indeksi).setArvo(arvo);
    }

    if (1.0 * this.arvoja / this.arvot.length > 0.75) {
        kasvata();
    }
}
Poistaminen
Lisätään hajautustauluun vielä toiminnallisuus avain-arvo -parin poistamiseen avaimen perusteella. Poistotoiminnallisuus palauttaa null-arvon mikäli arvoa ei löydy, 
muuten metodi palauttaa poistettavaan avaimeen liittyvän arvon.

Voimme hyödyntää valmiiksi toteuttamiamme metodeja poistotoiminnallisuudessa. Selitä itsellesi (ääneen) alla olevan metodin konkreettinen toiminta.

public V poista(K avain) {
    Lista<Pari<K, V>> arvotIndeksissa = haeAvaimeenLittyvaLista(avain);
    if (arvotIndeksissa.koko() == 0) {
        return null;
    }

    int indeksi = haeAvaimenIndeksi(arvotIndeksissa, avain);
    if (indeksi < 0) {
        return null;
    }

    Pari<K, V> pari = arvotIndeksissa.arvo(indeksi);
    arvotIndeksissa.poista(pari);
    return pari.getArvo();
}

******
VARSINAINEN TEHTÄVÄNANTO
******
Toteuta tehtäväpohjaan edellistä esimerkkiä noudattaen luokka Hajautustaulu. Toisin kuin esimerkissä, toteuta luokka siten, että se hyödyntää sisäisessä toteutuksessa 
Listan sijaan Javan valmista luokkaa ArrayList. Tehtäväpohjassa ei ole testejä — kokeile listaa materiaalin esimerkkien ja omien kokeilujen avulla. Tehtävä on kolmen
pisteen arvoinen.

