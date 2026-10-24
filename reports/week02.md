Viikko 2 – SNMP ja verkon perustason valvonta

1: Johdanto

SNMP on protokolla, jolla kerätään tietoa verkkolaitteista ja palvelimista. 
Manager lähettää kyselyitä ja SNMP-agentti vastaa niihin. 
SNMPä käytetään laitteiden tilan ja verkkorajapintojen valvontaan.

2: Asennus

Asensin SNMP-paketit web1-, db1- ja branch-client-laitteille komennoilla:

apt update
apt install snmp snmpd -y

Web1-laitteella muutin SNMP-agentin kuunteluasetusta, koska se kuunteli aluksi vain localhostissa. 
Asetin sen kuuntelemaan osoitteessa 0.0.0.0:161. 
Lisäksi muutin lukuoikeuksia, jotta myös verkkorajapintojen tiedot olivat saatavilla. 

3: Kerätyt tiedot

Käytin tietojen hakemiseen snmpget- ja snmpwalk-komentoja.
Esimerkki:
snmpget -v2c -c public web1 .1.3.6.1.2.1.1.5.0
Tulos oli web1.
Keräsin järjestelmätiedot kaikilta kolmelta laitteelta:
web1: nimi web1, Linux/WSL2, uptime noin 5 min 46 s.
db1: nimi db1, Linux/WSL2, uptime noin 4 min 19 s.
branch-client: nimi branch-client, Linux/WSL2, uptime noin 3 min 2 s.
Kaikki kolme laitetta vastasivat SNMP-kyselyihin. Uptime-arvot ovat kyselyhetkellä saatuja arvoja.

4: Verkkorajapinnat

Komento:
snmpwalk -v2c -c public web1 .1.3.6.1.2.1.2.2.1.2
Tulosteet:
iso.3.6.1.2.1.2.2.1.2.1 = STRING: "lo"
iso.3.6.1.2.1.2.2.1.2.64 = STRING: "eth0"

Web1-laitteelta löytyi kaksi verkkorajapintaa:
lo: paikallinen loopback-rajapinta.
eth0: kontin verkkorajapinta, jota käytetään verkkoliikenteeseen.
Molempien rajapintojen toimintatila oli 1, eli up.

5: OID-analyysi

sysName.0: kertoo laitteen nimen.
sysDescr.0: kertoo järjestelmän kuvauksen.
sysUpTime.0: kertoo SNMP-agentin hallintaosan toiminta-ajan.
ifDescr: kertoo verkkorajapintojen nimet.
ifOperStatus: kertoo verkkorajapintojen toimintatilan.

6: Pohdinta

Mitä opin tehtävän aikana?
Opin asentamaan SNMP-agentin, tekemään SNMP-kyselyitä ja keräämään tietoja usealta laitteelta.
Opin myös muuttamaan agentin kuuntelu- ja lukuoikeusasetuksia.

Mitkä ovat SNMP tärkeimmät hyödyt?
SNMP avulla laitteiden tietoja voidaan kerätä automaattisesti ja käyttää niitä verkon valvontaan sekä vikojen selvittämiseen.

Mitkä ovat SNMP rajoitukset tai haasteet?
SNMPv2c ei salaa liikennettä, ja yhteisönimi kulkee selväkielisenä. 
Väärät kuuntelu- tai käyttöoikeusasetukset voivat myös estää kyselyt.


