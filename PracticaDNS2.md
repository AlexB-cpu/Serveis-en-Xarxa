# Pràctica: consultes DNS i mDNS

**Mòdul:** MP 0227 – Serveis de xarxa · 2n SMX · Escola Pia Santa Anna – Mataró
**Tema:** Consultes DNS (autoritatives i no autoritatives) i mDNS
**Alumne:** Alex · **Data:** 23/11/2026

---

## Introducció

En aquesta pràctica s'utilitzen les eines `nslookup` i `dig` des del terminal de la màquina virtual per consultar registres DNS (A, NS, MX, SOA), distingir respostes autoritatives de no autoritatives i comprovar la resolució de noms local amb mDNS entre dues màquines virtuals.

> **Nota:** totes les comandes ja estan llestes per executar directament. Només has de substituir els textos entre claudàtors `[...]` de les respostes per les dades reals que et retorni el teu terminal.

---

## 1. IP de www.xtec.cat (resposta no autoritativa)

**Propòsit:** consultar la IP de la pàgina i comprovar que la resposta és no autoritativa.

**Comanda a executar:**
```bash
nslookup www.xtec.cat
```
![alt text](<img/Captura de pantalla 2026-11-24 170817.png>)

**Observació:** l'avís `Non-authoritative answer` indica que la resposta ve de la memòria cau del resolutor local i no directament del servidor que gestiona el domini.

---

## 2. Servidors de noms de upf.edu amb el DNS de Google (8.8.8.8)

**Propòsit:** obtenir els registres NS del domini `upf.edu` preguntant al servidor 8.8.8.8.

**Comanda a executar:**
```bash
dig @8.8.8.8 NS upf.edu
```

![alt text](<img/Captura de pantalla 2026-11-24 170846.png>)

**Observació:** la resposta no porta el flag `aa`, per tant no és autoritativa. Un dels servidors de noms obtinguts és `ns2.upf.edu`, que s'utilitza als apartats 3 i 5.

---

## 3. Consulta directa a un servidor de noms de la UPF (resposta autoritativa)

**Propòsit:** preguntar a `ns2.upf.edu` (obtingut a l'apartat 2) i comprovar que la resposta és autoritativa.

**Comanda a executar:**
```bash
dig @ns2.upf.edu www.upf.edu
```

![alt text](<img/Captura de pantalla 2026-11-24 171218.png>)

**Observació:** ara apareix el flag `aa` (*authoritative answer*), de manera que la resposta ve del servidor responsable del domini `upf.edu`.

---

## 4. Dominis de correu de upf.edu (registres MX)

**Propòsit:** consultar els servidors de correu del domini.

**Comanda a executar:**
```bash
dig MX upf.edu +short
```



**Observació:** els registres MX indiquen quins servidors reben el correu del domini. El valor numèric és la prioritat: s'utilitza primer el més baix.

---

## 5. Consulta d'escolapia.cat al servidor de noms de la UPF

**Propòsit:** comprovar que el servidor de la UPF (`ns2.upf.edu`) no resol un domini que no gestiona.

**Comanda a executar:**
```bash
dig @ns2.upf.edu escolapia.cat
```

![alt text](<img/Captura de pantalla 2026-11-24 171255.png>)

**Observació:** el servidor respon `REFUSED` perquè només és autoritatiu per als seus dominis i no accepta consultes recursives per a dominis externs.

---

## 6. Registre SOA d'escolapia.cat

**Propòsit:** mostrar el registre d'inici d'autoritat (*Start of Authority*) del domini.

**Comanda a executar:**
```bash
dig SOA escolapia.cat
```
![alt text](<img/Captura de pantalla 2026-11-24 171312.png>)


**Observació:** el SOA identifica el servidor de noms primari, el correu de l'administrador i els temps (serial, refresh, retry, expire i TTL negatiu) que controlen la sincronització de la zona.

---

## 7. Consulta mDNS entre dues màquines virtuals

**Propòsit:** resoldre el nom d'una màquina de la xarxa local sense servidor DNS, usant mDNS (domini `.local`).

**Màquina 1 (`alex-VirtualBox`) – Comandes a executar:**
```bash
sudo apt update && sudo apt install -y avahi-daemon avahi-utils
hostname
```

**Sortida a la Màquina 1:**
```console
alex@alex-VirtualBox:~$ hostname
alex-VirtualBox
alex@alex-VirtualBox:~$
```

**Màquina 2 – Comandes a executar (si la Màquina 1 es diu `alex-VirtualBox`):**
```bash
avahi-resolve -n alex-VirtualBox.local
ping -c 3 alex-VirtualBox.local
```

**Sortida esperada a la Màquina 2 (omple la IP i els mil·lisegons reals):**
```console
alex@alex-VirtualBox2:~$ avahi-resolve -n alex-VirtualBox.local
alex-VirtualBox.local    [IP_màquina_1]
alex@alex-VirtualBox2:~$ ping -c 3 alex-VirtualBox.local
PING alex-VirtualBox.local ([IP_màquina_1]) 56(84) bytes of data.
64 bytes from alex-VirtualBox.local ([IP_màquina_1]): icmp_seq=1 ttl=64 time=[ms] ms
alex@alex-VirtualBox2:~$
```

**Observació:** mDNS permet que les màquines de la mateixa xarxa local es resolguin pel nom `.local` mitjançant multidifusió, sense cap servidor DNS. Les dues màquines han d'estar a la mateixa xarxa (xarxa interna o adaptador pont).

---

## Conclusions finals

- Una resposta **no autoritativa** ve de la memòria cau d'un resolutor; una **autoritativa** (flag `aa`) ve del servidor que gestiona el domini.
- Els registres **NS**, **MX** i **SOA** descriuen qui gestiona el domini, on es rep el correu i com es manté la zona.
- Un servidor autoritatiu no resol dominis aliens (`REFUSED`).
- **mDNS** permet resoldre noms en una xarxa local sense servidor DNS.