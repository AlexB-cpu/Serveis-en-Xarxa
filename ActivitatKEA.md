# AA2 – Anàlisi i Configuració de Kea DHCP4

**Projecte:** Implementació de servei DHCP amb Kea DHCP4  
**Empresa:** TechMataró SL  
**Autor:** Biel Clave  
**Mòdul:** M0227 – Serveis de Xarxa  
**Entorn:** Linux (Ubuntu Server / Debian), Visual Studio Code  

---

## 1. Introducció

Aquest projecte recull la configuració, gestió i verificació del servei **Kea DHCP4** (desenvolupat per ISC) en un entorn corporatiu per a l'empresa **TechMataró SL**. L'objectiu principal és desplegar l'adreçament IP dinàmic, gestionar reserves d'adreces estàtiques mitjançant adreces MAC, i establir procediments de diagnòstic i manteniment del servei.

---

## 2. Bloc 1 – Fonaments del protocol i anàlisi de tràfic

### 2.1 Procés de negociació DHCP (DORA)

El procés de negociació DHCP segueix el flux de 4 passos conegut com a **DORA**:

1. **Discover:** El client envia una petició per localitzar servidors DHCP disponibles a la xarxa.
2. **Offer:** El servidor respon oferint una adreça IP disponible i paràmetres bàsics.
3. **Request:** El client sol·licita formalment l'assignació de l'adreça IP ofeta.
4. **Acknowledge (ACK):** El servidor confirma l'assignació de l'adreça IP i li atorga la cessió (*lease*).

#### Tipus de transmissió (Broadcast / Unicast) en xarxa local sense DHCP Relay:
- **Discover:** `Broadcast` (El client no té IP assignada ni coneix la IP del servidor).
- **Offer:** `Broadcast` (o `Unicast` segons la implementació de la pila TCP/IP de la targeta de xarxa).
- **Request:** `Broadcast` (Comunica a tots els servidors quin ha estat l'oferta acceptada).
- **Acknowledge:** `Broadcast` / `Unicast` (Confirmació final de la concessió).

---

### 2.2 Ports i capçaleres DHCP

DHCP opera sobre el protocol de transport **UDP**:
- **Port UDP del client:** `68`
- **Port UDP del servidor:** `67`

#### Identificació del client
El camp de la capçalera DHCP que identifica unívocament la interfície de xarxa del client és **`chaddr`** (*Client Hardware Address*), que conté l'adreça física MAC.

---

### 2.3 Opcions DHCP (DHCP Options)

| Opció | Número | Descripció |
|---|---|---|
| **Router** | `3` | Gateway o porta d'enllaç per defecte |
| **DNS** | `6` | Llista de servidors de noms de domini (*Domain Name Servers*) |
| **Domain Name** | `15` | Domini DNS local per a la resolució de noms de la xarxa |

---

## 3. Bloc 2 – Configuració del servidor Kea DHCP4

### 3.1 Paràmetres del servei

- **Interfície d'escolta:** `eth0`
- **Tipus de base de dades de cessions:** `memfile` (amb persistència activada)
- **Temps de cessió (valid-lifetime):** 7200 segons (2 hores)
- **Temps de renovació (renew-timer):** 3600 segons (1 hora)
- **Subxarxa:** `192.168.1.0/24`
- **Pool dinàmic:** `192.168.1.50` – `192.168.1.200`
- **Gateway:** `192.168.1.1`
- **DNS Primari:** `1.1.1.1`
- **DNS Secundari:** `8.8.8.8`
- **Domini local:** `oficina.local`

---

### 3.2 Fitxer de configuració complet (`/etc/kea/kea-dhcp4.conf`)

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