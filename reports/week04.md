
1: Johdanto

Infrastructure as Code (IaC) tarkoittaa, että infrastruktuuria hallitaan koodin ja asetustiedostojen avulla. 
Palveluiden asennuksia ei tarvitse tehdä käsin jokaisella koneella, vaan ne voidaan suorittaa automaattisesti.
Ansible on työkalu, jolla hallitaan useita koneita. 
Inventory määrittää hallittavat laitteet ja niiden ryhmät. 
Playbook sisältää tehtävät, jotka Ansible suorittaa kohdekoneilla.
Automaation avulla asennukset ovat nopeampia ja yhdenmukaisempia. 
Samat tehtävät voidaan suorittaa uudelleen ilman, että jokaista komentoa tarvitsee kirjoittaa käsin.

2: Inventory

Inventory sijaitsee Ansible-kontissa tiedostossa `/ansible/inventory.ini`.

| Ryhmä | Laitteet |
|---|---|
| routers | r1, r2, r3 |
| clients | client1, attacker, branch-client |
| servers | web1, db1 |
| monitoring | Prometheus, Grafana, Zabbix, cAdvisor |
| management | Ansible |
| ubuntu_hosts | client1, web1, db1, branch-client |
| linux_hosts | clients, servers, monitoring, management |
| node_exporter | routers, clients, servers |

Lisäksi inventory sisältää ryhmät `user_network`, `server_network` ja `branch_office`.
Ryhmien avulla sama playbook voidaan suorittaa usealle koneelle kerralla. Esimerkiksi `ubuntu_hosts`-ryhmää käytettiin SNMP:n ja Node Exporterin asentamiseen neljälle Ubuntu-koneelle.
Inventoryn yhteystesteissä Linux-koneiden ja reitittimien yhteydet toimivat. Prometheus-, Grafana-, Zabbix-, cAdvisor- ja Ansible-konttien SSH-yhteydet eivät toimineet, koska portti 22 hylkäsi yhteyden. 
Konttien käynnissäolo ei siis yksin tarkoita, että niihin voi yhdistää SSH:lla.

3: Ensimmäinen playbook

Ensimmäinen playbook oli `/ansible/playbooks/ping.yml`.
Playbook testaa Linux-koneiden yhteyksiä Ansiblella. Se yrittää myös suorittaa reitittimillä `show version` -komennon.
Linux-koneiden ping-testit onnistuivat tavoitettavissa olevilla koneilla. Reitittimien komentotehtävässä ilmeni ongelmia, jotka liittyivät Paramiko-kirjastoon ja SSH-yhteyteen. 
Erillisessä inventory-testissä reitittimien Ansible-ping kuitenkin onnistui.
Tuloksista opin, että yhteystesti voi onnistua yhdellä Ansible-moduulilla, vaikka toinen yhteystapa tai komentotehtävä ei toimisi.

4: SNMP Playbook

Tiedosto: `/ansible/playbooks/install-snmp.yml`

Lähdekoodi

yaml
---
- name: Install and configure SNMP
  hosts: ubuntu_hosts
  become: true

  vars:
    snmp_community: public

  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install SNMP packages
      apt:
        name:
          - snmp
          - snmpd
        state: present

    - name: Configure SNMP community
      copy:
        dest: /etc/snmp/snmpd.conf
        content: |
          rocommunity {{ snmp_community }}

          sysLocation HAMK Virtual Lab
          sysContact Administrator

          agentaddress udp:161

      notify: restart snmpd

    - name: Enable SNMP service
      service:
        name: snmpd
        enabled: yes
        state: started

    - name: Verify SNMP process
      shell: pgrep snmpd
      register: snmpd_process
      changed_when: false

    - name: Display verification
      debug:
        msg: "SNMP agent running on {{ inventory_hostname }}"

  handlers:
    - name: restart snmpd
      service:
        name: snmpd
        state: restarted


Käytetyt moduulit ja muuttujat

- `apt` päivittää pakettilistan ja asentaa SNMP-paketit.
- `copy` luo SNMP-asetustiedoston.
- `service` käynnistää palvelun ja määrittää sen asetuksia.
- `shell` tarkistaa, onko SNMP-prosessi käynnissä.
- `debug` näyttää vahvistusviestin.
- `snmp_community`-muuttuja määrittää SNMP-yhteisön arvoksi `public`.

`notify` käynnistää handlerin, jos asetustiedosto muuttuu. Handler `restart snmpd` käynnistää SNMP-palvelun uudelleen.

Suoritus ja varmistaminen

Playbook suoritettiin neljällä Ubuntu-koneella. Pakettien asennus ja asetustiedostojen luonti onnistuivat, mutta palvelunhallintatehtävä palautti virheen `Service is in unknown state`.
Tarkistin palveluiden tilat erikseen ja käynnistin SNMP-palvelun client1-koneella. Tämän jälkeen kaikki neljä konetta ilmoittivat SNMP-palvelun olevan käynnissä.
Playbookin suoritus ei siis onnistunut täysin ilman virheitä, mutta palveluiden tila tarkistettiin ja korjattiin erikseen.

5: Node Exporter Playbook

Tiedosto: `/ansible/playbooks/install-node-exporter.yml`

Lähdekoodi

yaml
---
- name: Install Prometheus Node Exporter
  hosts: ubuntu_hosts
  become: true

  vars:
    node_exporter_version: "1.9.1"

  tasks:
    - name: Install prerequisite packages
      apt:
        name:
          - wget
          - tar
        state: present
        update_cache: yes

    - name: Create installation directory
      file:
        path: /opt/node_exporter
        state: directory
        mode: "0755"

    - name: Download Node Exporter
      get_url:
        url: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
        dest: /tmp/node_exporter.tar.gz

    - name: Extract archive
      unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /tmp
        remote_src: yes

    - name: Install exporter binary
      copy:
        src: "/tmp/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter"
        dest: /opt/node_exporter/node_exporter
        mode: "0755"
        remote_src: yes

    - name: Start node_exporter
      shell: |
        pkill node_exporter || true
        nohup /opt/node_exporter/node_exporter > /tmp/node_exporter.log 2>&1 &
      args:
        executable: /bin/bash

    - name: Verify exporter responds
      uri:
        url: http://localhost:9100/metrics
        status_code: 200
      register: result

    - name: Show verification result
      debug:
        msg: "Node Exporter running on {{ inventory_hostname }}"


Käytetyt moduulit ja muuttujat

- `apt` asentaa tarvittavat paketit.
- `file` luo asennushakemiston.
- `get_url` lataa Node Exporterin.
- `unarchive` purkaa ladatun arkiston.
- `copy` kopioi ohjelman asennushakemistoon.
- `shell` käynnistää ohjelman taustalla.
- `uri` tarkistaa mittausrajapinnan.
- `debug` näyttää vahvistusviestin.

Muuttuja `node_exporter_version` määrittää asennettavan version. Playbookissa ei ole handleria, koska ohjelma käynnistetään suoraan omassa tehtävässään ja sen toiminta tarkistetaan seuraavassa tehtävässä.

Suoritus ja varmistaminen

Playbook suoritettiin kaikilla neljällä Ubuntu-koneella. Jokaisen koneen `http://localhost:9100/metrics`-rajapinta palautti HTTP 200 -vastauksen.

| Laite | Tulos | Virheet |
|---|---|---|
| client1 | Onnistui | 0 |
| web1 | Onnistui | 0 |
| db1 | Onnistui | 0 |
| branch-client | Onnistui | 0 |


6: Oma playbook: Nginx

Oma playbook tallennettiin tiedostoon `configs/ansible/install-webserver.yml`. Se asentaa Nginxin web1-koneelle, luo etusivun ja tarkistaa, että palvelin vastaa HTTP-pyyntöön.

Playbook käyttää `apt`-moduulia asennukseen, `copy`-moduulia etusivun luomiseen, `command`- ja `shell`-moduuleja palvelun käyttöönottoon sekä `uri`- ja `assert`-moduuleja toiminnan tarkistamiseen. Muuttuja `webserver_name` määrittää sivulla näytettävän palvelimen nimen.

Lähdekoodi

yaml
---
- name: Install and configure web server
  hosts: web1
  become: true
  gather_facts: true

  vars:
    webserver_name: web1

  tasks:
    - name: Update package list and install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Create index page
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html>
          <head><title>{{ webserver_name }}</title></head>
          <body><h1>Welcome to {{ webserver_name }}</h1></body>
          </html>
        mode: "0644"

    - name: Enable nginx at startup
      ansible.builtin.command: update-rc.d nginx defaults
      changed_when: false

    - name: Start nginx if it is not running
      ansible.builtin.shell: |
        if ! service nginx status >/dev/null 2>&1; then
          service nginx start
        fi
      args:
        executable: /bin/bash
      register: nginx_start
      changed_when: "'Starting nginx' in nginx_start.stdout"

    - name: Verify web page
      ansible.builtin.uri:
        url: http://localhost
        return_content: true
        status_code: 200
      register: web_response

    - name: Confirm server name appears on page
      ansible.builtin.assert:
        that:
          - "'Welcome to web1' in web_response.content"
        success_msg: "Nginx is responding and the web page identifies web1."


Suoritus ja tulokset

Playbook suoritettiin komennolla:

bash
ansible-playbook -i /ansible/inventory.ini /ansible/playbooks/install-webserver.yml


Suoritus onnistui tuloksella `ok=6`, `changed=1`, `unreachable=0` ja `failed=0`. 
HTTP-tarkistus palautti tilakoodin 200, ja `assert` vahvisti, että etusivulla näkyy `Welcome to web1`.

Ensimmäisellä yrityksellä Nginxin palvelunhallinta epäonnistui konttiympäristössä. 
Käynnistin palvelun erikseen, muutin playbookia ja varmistin sen toiminnan uudella suorituksella.

7: Järjestelmätietojen kerääminen

Järjestelmätiedot kerättiin Ansiblella `setup`-moduulin avulla. Alla ovat tavoitettavissa olevien Linux-koneiden tulokset.

| Laite | Käyttöjärjestelmä | IPv4-osoite | Prosessorit (vCPU) | Muisti (MB) |
|---|---|---|---:|---:|
| client1 | Ubuntu | 172.20.20.8 | 12 | 15878 |
| web1 | Ubuntu | 172.20.20.6 | 12 | 15878 |
| db1 | Ubuntu | 172.20.20.9 | 12 | 15878 |
| branch-client | Ubuntu | 172.20.20.10 | 12 | 15878 |
| attacker | Kali | 172.20.20.2 | 12 | 15878 |
| r1 | Ubuntu (Ansible-fakta) | Ei saatu | 12 | 15878 |
| r2 | Ubuntu (Ansible-fakta) | Ei saatu | 12 | 15878 |
| r3 | Ubuntu (Ansible-fakta) | Ei saatu | 12 | 15878 |

Reitittimien käyttöjärjestelmäksi Ansible ilmoitti Ubuntu, mutta niiden `ansible_default_ipv4`-fakta oli tyhjä. Siksi niiden IP-osoitteita ei voitu lisätä taulukkoon näiden tulosten perusteella.
Prometheus-, Grafana-, Zabbix-, cAdvisor- ja Ansible-konttien järjestelmätietoja ei saatu SSH-yhteyden puuttumisen vuoksi.
Ansible palautti useille koneille samat prosessori- ja muistiarvot. Konttiympäristössä nämä arvot eivät välttämättä kuvaa kunkin kontin omia resurssirajoja.

8: Vertailu ja analyysi

Käsin asennettaessa komennot pitää suorittaa erikseen jokaisella koneella. Tämä vie aikaa ja lisää virheiden mahdollisuutta. 
Myös asetukset voivat erota koneiden välillä, jos kaikkia vaiheita ei tehdä samalla tavalla.
Ansiblella tehtävät määritellään playbookissa ja kohdekoneet valitaan inventoryn avulla. 
Sama playbook voidaan suorittaa usealla koneella, ja tuloksista nähdään, mitkä tehtävät onnistuivat tai epäonnistuivat.
Automaation tärkeimmät hyödyt ovat ajansäästö, yhdenmukaisuus ja toistettavuus. Automaatio on erityisen tärkeää, kun hallitaan useita palvelimia tai sama asennus pitää tehdä usein.
Automaatio ei kuitenkaan poista tarkistusten tarvetta. Tässä harjoituksessa SNMP:n ja Nginxin palvelunhallinta aiheutti virheitä, vaikka ohjelmat saatiin toimimaan erikseen tehtyjen tarkistusten avulla.

9: Yhteenveto

Opin käyttämään Ansible-inventorya, suorittamaan playbookeja ja keräämään järjestelmätietoja useilta koneilta. Opin myös, miten moduulit, muuttujat ja handlerit toimivat.
Node Exporter asennettiin onnistuneesti neljälle Ubuntu-koneelle. Oma Nginx-playbook asensi web-palvelimen web1-koneelle, loi etusivun ja varmisti HTTP-vastauksen. 
SNMP-palvelut saatiin käyntiin, vaikka playbookin palvelunhallintatehtävässä ilmeni virhe.
Infrastructure as Code helpottaa nykyaikaista järjestelmähallintaa, koska asetukset ja asennukset voidaan määritellä koodina, toistaa ja dokumentoida. 
Samalla on tärkeää tarkistaa, että automaatio toimii todellisessa ympäristössä.
