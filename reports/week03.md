## Johdanto
Tehtävän tarkoituksena oli kerätä palvelimen suorituskykymittareita, visualisoida niitä dashboardilla ja tutkia kuormituksen vaikutusta järjestelmän toimintaan. Mittauksen kohteena oli web1-kontti.


## Node Exporterin käyttöönotto
Node Exporter asennettiin web1-konttiin. Sen avulla kerättiin palvelimen mittareita, kuten prosessorin käyttöä, muistin käyttöä, levytilaa ja verkkoliikennettä.
Asennuksen jälkeen Node Exporter käynnistettiin ja sen toimintaa testattiin HTTP-rajapinnan kautta. Testin tuloksena mittareita saatiin näkyviin, joten Node Exporter toimi odotetusti.


## Prometheus
Prometheus oli käynnissä monitorointiympäristössä. Sen Targets-sivulta tarkistettiin Node Exporterin yhteys.
web1 näkyi Prometheuksessa tilassa UP, mikä osoitti, että Prometheus pystyi hakemaan mittareita onnistuneesti. Prometheus yhdistyi onnistuneesti myös grafanaan (kts. onnistunut_yhteys.png)


## Dashboard
Löytyy kuvakansiosta. (kts. lähtötilanne.png ja kuormituksen_jälkeen.png)


## Kuormitustesti
Kuormitustestin aikana tarkasteltiin Grafanan mittareita. CPU-kuormitustesti suoritettiin stress-ng-työkalulla. CPU Usage -kuvaajassa näkyi selviä piikkejä, joiden korkein arvo oli noin 80–85 %. 
Kuormitusjakson jälkeen prosessorin käyttö laski selvästi.
Memory Usage -kuvaajassa muistin käyttö vaihteli ja nousi kuvaajan lopussa noin 42 prosenttiin.
Network Receive- ja Network Transmit -kuvaajissa näkyi hetkellisiä liikennepiikkejä. Tämä osoittaa, että verkkoliikenteen määrä vaihteli mittausjakson aikana. Kaikki piikit eivät välttämättä johtuneet kuormitustestistä.
Disk Usage -paneeli ei suostunut toimimaan, joten sen datasta ei ole näyttöä.

## SNMP vs Prometheus
SNMP sopii erityisesti verkkolaitteiden ja niiden tilatietojen valvontaan. 
Prometheus soveltuu hyvin palvelimien, sovellusten ja konttien suorituskykymittareiden keräämiseen, kun käytössä on sopiva mittarien tarjoaja eli exporter.

## Yhteenveto
Harjoituksessa Node Exporter saatiin toimimaan, Prometheus tunnisti web1-kohteen ja Grafana yhdistettiin onnistuneesti Prometheukseen. 
Dashboardin avulla pystyttiin seuraamaan CPU:n ja verkkoliikenteen mittareita.
Harjoituksesta opittiin käytännön kokemusta monitorointityökaluista ja mittaustulosten tulkinnasta, sekä eri järjestelmien yhteiskäyttöä.
