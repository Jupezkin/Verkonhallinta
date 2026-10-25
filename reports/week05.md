Viikko 5 – Tietoturva, lokit ja verkkoliikenteen analysointi

1. Johdanto

Tässä tehtävässä tutkittiin verkkoliikennettä, porttiskannausta, DNS-kyselyitä ja palvelimen lokeja. Tavoitteena oli oppia tunnistamaan epäilyttävää verkkoliikennettä ja arvioimaan verkon tietoturvaa.

2. Liikenteen kaappaus

Liikennettä tallennettiin tcpdumpilla ja tutkittiin Wiresharkilla. Porttiskannaus tallennettiin tiedostoon captures/week05.pcap, ja DNS-liikenne tiedostoon captures/week05-dns3.pcap.
Tehtävänannossa annettua IP-osoitetta 10.10.20.101 ei saatu tavoitettua. 
Siksi käytettiin web1-kontin osoitetta 172.20.20.5. Attacker-kontin osoite oli 172.20.20.16.

3. Porttiskannaus

Porttiskannaus tehtiin attacker-kontista web1-konttiin Nmapilla. Skannauksessa tarkistettiin 1000 TCP-porttia. 
Nmap ilmoitti kohteen olevan toiminnassa, mutta kaikki tarkistetut portit olivat suljettuja. Myös erikseen tarkistetut portit 80, 8080, 8000, 3000 ja 5000 olivat suljettuja.
Wiresharkissa näkyi paljon SYN-paketteja attacker-kontilta web1-konttiin eri porteille. 
Tämä on tyypillistä porttiskannaukselle. Kaappauksessa näkyi myös RST/ACK-vastaus kohteelta. 
Se tarkoittaa, että yhteyttä ei hyväksytty kyseisessä portissa.

Kuva 1: Nmap-skannauksen SYN-paketit.
Kuva 2: Kohteen RST-vastaus.

4. DNS- ja HTTP-liikenne

DNS
DNS-liikennettä tutkittiin Wiresharkissa suodattimella dns. Attacker-kontista tehtiin dig example.com -kysely.
Kaappauksessa näkyi DNS-kysely verkkotunnukselle example.com ja siihen saatu vastaus. 
Kysely kulki ensin Dockerin paikallisen DNS-palvelun kautta ja sen jälkeen DNS-palvelimelle 10.255.255.254. 
Vastauksessa saatiin kaksi IP-osoitetta: 172.66.147.243 ja 104.20.23.154.

Kuva 3: DNS-kysely ja vastaus Wiresharkissa.
HTTP
HTTP-yhteyttä kokeiltiin web1-kontin osoitteeseen 172.20.20.5, mutta yhteyttä ei saatu muodostettua. Tarkistuksessa portti 80 ei ollut kuuntelevana.
Nginxin lokissa oli aiempia paikallisia HTTP-pyyntöjä, joihin oli vastattu koodilla 200. Tämä tarkoittaa, että pyynnöt olivat onnistuneet. 
Näitä pyyntöjä ei kuitenkaan havaittu tässä verkkokaappauksessa, joten HTTP-liikenteestä ei saatu tarkempaa pakettianalyysiä.
Kuva 4: HTTP-yhteyden ja porttien tarkistus.

5. Lokianalyysi

Palvelimen lokeista tutkittiin Nginxin ja node_exporterin lokitiedostoja. Syslog- tai journal-lokeja ei ollut käytettävissä.
Nginxin lokissa näkyi kolme aiempaa paikallista HTTP-pyyntöä, joihin oli vastattu koodilla 200. Nginxin virheloki oli tyhjä.
Node_exporterin lokissa mainittiin portti 9100 ja se, ettei TLS-salausta ollut käytössä. Tämä oli aiempi lokimerkintä, eikä tarkistushetkellä varmistettu, että palvelu oli edelleen käynnissä.
Lokeista ei löytynyt suoraa merkintää Nmap-skannauksesta. Tämä osoittaa, että kaikki verkkotapahtumat eivät välttämättä näy sovellusten lokeissa.

6. Tietoturva-arvio

Havaitut hyvät asiat:
Nmapin tarkistamat TCP-portit olivat suljettuja.
Kohde vastasi suljettuun porttiin RST/ACK-paketilla.
Nginxin virhelokissa ei ollut merkintöjä.

Havaitut riskit:
Porttiskannaus voi olla merkki verkon kartoittamisesta ennen mahdollista hyökkäystä.
Lokeja oli rajallisesti saatavilla, joten tapahtumien selvittäminen oli vaikeampaa.
Node_exporterin lokissa TLS oli pois käytöstä. Jos palvelu on verkossa saavutettavissa, tämä voi aiheuttaa riskin.

Parannusehdotukset:
Kerätään järjestelmän ja palveluiden lokit talteen.
Seurataan toistuvia yhteydenottoja eri portteihin.
Sallitaan verkosta vain tarvittavat yhteydet palveluihin.
Tarkistetaan, että hallintapalvelut ovat vain tarvittavien laitteiden käytettävissä.

7. Yhteenveto ja pohdinta

1. Miten porttiskannaus näkyi verkkoliikenteessä?
Se näkyi useina SYN-paketteina eri porteille. Kohde vastasi ainakin yhteen yhteydenottoon RST/ACK-paketilla.

2. Miten porttiskannaus näkyi lokeissa?
Skannauksesta ei löytynyt suoraa merkintää tarkistetuista lokeista. Nginxin lokissa näkyi vain aiempia paikallisia HTTP-pyyntöjä.

3. Mitä hyötyä verkkoliikenteen analysoinnista on?
Sen avulla voidaan nähdä yhteydenottoyritykset, DNS-kyselyt ja palvelinten vastaukset. Se auttaa havaitsemaan epäilyttävää toimintaa.

4. Miten valvonta voisi auttaa hyökkäysten havaitsemisessa?
Valvonta voisi ilmoittaa, jos samasta osoitteesta yritetään ottaa yhteyttä suureen määrään eri portteja. Näin mahdollinen porttiskannaus huomattaisiin aikaisemmin.


Kuvat: 
Nmap-skannauksen SYN-paketit
reports/images/week05-nmap-syn.png

Nmap-skannauksen RST-vastaus
reports/images/week05-nmap-rst.png

DNS-kysely ja vastaus
reports/images/week05-dns.png

HTTP-porttien tarkistus
reports/images/week05-http-ports.png
