## Zabbixiin tutustuminen

| Osio | Kuvaus |
|----------|----------|
| Hosts | Lisätään ja hallitaan valvottavia laitteita |
| Templates | Liitetään laitteisiin valmiita valvontamäärityksiä |
| Monitoring | Tarkastellaan mittareita, tapahtumia ja laitteiden tilaa |
| Dashboards | Kootaan mittarit yhteen näkymään |
| Alerts | Hallitaan hälytyksiin liittyviä asetuksia ja ilmoituksia |
| Reports | Tarkastellaan ja tuotetaan valvontaraportteja |


## Hostien lisääminen

db1 lisäämisessä tuli ongelma vastaan, koska mittareita ei näkynyt Linux by Zabbix -agentilla. Käyttöön otettiin sen sijaan Linux by Prom -agentti, jolla dataa alkoi löytyä.


## Valvontamittarien analyysi

| Mittari | Arvo | Merkitys |
|----------|----------|----------|
| CPU system time | 0.401 % | Aika tai osuus prosessorin ajasta, joka kuluu käyttöjärjestelmän ytimen tehtäviin, kuten laitteiden ja järjestelmätoimintojen käsittelyyn. Korkea arvo voi kertoa siitä, että käyttöjärjestelmä käyttää paljon prosessoriaikaa esimerkiksi laitteiden käsittelyyn tai järjestelmäkutsuihin. |
| CPU idle time | 98.1001 % | Osuus prosessorin ajasta, jolloin prosessori on joutilaana eli sillä ei ole suoritettavaa työtä. Auttaa arvioimaan, kuinka paljon prosessorilla on vapaata kapasiteettia. Pitkään jatkuva pieni joutoaika voi viitata suureen kuormitukseen. |
| Available memory | 4.62 GB | Käytettävissä olevan keskusmuistin määrä, jota järjestelmä voi hyödyntää uusien tehtävien suorittamiseen. Jos käytettävissä oleva muisti vähenee liikaa, järjestelmä voi hidastua ja joutua käyttämään levyä muistin korvikkeena. |
| Memory utilization | 38.2024 % | Kuinka suuri osuus keskusmuistista on käytössä. Auttaa havaitsemaan muistin loppumisen riskin ja mahdolliset muistivuodot tai liiallisen muistinkulutuksen. |
| Disk utilization | 12.5158 % | Kuinka suuri osuus levyn tallennuskapasiteetista on käytössä. Jos levytila täyttyy, lokien kirjoittaminen, tietokannat ja sovellukset voivat lakata toimimasta oikein. |


## Dashboardin luominen

Dashboard löytyy images-kansiosta. (kts. zabbix_dashboard.png)



## Triggerien määrittäminen

High CPU utilization: ehtona CPU utilization > 80 %, vakavuusluokka high. Tämä tunnistaa mahdollisen prosessorin ylikuormituksen.
Low free disk space: ehtona Free disk space < 20 %, vakavuusluokka warning. Tämä varoittaa levytilan loppumisesta.



## Hälytyksen simulointi

Hälytystä ei syntynyt kummastakaan triggeristä, vaikka terminaalissa on ajettu web1:ssä tehtävässä annetut molemmat komennot.



## Häiriötilanne

Valitettavasti myöskään häiriötilannetta ei onnistuttu simuloimaan tuntemattomasta syystä. Tehtävässä annetut komennot ajettiin esim web1:ssä, mutta tilanne ei tullut näkyviin Zabbixiin.



## Kurssin työkalujen vertailu

| Ominaisuus | SNMP | Prometheus | Zabbix |
|------------|------|------------|--------|
| Tiedonkeruu | Kerää verkkolaitteiden tietoja, kuten liikennettä, tilaa ja suorituskykyä | Kerää aikasarjamuotoisia mittareita | Kerää tietoja agenteilla, SNMP:llä ja muilla tarkistuksilla |
| Dashboardit | Ei sisällä omia dashboardeja; tiedot esitetään erillisissä valvontatyökaluissa | Sisältää kysely- ja visualisointiominaisuuksia | Sisältää sisäänrakennetut dashboardit ja valmiita visualisointiwidgettejä |
| Hälytykset | Voi lähettää trap-ilmoituksia, mutta ei yksin tarjoa kattavaa hälytysten hallintaa | Hälytyssääntöjä hallitaan yleensä Prometheus Alertmanagerilla | Sisältää triggerit, ongelmien seurannan ja hälytysten ilmoituskanavat |
| Käyttöönotto | Usein helppo ottaa käyttöön, jos laite tukee SNMP:tä | Edellyttää Prometheuksen, mittauskohteiden ja tarvittaessa exporterien määrittämistä | Sisältää enemmän toimintoja samassa järjestelmässä |
| Skaalautuvuus | Soveltuu laajojen verkkolaitemäärien valvontaan, mutta kapasiteetti riippuu kyselytiheydestä ja kerääjästä | Soveltuu hyvin aikasarjamittareiden keräämiseen | Skaalautuu pienistä ympäristöistä suuriin yritysverkkoihin |
| Yrityskäyttö | Erittäin yleinen verkkolaitteiden valvonnassa, kuten kytkimissä, reitittimissä ja tulostimissa | Yleinen erityisesti pilvi-, kontti- ja sovellusympäristöjen mittareiden seurannassa | Soveltuu kokonaisvaltaiseen infrastruktuurin, palvelinten, verkkojen ja sovellusten valvontaan |



## Pohdinta

1. Keskitetyn valvonnan avulla kaikkien palvelimien ja verkkolaitteiden tilaa voidaan seurata yhdestä paikasta. Tämä helpottaa ongelmien havaitsemista ja vianetsintää, koska mittarit ja hälytykset ovat samassa järjestelmässä.
2. Mielestäni tärkeimpiä mittareita ovat prosessorin kuormitus, käytettävissä oleva muisti, muistin käyttöaste, vapaa levytila ja verkkoliikenne. Niiden avulla voidaan havaita esimerkiksi palvelimen ylikuormitus, muistin loppuminen, levytilan täyttyminen ja verkon ruuhkautuminen.
3. Ylläpitäjän pitäisi saada hälytys esimerkiksi silloin, kun prosessorin kuormitus on pitkään liian korkea, vapaa levytila vähenee kriittisesti tai muisti on lähes loppunut. Hälytys tarvitaan myös silloin, kun palvelin tai tärkeä verkkopalvelu ei vastaa tai verkkoyhteys katkeaa.
4. Erityisesti sovellusten, palveluiden ja konttiympäristöjen mittareiden keräämiseen ja analysointiin.
5. Käyttäisin Zabbixia silloin, kun halutaan valvoa keskitetysti monenlaisia laitteita ja palvelimia samassa järjestelmässä.
6. Lisäisin ympäristöön palveluiden vastausaikojen ja käytettävyyden seurannan sekä hälytykset levytilan, muistin ja prosessorin kriittisistä arvoista. Lisäksi ottaisin käyttöön lokien seurannan, jotta virhetilanteiden syitä olisi helpompi selvittää.


