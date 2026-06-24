# RIPv2 i konfigurisanje RIPv2 protokola u Cisco Packet Tracer-u
## Šta je ovo?
Ovo je vežba za administratore računarskih mreža koju sam radio tokom mog školovanja. Nalazi se jedan .pkt fajl koji se preuzme i to će Vam biti početno stanje. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — RIPv2 (Routing Information Protocol version 2)

### 1. Šta je RIPv2?
**RIPv2** je **distance-vector** protokol za dinamičko rutiranje, druga (poboljšana) verzija originalnog RIP protokola, definisana u **RFC 2453**. Koristi **Bellman-Ford algoritam** za izračunavanje najboljeg puta, a kao metriku koristi **broj skokova (hop count)** do odredišne mreže — ruta sa manjim brojem skokova se smatra boljom, bez obzira na propusni opseg ili kašnjenje na linkovima.

---

### 2. Koje su razlike između RIPv1 i RIPv2?
| Karakteristika | RIPv1 | RIPv2 |
|---|---|---|
| Tip rutiranja | Classful (bez maske) | **Classless** (šalje i masku — podržava VLSM/CIDR) |
| Način slanja update-a | Broadcast (255.255.255.255) | **Multicast** (224.0.0.9) |
| Autentifikacija | Ne podržava | **Podržava** (plain text i MD5) |
| Manuelni sažetak ruta | Nije moguć | **Moguć** (manual route summarization) |

---

### 3. Koje su osnovne prednosti RIPv2 protokola?
RIPv2 nam omogućava:
1. **Jednostavnost konfiguracije** — RIP je jedan od najlakših protokola za podešavanje, idealan za male mreže i učenje osnova rutiranja.
2. **Podršku za VLSM i CIDR** — pošto šalje i informaciju o maski mreže, može se koristiti u mrežama sa promenljivom dužinom maske (za razliku od RIPv1).
3. **Autentifikaciju** — mogućnost da se sprovede provera identiteta između rutera pre razmene rutirajućih informacija, što povećava sigurnost.
4. **Manji utrošak resursa rutera** — jednostavniji algoritam (u odnosu na link-state protokole) znači manje opterećenje CPU-a, što je pogodno za stariju ili slabiju opremu.

---

### 4. Koji su nedostaci RIPv2 protokola?
- **Spora konvergencija** — u poređenju sa OSPF-om, RIP sporije reaguje na promene u topologiji.
- **Ograničenje na 15 skokova** — mreže udaljene 16 ili više skokova smatraju se **nedostupnim** (metrika 16 = infinity), što ograničava RIP na manje mreže.
- **Metrika ne uzima u obzir propusni opseg** — RIP može izabrati sporiju putanju (npr. sporu serijsku vezu) ako ima manje skokova, umesto bržu putanju sa više skokova.
- **Periodično slanje kompletne routing tabele** — RIP šalje celu tabelu svakih 30 sekundi (default), što nepotrebno opterećuje propusni opseg u odnosu na link-state protokole koji šalju samo promene.

---

### 5. Koji su RIP tajmeri i čemu služe?
- **Update tajmer (30s)** — interval slanja periodičnih update poruka svim RIP susedima.
- **Invalid (Hold-down povezan) tajmer (180s)** — ako ruta nije ponovo potvrđena (refresh-ovana) u ovom periodu, proglašava se nevažećom (metrika se postavlja na 16/infinity).
- **Hold-down tajmer (180s)** — period u kome ruter ignoriše nove (potencijalno lošije/nestabilne) informacije o nekoj ruti, kako bi se sprečile petlje (routing loops) usled nestabilnosti.
- **Flush tajmer (240s)** — vreme nakon kog se nevažeća ruta **u potpunosti briše** iz routing tabele.

---

### 6. Kako se konfiguriše osnovni RIPv2 na Cisco ruteru?
Osnovna konfiguracija RIP procesa, prebacivanje na verziju 2, i oglašavanje mreža:

"Router(config)# router rip"

"Router(config-router)# version 2"

"Router(config-router)# network 192.168.1.0"

"Router(config-router)# network 192.168.2.0"

*(Komanda `network` se unosi sa **classful** adresom mreže, bez maske — RIP sam prepoznaje koji interfejsi spadaju u te mreže.)*

---

### 7. Kako se isključuje automatska sumarizacija (auto-summary) u RIPv2?
RIPv2 po default-u automatski sumira rute na granicama klasnih mreža, što može uzrokovati probleme u mrežama sa **diskontinualnim (discontiguous)** podmrežama. Isključuje se komandom:

"Router(config)# router rip"

"Router(config-router)# no auto-summary"

---

### 8. Kako se konfiguriše passive interfejs u RIPv2?
Kada interfejs treba da bude **oglašen** u RIP mreži, ali na njemu **ne treba slati RIP update poruke** (npr. interfejs prema krajnjim korisnicima), koristi se:

"Router(config)# router rip"

"Router(config-router)# passive-interface GigabitEthernet0/0"

Ovo smanjuje nepotreban saobraćaj i povećava sigurnost, jer se RIP poruke ne šalju na mrežu gde nema drugih rutera.

---

### 9. Kako se konfiguriše autentifikacija u RIPv2?
RIPv2 podržava autentifikaciju, koja se konfiguriše preko **key chain-a** i primenjuje na interfejsu (plain text ili MD5):

"Router(config)# key chain MOJ_KLJUC"

"Router(config-keychain)# key 1"

"Router(config-keychain-key)# key-string cisco123"

"Router(config)# interface GigabitEthernet0/0"

"Router(config-if)# ip rip authentication mode md5"

"Router(config-if)# ip rip authentication key-chain MOJ_KLJUC"

---

### 10. Kako se podešava propagacija default rute kroz RIP?
Da bi se postojeća default ruta (0.0.0.0/0.0.0.0) na jednom ruteru **prosledila** ostalim RIP ruterima u mreži, koristi se:

"Router(config)# router rip"

"Router(config-router)# default-information originate"

---

### 11. Koje su komande za proveru RIPv2 konfiguracije i stanja?
Najvažnije komande za verifikaciju:

"Router# show ip route rip"

"Router# show ip protocols"

"Router# debug ip rip"

- `show ip route rip` — prikazuje samo rute naučene putem RIP-a u routing tabeli (oznaka **R** u tabeli rutiranja).
- `show ip protocols` — prikazuje detaljne informacije o RIP procesu: verziju, tajmere, oglašene mreže, susede, administrativnu distancu (default AD za RIP = **120**).
- `debug ip rip` — prikazuje RIP update poruke u realnom vremenu (slanje i primanje), korisno za troubleshooting.

---

### 12. Kako RIP sprečava routing petlje (loops)?
RIP koristi nekoliko mehanizama za sprečavanje petlja:
- **Maksimalan broj skokova (15)** — bilo koja ruta sa metrikom 16 smatra se nedostupnom (infinity), čime se ograničava širenje petlje.
- **Split horizon** — ruter ne šalje update o ruti **nazad** kroz interfejs sa kog je tu rutu i naučio.
- **Route poisoning** — kada mreža postane nedostupna, ruter je oglašava sa metrikom **16 (infinity)**, umesto da je jednostavno ukloni iz update-a, čime se brže informišu ostali ruteri.
- **Hold-down tajmeri** — sprečavaju prihvatanje novih (potencijalno lažnih/nestabilnih) informacija o ruti tokom određenog perioda nakon što je ona postala nedostupna.

---
