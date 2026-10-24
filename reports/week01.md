## Johdanto
Verkkoympäristön tarkoitus on opettaa insinööriopiskelijoita tutkimaan ja hallitsemaan verkkoa. Harjoituksen tavoitteena on tutustua virtuaaliseen verkkoympäristöön ja dokumentoida sen rakenne, laitteet, IP-osoitteet sekä reititys.



## 1.1 Topologian kartoitus


r1: Käyttäjäverkon reititin ja yhteys muihin verkkoalueisiin.

r2: Palvelin- ja hallintaverkkojen keskeinen reititin.

r3: Sivutoimipisteen verkon reititin.

client1: Käyttäjäverkon asiakaslaite, jolla suoritetaan yhteys- ja reititystestit.

attacker: Käyttäjäverkossa sijaitseva testaukseen tarkoitettu laite.

web1: Palvelinverkossa sijaitseva verkkopalvelin.

db1: Palvelinverkossa sijaitseva tietokantapalvelin.

branch-client: Sivutoimipisteen verkon asiakaslaite.

ansible: Verkon laitteiden hallintaan ja automatisointiin tarkoitettu järjestelmä.

prometheus: Valvontajärjestelmä, joka kerää mittareita laitteista ja palveluista.

grafana: Valvontatietojen visualisointiin tarkoitettu järjestelmä.

zabbix: Verkon ja palveluiden valvontaan tarkoitettu järjestelmä.



## 1.2 Verkkokaavio

Löytyy kuvatiedostosta.



## 1.3 – IP-osoitteiden dokumentointi

| Verkko | Tarkoitus | Yhdyskäytävä | Laitteet | Käyttötarkoitus |
|---------|---------|---------|---------|---------|
| 10.10.10.0/24 | Käyttäjäverkko | 10.10.10.1 | client1, attacker, r1 | Käyttäjälaitteiden verkkoyhteydet ja tietoturvatestaukset |
| 10.10.20.0/24 | Palvelinverkko | 10.10.20.1 | web1, db1, r2 | Verkkopalveluiden ja tietokantapalveluiden tarjoaminen |
| 10.10.30.0/24 | Sivutoimipisteen verkko | 10.10.30.1 | branch-client, r3 | Sivutoimipisteen asiakaslaitteiden verkkoyhteydet |
| 10.10.99.0/24 | Hallintaverkko | 10.10.99.1 | ansible, prometheus, grafana, zabbix | Verkon laitteiden hallinta, automatisointi, valvonta ja lokitus |
| 10.255.12.0/30 | Reitittimien välinen yhteys | ei | r1 ja r2 | Käyttäjäverkon ja muiden verkkojen välisen liikenteen reititys |
| 10.255.23.0/30 | Reitittimien välinen yhteys | ei | r2 ja r3 | Sivutoimipisteen yhdistäminen muuhun verkkoon |




## 1.4 – Reitityksen tutkiminen

### Laitteiden tila ja client1:n reititystaulu

Containerlabin "inspect"-komennolla selvisi, että kaikki 15 konttia olivat käynnissä. 

Client1:n reititystaulusta saatiin seuraavat tiedot:

- IP-osoite: "172.20.20.5"
- Suoraan kytketty verkko: "172.20.20.0/24"
- Oletusreitti: "default via 172.20.20.1 dev eth0"

Reititystaulun perusteella client1 lähettää muihin kuin suoraan kytkettyihin verkkoihin tarkoitetun liikenteen oletusyhdyskäytävälle "172.20.20.1". Reititystaulussa näkyvä verkko ja oletusreitti eivät kuitenkaan yksin vahvista, että liikenne saavuttaa kohdeverkot onnistuneesti.

### Ping-testit

Yhteyksiä testattiin client1-laitteelta kahteen IP-osoitteeseen: 10.10.20.101 ja 10.10.30.101.

Molemmissa testeissä lähetettiin neljä ICMP-pakettia, mutta yhtään vastausta ei saatu. Pakettihävikki oli molemmissa testeissä 100 %. Pelkkien ping-tulosten perusteella ei voitu päätellä, johtuiko epäonnistuminen reitityksestä, kohdelaitteiden tilasta, osoitteistuksesta vai jostain muusta.

### Traceroute

Reittiä osoitteeseen 10.10.30.101 tutkittiin traceroute-komennolla. Tulosteessa näkyivät seuraavat hypyt:

| Hyppy | Havaittu osoite tai tulos |
|---|---|
| 1 | 172.20.20.1 |
| 2 | 172.28.192.1 |
| 3 | 10.71.119.58 |
| 4 | 172.20.50.71 sekä vastaamattomia probeja |
| 5 | 172.20.50.72 ja 172.20.50.70 |

Ensimmäinen hyppy vastaa client1:n reititystaulussa määritettyä oletusyhdyskäytävää. Seuraavilla hypyillä näkyy muita IP-osoitteita. Neljännellä hypyllä kaikki lähetetyt probe-paketit eivät saaneet vastausta, ja viidennellä hypyllä vastaukset tulivat eri osoitteista.

Traceroute-tuloksesta ei voida päätellä kohteen saavutettavuutta, koska tulosteessa ei näy kohdeosoitetta 10.10.30.101.

### Reitityksen yhteenveto

Testien perusteella client1:n reititystaulussa oli oletusreitti, mutta yhteyttä kumpaankaan testattuun kohdeosoitteeseen ei saatu ping-testillä. Traceroute näytti useita välissä olevia IP-osoitteita, mutta ei vahvistanut reitin päätymistä kohdeosoitteeseen.



## Tehtävän yhteenveto

Eniten aikaa kulutti oikeiden komentojen löytäminen command promptissa, koska väärässä tilassa (Verkonhallinta-kansio, client1, r1, jne) ei yllättäen löydy oikeaa tietoa. Dokumentaatio auttaa IT-asiantuntijaa ylläpitämään verkkoa, paikantamaan yhteysongelmia ja suunnittelemaan muutoksia. 
