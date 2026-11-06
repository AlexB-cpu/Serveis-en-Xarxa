# AA2 – Avaluació: Anàlisi Kea DHCP

**Mòdul:** M0227 – Serveis de Xarxa  
**Alumne:** Biel Clave  
**Curs:** SMX2  
**Empresa:** TechMataró SL  
**Data:** 04/11/2026

---

# 1. Introducció

En aquesta activitat es treballarà la instal·lació, configuració i gestió del servei Kea DHCP4 en un sistema Linux. L'objectiu és aprendre a configurar l'adreçament IP dinàmic, crear reserves d'adreces IP i comprovar el funcionament del servidor mitjançant diferents eines de diagnòstic.

L'activitat es desenvolupa en l'escenari d'una empresa anomenada TechMataró SL, que necessita un servidor DHCP per gestionar les adreces IP dels seus dispositius.

---

# 2. Bloc 1 – Fonaments del protocol i anàlisi de tràfic

## 2.1 Procés de negociació DHCP (DORA)

El procés DORA és el procediment que segueix un client per obtenir una adreça IP automàticament.

- **Discover:** el client busca un servidor DHCP disponible.
- **Offer:** el servidor ofereix una adreça IP al client.
- **Request:** el client sol·licita l'adreça IP que vol utilitzar.
- **Acknowledge:** el servidor confirma l'assignació de l'adreça IP.

### Broadcast i Unicast

Indicar quines fases del procés DORA utilitzen Broadcast i quines Unicast en una xarxa local sense DHCP Relay.

**Resposta:**
- **Discover:** Utilitza **Broadcast** (el client no té IP assignada ni sap quina és la IP del servidor).
- **Offer:** Utilitza **Broadcast** (o Unicast depenent de la pila de xarxa, però habitualment Broadcast ja que pot no tenir IP directa).
- **Request:** Utilitza **Broadcast** (perquè tots els servidors sàpiguen quina oferta s'ha acceptat).
- **Acknowledge:** Utilitza **Broadcast** o **Unicast** (confirmació final de la concessió).

---

## 2.2 Ports i capçaleres DHCP

DHCP utilitza el protocol UDP per comunicar el client amb el servidor.

- **Port UDP del client:** `68`
- **Port UDP del servidor:** `67`

### Identificació del client

Indicar quin camp de la capçalera DHCP permet identificar la targeta de xarxa del client.

**Resposta:**
El camp de la capçalera DHCP que identifica la targeta de xarxa del client és el camp **`chaddr`** (*Client Hardware Address*), on s'especifica l'adreça física o MAC.

---

## 2.3 Opcions DHCP

Completar la informació de les opcions DHCP utilitzades per transmetre els paràmetres de xarxa.

| Opció | Número |
|---|---|
| Router | `3` |
| DNS | `6` |
| Domain Name | `15` |

### Evidència pràctica

Afegir una captura de Wireshark o tcpdump on es pugui observar el tràfic DHCP.

**Captura:** *(Inserir captura de Wireshark/tcpdump mostrant paquets DHCP Discover/Offer/Request/ACK)*

---

# 3. Bloc 2 – Configuració del servidor Kea DHCP

## 3.1 Configuració de la interfície i base de dades

El servidor Kea DHCP4 s'ha de configurar perquè escolti les peticions DHCP a través de la interfície de xarxa `eth0`.
També s'ha d'utilitzar el sistema d'emmagatzematge `memfile` per guardar les cessions DHCP, amb persistència activada.

## 3.2 Paràmetres globals i temps de cessió

Configurar els següents paràmetres:

| Paràmetre | Valor |
|---|---|
| valid-lifetime | 7200 segons |
| renew-timer | 3600 segons |

## 3.3 Configuració de la subxarxa i pool

La xarxa de l'empresa utilitza una subxarxa `192.168.1.0/24`.
El servidor ha de permetre l'assignació dinàmica d'adreces IP dins del següent rang:

- **Subxarxa:** `192.168.1.0/24`
- **Pool dinàmic:** `192.168.1.50 – 192.168.1.200`

## 3.4 Opcions de xarxa

Configurar les opcions necessàries perquè els clients rebin correctament els paràmetres de xarxa.

| Paràmetre | Valor |
|---|---|
| Router | `192.168.1.1` |
| DNS primari | `1.1.1.1` |
| DNS secundari | `8.8.8.8` |
| Domini local | `oficina.local` |

## 3.5 Fitxer de configuració kea-dhcp4.conf

Redactar el fitxer JSON complet comú de configuració:

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "eth0" ]
    },
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 7200,
    "renew-timer": 3600,
    "rebind-timer": 5400,
    "option-data": [
      {
        "name": "domain-name-servers",
        "data": "1.1.1.1, 8.8.8.8"
      },
      {
        "name": "domain-name",
        "data": "oficina.local"
      }
    ],
    "subnet4": [
      {
        "id": 1,
        "subnet": "192.168.1.0/24",
        "pools": [
          {
            "pool": "192.168.1.50 - 192.168.1.200"
          }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "192.168.1.1"
          }
        ],
        "reservations": [
          {
            "hw-address": "aa:bb:cc:11:22:33",
            "ip-address": "192.168.1.20",
            "hostname": "impresora-dir"
          }
        ]
      }
    ]
  }
}