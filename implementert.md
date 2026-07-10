# DG Field Work – Implementert og ferdigstilt

## 1. Statistikk å hente ut fra kastene
Utover det opprinnelige (lengst/snitt/kortest per disk, kronologisk kastliste), er følgende implementert i disk-detaljvisningen:
- **Konsistens/spredning** — standardavvik i kastelengde per disk (4. stat-boks).
- **Sideveis tendens (fade/turn-bias)** — gjennomsnittlig sideavvik per disk, vist som chip.
- **Utvikling over tid** — trend siste 5 kast vs. tidligere kast (krever ≥6 kast for å vises).
- **Sammenligning mellom disker** — egen "Sammenlign disker"-skjerm, gruppert på type (driver/midrange/putter), rangert på snittavstand.
- **Forhold under kastet** — vind (retning + styrke) settes én gang per runde på utgangspunkt-skjermen; kasttype (hyzer/anhyzer/flatt) velges per kast. Disk-detaljvisningen har filter-chips for begge, slik at statistikken over kan brytes ned på f.eks. "Destroyer i motvind".

*Datamodell:* `rounds[i].wind = {direction, strength}`, `rounds[i].throws[j].throwType`. Begge valgfrie; gammel lagret data uten disse feltene håndteres som "ikke satt", ikke egen filterkategori.

## 2. Diskregister (bag) — rediger/slett
Rediger- og slett-knapp per disk-kort. Sletting er ikke-destruktivt: disken forsvinner fra bag/historikk/sammenlign, men rådata for tidligere kast med disken ligger fortsatt i `localStorage`.

## 3. Historikk — slett runde
Ny "Alle runder"-visning (nås fra Historikk-fanen) med slett-knapp + bekreftelse per runde.

## 4. Git + publisering
- Lokalt git-repo initialisert.
- GitHub-repo opprettet og pushet, gjort offentlig (bruker bekreftet — ingen hemmeligheter i koden): https://github.com/lg2244-cpu/DGFW---Disc-Golf-Field-Work
- GitHub Pages aktivert og live: **https://lg2244-cpu.github.io/DGFW---Disc-Golf-Field-Work/**. Oppdateres automatisk ved push til `master`.

## 5. Fargedesign for sollys-lesbarhet
Hele appen byttet fra mørkt til lyst, høykontrast-tema, siden appen alltid brukes utendørs i felt (aldri innendørs) — samme prinsipp som turgps-er/sportsklokker.
- Bakgrunn/kort: lys kremhvit (`--chalk`/`--chalk-dim`), mørk skoggrønn ink-tekst (`--forest-dark`/`--forest`) som primærfarge for all lesetekst.
- Aksentfargene oransje og gull fikk egne mørkere "ink"-varianter (`--disc-orange-ink`, `--turf-yellow-ink`) til bruk som *tekst* (stat-tall, overskrifter, chips), mens de opprinnelige lyse variantene beholdes til *fyll* (knapper, prikker, progresjon) — nødvendig fordi samme lyse farge ikke har nok kontrast både som bakgrunnsfarge-med-mørk-tekst og som tekstfarge-på-lys-bakgrunn.
- Korridor-/bane-visualiseringen (kast-registrering, oversikt) var først bevisst mørkegrønn som et tematisk "fairway"-panel, men ble senere gjort lys som resten av appen etter tilbakemelding om at den fortsatt var vanskelig å lese i sol — det finnes nå ingen bevisst mørke områder igjen.
- **Viktig funn:** `service-worker.js` cachet gammel `index.html` uten at endringer noensinne ble hentet på nytt, siden `CACHE_NAME` aldri endret seg (service worker-skriptet så uendret ut for nettleseren → ingen oppdatering ble trigget). Fikset ved å bumpe `CACHE_NAME`. **Husk å bumpe `CACHE_NAME` ved hver fremtidig endring i cachede filer**, ellers ser ikke brukere som allerede har besøkt siden noen oppdateringer.

## 6. Navnebytte: "Kastlinje" → "DG Field Work"
Appnavnet "Kastlinje" var dårlig, byttet overalt: `<title>`, synlig merkevaretekst i header, `manifest.json` (name/short_name), README, denne filen. `localStorage`-nøklene ble også byttet fra `kastlinje:*` til `dgfw:*` — en engangs migrering (`migrateOldStorageKeys()`) kopierer over evt. data lagret under det gamle navnet, slik at ingen mister lagrede disker/runder. `service-worker.js` sitt `CACHE_NAME` byttet til `dgfw-v1`.

## 7. Diskkategorier: 4 nye typer
Byttet ut de 3 gamle kategoriene ("Driver"/"Midrange"/"Putter") med 4 nye: **Distance Driver**, **Fairway Driver**, **Mid-range**, **Putt and Approach**. Endret i type-select i legg til/rediger-disk-modalen, standardverdi for nye disker, seed-diskene, og gruppe-rekkefølgen i "Sammenlign disker". Lagt til `migrateDiscTypes()` (kjører i `initApp()`) som mapper gamle lagrede disker over til nye kategorier én gang (`Driver→Distance Driver`, `Midrange→Mid-range`, `Putter→Putt and Approach`) og persisterer, slik at ingen disker forsvinner fra Sammenlign-grupperingen.

## 8. Bedre GPS-nøyaktighet: watchPosition + nøyaktighets-sperre
Byttet fra `getCurrentPosition` (tar det aller første, ofte upresise fiksen) til `watchPosition` på alle tre målepunkter (utgangspunkt, sikteretning, kast). Ny delt mekanisme `startPositionWatch()`/`stopPositionWatch()`:
- Strømmer posisjonsoppdateringer og viser live nøyaktighet i statusteksten og et fargekodet nøyaktighetsmerke (gult mens den søker, grønt når den er god).
- Hovedknappen ("Sett utgangspunkt"/"Registrer sikteretning"/"Registrer posisjon") er deaktivert helt til nøyaktigheten er ≤ `ACCURACY_GOOD_M` (7 m).
- En "Bruk nåværende posisjon likevel"-knapp dukker opp så snart det finnes ett fiks (uansett kvalitet), som en manuell nødluke slik at man aldri kan bli sittende fast i tett skog der signalet aldri blir bra nok.
- GPS-pin-ikonet pulserer mens det søkes, roer seg når signalet er godt.
- `stopPositionWatch()` kalles ved bekreftet måling, ved "← Velg annen disk", ved "Hopp over"-sikteretning, og som sikkerhetsnett øverst i `showTab()` — ingen watch skal fortsette å kjøre i bakgrunnen og tappe batteri etter at brukeren har forlatt måleskjermen.
- `getCurrentPosition`-baserte `getPosition()`-helperen er fjernet (ubrukt etter omleggingen). `describeGeoError`s timeout-tekst oppdatert til 15 sekunder (matcher nytt `timeout`-alternativ i `watchPosition`).

## 9. Vindstille som eget vindalternativ
Lagt til «Vindstille» som fjerde valg i vindretning-chippene (ved siden av Medvind/Motvind/Sidevind). Når det velges skjules styrke-raden helt og `windStrength` tvinges til `null`, siden styrke ikke gir mening ved vindstille. Gjenbrukt samme liste (`WIND_DIRECTIONS`) i filter-chippene på disk-detaljvisningen, så man også kan filtrere kast på «vindstille» der.

## 10. Flight-tall på disker (Speed/Glide/Turn/Fade)
Fire valgfrie numeriske felt i legg til/rediger-disk-modalen (halve poeng støttet, f.eks. Turn -1.5, som er vanlig på ekte disk-spesifikasjoner): Speed (1–15), Glide (1–7), Turn (-5–1), Fade (0–5). Vises som en kompakt linje under disk-typen i Bag-listen (f.eks. «12 | 5 | -1.5 | 3») kun når minst ett av tallene er satt. Ikke obligatorisk — eksisterende disker uten disse verdiene fungerer som før.

## 11. Import/eksport av bag som .txt
To knapper øverst i Bag-fanen. Eksport laster ned en `.txt`-fil (kommaseparert, én disk per linje: `navn,type,speed,glide,turn,fade,farge`, med en `#`-kommentarlinje øverst som forklarer formatet) via Blob + midlertidig nedlastingslenke. Import leser en valgt fil via FileReader og «upserter»: disker matches på navn (case-insensitivt) — finnes en disk med samme navn fra før oppdateres den, ellers legges en ny disk til. Ingen duplikater. Ukjente/manglende diskType-strenger normaliseres mot de fire kjente kategoriene (case-insensitivt), rader uten navn/type hoppes over. Viser oppsummering («X nye, Y oppdatert, Z hoppet over») etter import.

## 12. Session-nivå over Ny runde
Nytt lag mellom Bag og runde-registrering: en **session** låser utgangspunkt *og* sikteretning på tvers av så mange runder man vil, siden det vanligste er å teste samme disker gjentatte ganger fra samme fysiske sted.
- Trykker man på «Ny runde»-fanen uten aktiv session, startes automatisk et nytt session-oppsett (samme GPS-skjermer som før, nå med overskriftene «Ny session · Utgangspunkt»/«Ny session · Sikteretning»). Er en session allerede aktiv, hopper man rett til rundevelgeren.
- Rundevelgeren (`screen-round`) har fått en «Rediger utgangspunkt/sikteretning»-knapp (kaller `editSessionPosition()`) og en «Avslutt session»-knapp (`endSession()`). Redigering midt i en runde med allerede registrerte kast varsler først og nullstiller de kastene (siden de er beregnet fra det gamle utgangspunktet og blir ugyldige).
- **Vind flyttet fra sesjon- til rundenivå** — chippene sitter nå på rundevelgeren i stedet for utgangspunkt-skjermen, siden forholdene kan endre seg over en lengre økt selv om posisjonen ikke gjør det.
- Diskvalget (`selectedForRound`) beholdes bevisst mellom runder i samme session (nullstilles kun ved helt ny session) — vanligvis vil man teste samme disker om igjen. `resetRound()` (Ny runde-knappen på Oversikt) nullstiller derfor bare kast/vind, ikke diskvalget eller selve sesjonen.
- Kun in-memory, som resten av GPS-tilstanden — en session overlever ikke at appen lukkes/refreshes.

## 13. UI-finpuss fra brukertesting
- Diskrader i rundevelgeren er nå klikkbare i hele bredden (var kun avhukingsboksen før).
- «Registrer posisjon (GPS)»-knappen i kastregistrering flyttet opp rett under disknavn/type, foran bane-visualiseringen, så den er synlig uten scrolling.
- Import/eksport av disker byttet fra komma- til semikolon-skilletegn i `.txt`-formatet, siden komma også er vanlig desimaltegn på norsk (f.eks. glide 4,5) — import tåler nå komma som desimaltegn.

## 14. Sticky «fortsett»-knapper
Alle knapper man må trykke for å komme videre (Start runde, Neste på begge session-skjermene, Fortsett etter et kast, «alle disker registrert»-knappen) er klebende (`position: sticky`) nederst på skjermen, med bakgrunnsfarge som matcher skjermen. Løser samme problem som GPS-knapp-flyttingen i pkt. 13, men generelt for alle skjermer uavhengig av innholdsmengde, i stedet for å måtte flikke skjerm for skjerm.

## 15. Fiks av tre lagrings-/krasjbugs
Funnet ved full manuell gjennomgang av appen:
- Kast registrert etter første besøk på Oversikt-fanen ble aldri lagret (`roundSaved` var en engangs-lås). Byttet til `savedRoundId` som oppdaterer samme lagrede runde ved senere besøk.
- En ferdig runde gikk tapt hvis man startet neste runde eller avsluttet sesjonen uten noen gang å ha vært innom Oversikt. `startRoundInSession()`, `endSession()` og `resetRound()` lagrer nå eventuelle ventende kast før de nullstilles.
- Sletting av en disk som hadde kast i pågående runde krasjet Oversikt/kastvelgeren (oppslag på slettet disks navn/farge). `deleteDisc()` rydder nå også disken ut av `results`/`throwQueue`/`currentDiscId`.

## 16. Sikkerhetskopi av alt (eksporter/importer disker+runder)
To knapper i Historikk-fanen: «Sikkerhetskopi →» laster ned én JSON-fil med alle disker og alle runder (`exportAllData()`). «Gjenopprett →» leser en valgt fil (`importAllDataFromText()`) og **erstatter** alt i appen med innholdet — ikke en sammenslåing som disk-import i pkt. 11, siden dette er ment som gjenoppretting fra et øyeblikksbilde, ikke å legge til flere disker. Varsler med antall disker/runder før og etter og krever bekreftelse siden det ikke kan angres. Validerer at filen er gyldig JSON med `discs`/`rounds`-arrays før noe skjer, kjører `migrateDiscTypes()` på det gjenopprettede innholdet i tilfelle backupen er fra før 4-kategori-omleggingen, og beregner `nextId` på nytt.

## 17. Median, merking av usikre målinger, og utvidet trendvisning i disk-detalj
- **Median** — femte stat-boks ved siden av Kast/Lengst/Snitt/Spredning, mer robust mot GPS-uteliggere enn gjennomsnitt.
- **Nøyaktighet lagret per kast** — `confirmThrow()` lagrer nå `coords.accuracy` på hvert kast, ført gjennom `maybeSaveRound()` og `throwsForDisc()`. Kast med nøyaktighet dårligere enn `ACCURACY_GOOD_M` (7 m) merkes med «±Xm usikker» i kastlisten i disk-detalj. Ny filter-chip «Skjul upresise kast (>7 m)» (`detailHideImprecise`) utelater dem fra både statistikk og liste når aktivert.
- **Personlig rekord** — det lengste kastet i (filtrert) utvalg merkes med «PR» i kastlisten.
- **Beste måned** — grupperer kastene per kalendermåned og viser måneden med høyest snittavstand som egen chip, kun når data spenner over minst to måneder.
- **Trendgraf** — ny `buildTrendLineSVG()`-funksjon tegner en linje over tid (avstand per kast, kronologisk) i en egen boks over stolpediagrammet, som supplement til det (ikke erstatning). Vises kun ved ≥2 kast.
- Gjenstår i TODO.md: eget snitt for siste 10 kast, og sammenligning mot en brukervalgt tidligere periode (krever ny periodevelger-UI).

## 18. Innstillinger-fane, med dev-mode for Historikk
Ny femte hovedfane «Innstillinger» (`screen-settings`) i bunn-navigasjonen.
- **Samlet eksport/import** — disk-eksport/import (.txt) flyttet hit fra Bag, JSON-sikkerhetskopi/gjenopprett flyttet hit fra Historikk. Begge fortsetter å operere på ekte `discs`/`rounds` uansett dev-mode-status — dette er databehandling, ikke en visningsinnstilling.
- **Dev-mode** — en chip-toggle som viser Historikk-fanen med kunstig genererte disker og runder (`generateDevData()`) i stedet for ekte historikk, til bruk for å se hvordan skjermen ser ut med mye data. Bevisst **isolert i et helt eget lagringsrom** (`dgfw-dev:discs`/`dgfw-dev:rounds`, egen `devMode`-flagg under `dgfw:devMode`) — ekte data røres aldri, uansett hvor lenge dev-mode står på.
- **Kun Historikk påvirkes** — Bag, Ny runde og Oversikt bruker alltid ekte data uavhengig av dev-mode; kun de fem Historikk-relaterte render-funksjonene (`renderHistoryList`, `throwsForDisc`, `renderDiscDetail`, `renderAllRounds`, `renderCompareDiscs`) og `deleteRound()` sjekker `devMode` via de nye hjelperne `activeDiscs()`/`activeRounds()`.
- **Tydelig merket** — et gult «🧪 Viser testdata»-bånd vises øverst på alle fire Historikk-skjermene (disk-liste, disk-detalj, alle runder, sammenlign) når dev-mode er aktivt, slik at ekte og falsk historikk aldri kan forveksles.
- Dev-mode forblir på til det skrus manuelt av (ingen automatisk avstenging) — akseptabelt siden det uansett aldri kan påvirke registrering av ekte kast.
- Genererer 8 falske disker (fordelt på de fire kategoriene) og 30 falske runder spredt over ~4 måneder, med tilfeldig vind/kasttype/nøyaktighet (inkl. en del "usikre" målinger for å teste pkt. 17 sitt filter også). «Generer nye testdata →»-knapp lar deg lage et nytt tilfeldig datasett når som helst mens dev-mode er på.

## 19. Historikk → Statistikk, og akser/forklarende tekst på alle grafer
Fanen «Historikk» (og skjermtitlene «Historikk · Per disk»/«Historikk · Alle runder») hetet fra nå av **«Statistikk»** — kun synlig tekst endret, interne id-er/funksjonsnavn (`screen-history`, `renderHistoryList()`, `data-tab="history"` osv.) er bevisst uendret for å unngå unødvendig risiko ved en ren tekstendring.

Alle tre grafene i appen hadde tidligere ingen synlige akser eller forklaring — bare en visuell form brukeren måtte tolke selv:
- **Korridor-SVG** (`buildCorridorSVG()`, brukt i kastregistrering og Oversikt) — fikk avstandsskala øverst (maks-verdi), «V»/«H»-merking på sidene, og «Utgangspunkt»-tekst ved startpunkt-sirkelen. En forklarende bildetekst under selve visualiseringen ("Loddrett = kastelengde ..., vannrett = sideavvik ...") ble lagt til begge steder den brukes.
- **Stolpediagrammet** (kastelengde per registrering i disk-detalj) — fikk en Y-akse (maks-verdi øverst, "0 m" nederst, ny `.chart-yaxis`) og en X-akse (eldste/nyeste dato under, ny `.chart-xaxis`), pluss en overskrift som sier hva grafen faktisk viser.
- **Trendgrafen** (`buildTrendLineSVG()`) — fikk samme type Y-akse (maks/min-verdi) og X-akse (eldste/nyeste dato) tegnet direkte inn i SVG-en, med en grunnlinje for referanse.

Resultatlistene (Oversikt, Sammenlign disker) rørt ikke — disse viser allerede eksplisitte tall/enheter direkte i teksten, ikke bare en visuell stolpe, og ble vurdert som allerede utvetydige.

## 20. Kastkart per disk (alle runder) og Bag-rapportkort

To nye visninger i Statistikk-fanen, valgt ut fra en mockup-runde med brukeren (5-6 alternative statistikk-konsepter ble skissert som Claude-artifacts, ikke i selve appen — disse to ble valgt for reell implementasjon):

- **Kastkart i disk-detalj** (`detail-kastkart`) — samler *alle* (filtrerte) kast for disken i ett spredningsplott, på tvers av alle runder/sesjoner, i stedet for kun én runde av gangen. Gjenbruker `buildCorridorSVG()` uendret: `lateral`/`along` er allerede lagret meter-relativt til hver runde sin egen siktelinje, så punkter fra ulike fysiske utgangspunkt kan trygt overlegges i samme koordinatsystem. Vises mellom stat-boksene og trendgrafen, med kasteantall i bildeteksten.
- **Bag-rapportkort** (`screen-bagreport`, nås via egen knapp øverst i Statistikk) — hele bagen rangert etter snittavstand i én tabell: rangering, disknavn/type, snitt, spredning (standardavvik), og en liten sparkline (`buildSparklineSVG()`) over de siste 8 kastenes lengde. To badges fremhever ytterpunkter: «Lengst» (høyest snitt) og «Mest konsistent» (lavest spredning — krever minst 3 kast for å kvalifisere, ellers ville en disk med 1-2 kast vinne på falskt nær-null-avvik). Disker uten kast havner nederst i lista uten tall. Respekterer dev-mode-isolasjonen fra pkt. 18 fullt ut (eget testdata-bånd, bruker `activeDiscs()`/`throwsForDisc()`).

## 21. Minimerte filtergrupper i disk-detalj

De tre filterne i disk-detalj (vindretning, kasttype, presisjon) tok tidligere alltid opp fast plass som tre alltid-synlige chip-rader over statistikken. Erstattet med native `<details>`/`<summary>`-elementer (`.filter-group`), kollapset som standard — ingen egen JS trengs for åpne/lukke-oppførselen. Et lite `.filter-group-value`-merke i `<summary>` (satt av `renderDetailFilters()`) viser gjeldende valg (f.eks. «Motvind») selv når gruppen er lukket, slik at et aktivt filter aldri er usynlig.

**Fallgruve funnet under verifisering:** appens egen `.chip-select-row{display:flex}`-regel overstyrte nettleserens innebygde skjuling av lukket `<details>`-innhold, siden en vanlig (ikke-`!important`) author-regel alltid vinner over UA-stilarket uansett selektor-spesifisitet. Løst med en eksplisitt `.filter-group:not([open]) .chip-select-row{display:none;}`-regel.

## 22. Simulert GPS i dev-mode, for å teste «Ny runde» uten ekte posisjon

`startPositionWatch()` bruker nå simulerte posisjoner i stedet for `navigator.geolocation.watchPosition` når `devMode` er på — hele flyten utgangspunkt → sikteretning → kast kan dermed testes fra kontorpulten, uten GPS-maskinvare eller HTTPS/felt-tilgang.

- `simOrigin`/`simAimBearing` (nye globale variabler) settes én gang per økt i `beginNewSession()`, slik at simulert utgangspunkt og sikteretning henger sammen på tvers av skjermene akkurat som ekte GPS ville gjort.
- `simulatedTargetCoords(simMode)` regner ut et fiktivt mål-punkt ut fra hvilken skjerm som spør (`simMode: 'startpoint'|'aim'|'throw'`, satt i `cfg` på hvert av de fire kallene til `startPositionWatch()`): utgangspunktet er et fast tilfeldig punkt, sikteretningen er 10 m unna i en tilfeldig kompassretning, og hvert kast simulerer en realistisk kastelengde (60–115 m) med sidespredning (±10 m) langs nettopp den retningen — ellers ville avstand/sideavvik alltid blitt 0.
- `startSimulatedPositionWatch()` etterligner selve nøyaktighets-konvergensen til ekte `watchPosition` (starter upresist, blir gradvis bedre inntil ≤ `ACCURACY_GOOD_M`), slik at nøyaktighetssperren i UI-et (jf. CLAUDE.md) fortsatt oppleves riktig i dev-mode — statusteksten er tydelig merket «🧪 Simulert posisjon (dev-mode)» slik at simulerte og ekte fiks aldri kan forveksles.
- Ekte GPS (`navigator.geolocation`) røres aldri når dev-mode er på — ren erstatning, ingen risiko for at simulerte koordinater lekker inn i en ekte måling eller omvendt.

## 23. Versjonsnummer og "sist oppdatert" nederst i Innstillinger

To nye konstanter øverst i `<script>`-blokken, `APP_VERSION` og `APP_UPDATED_AT` (ISO-tidspunkt med tidssone), vises nederst i Innstillinger som «Versjon 17 · Sist oppdatert 10.07.2026 kl. 18:45» (`formatAppUpdatedAt()`, satt i `initApp()`). Gir brukeren en enkel måte å se om telefonen faktisk har hentet siste versjon fra service worker-cachen.

Siden appen ikke har noe build-steg, er disse to konstantene rent manuelle — CLAUDE.md er oppdatert med at de skal bumpes/settes sammen med `CACHE_NAME` ved hver fremtidig endring, samme rutine som cache-bumpen i punkt 5.

## 24. Varsel om ny versjon ("🔄 Ny versjon er klar")

Punkt 5 og 23 avdekket samme friksjon fra to kanter: cache-først service worker betyr at en bruker som allerede har appen åpen/installert, ikke ser en ny versjon før et *andre* besøk etter deploy — helt stille, uten noe varsel om at det finnes noe å oppdatere til. Løst med en liten banner i headeren («🔄 Ny versjon er klar», med en «Oppdater»-knapp som kaller `location.reload()`), skjult som standard.

- Service worker-registreringen (helt nederst i `<script>`-blokken) lytter nå på `updatefound` på `ServiceWorkerRegistration`-objektet, og videre på `statechange` for den nye installerende workeren. Banneret vises kun når state blir `'installed'` **og** `navigator.serviceWorker.controller` allerede finnes — det skillet er avgjørende: uten det ville banneret feilaktig vist seg ved aller første installasjon også (der det jo ikke finnes noen "gammel versjon" å oppdatere fra).
- Siden `service-worker.js` allerede kaller `self.skipWaiting()` ubetinget i `install`, aktiveres og overtar den nye workeren kontrollen (`clients.claim()`) i praksis rett etter at banneret vises — et vanlig `location.reload()` er derfor nok, ingen `postMessage`-håndtering trengs for å be workeren hoppe over en «waiting»-fase.
- Verifisert ende-til-ende i preview ved å simulere en ekte oppgradering (registrere en "gammel" versjon, endre `CACHE_NAME`, reloade uten å tømme cache) — banneret dukket opp korrekt, og forsvant igjen etter at "Oppdater" ble trykket og siden hadde hentet den nye versjonen.

## 25. Snitt siste 10 kast + sammenligning mot alle tidligere kast

Utvidet disk-detaljens chip-rad med to nye chips (vises når en disk har >10 kast, ved siden av den eksisterende "siste 5 kast"-trenden): rått snitt for de siste 10 kastene, og en sammenligning av det snittet mot *alle* kast før de siste 10 (i stedet for én valgt tidligere periode — det ville krevd en egen datovelger-UI som ikke finnes i appen). Samme visuelle mønster (pil + farge) som den eksisterende korttids-trenden, bare med et lengre og mer stabilt tidsvindu.

## 26. Kartvisning av siktepunkt/landingspunkt i kastregistrering

Ny veksleknapp ("Diagram"/"Kart") ved siden av korridor-SVG-en i kastregistrering, rett etter at et kast er bekreftet. "Kart" viser et ekte Leaflet-kart (OpenStreetMap-fliser) med to markører — gult utgangspunkt (siktepunkt) og oransje landingspunkt — og en stiplet linje mellom dem, zoomet til å vise begge (`fitBounds`).

- **Arkitekturunntak**: Leaflet lastes fra CDN (`unpkg.com`, med SRI-hash) i `<head>` — det eneste stedet appen bruker et eksternt bibliotek i selve koden (jf. CLAUDE.md). Kartflisene krever nettverkstilgang og fungerer ikke offline, i motsetning til resten av appen.
- **Datamodell-utvidelse**: appen lagret tidligere *ingen* faktiske GPS-koordinater — kun lokale meter (avstand/sideavvik) relativt til utgangspunktet. Hvert kast får nå også `lat`/`lon` (satt i `confirmThrow()`), og hver lagret runde får `startpoint` (`{lat, lon}`, satt i `maybeSaveRound()`). Eldre runder mangler disse feltene — helt ufarlig, siden ingen eksisterende kode leser dem.
- **Bevisst avgrenset**: kartet kan derfor kun vises for kastet som *akkurat* er registrert (live GPS-fiks tilgjengelig i `lastThrowCoords`), ikke for historiske kast i Oversikt/Statistikk — de manglet lat/lon frem til nå. Fremtidige kast får dataen lagret, så en historisk kartvisning kan bygges senere uten datamigrering.
- Verifisert i preview med simulert GPS (dev-mode, se punkt 22): bekreftet at kartfliser faktisk hentes (4 nettverkskall til `tile.openstreetmap.org`, alle 200 OK), at begge markører + linje tegnes riktig, og at `startpoint`/`lat`/`lon` faktisk persisteres korrekt på en lagret runde.

## 27. Kartvisning i Oversikt også, og kart som standardvisning

Utvidet punkt 26 til alle stedene appen viser en siktelinje (bruker ba om «alle steder», og at kart skal være default fremfor diagrammet):

- **Delt kart-funksjon**: kartlogikken i punkt 26 var skrevet spesifikt for ett kast — brutt ut til en gjenbrukbar `renderPointsMap(mapWrapId, innerId, origin, points, emptyMsg)` som tar imot en liste med punkter i stedet for ett enkelt. Både kastregistrering og Oversikt kaller nå denne samme funksjonen.
- **Oversikt** (`screen-summary`) fikk samme "Diagram"/"Kart"-veksleknapp som kastregistrering, med ett landingspunkt per disk i runden — alle med felles utgangspunkt siden de deler samme økt. Fargekodet per disk, akkurat som korridor-SVG-en gjør i dag.
- **Kart er nå standard** begge steder (`throwVizMode`/`summaryVizMode` starter på `'map'`) — diagrammet er fortsatt tilgjengelig ved trykk på "Diagram", men brukeren må aktivt velge det bort nå, ikke motsatt.
- **Bevisst holdt utenfor**: Kastkart i disk-detalj (aggregerer kast på tvers av mange ulike økter/utgangspunkt) fikk *ikke* kartvisning — avklart med bruker, siden den visningen normaliserer bort den fysiske plasseringen med vilje for å kunne sammenligne kast fra ulike steder, og har ingen felles siktelinje å tegne på et ekte kart.
- Verifisert i preview med simulert GPS: bekreftet kart vises som standard med riktig antall markører/linjer i begge skjermer (1 kast → 3 elementer, 2 kast i Oversikt → 5 elementer), at kartflisen faktisk lastet (`naturalWidth: 256`, `complete: true`), og at veksling til diagram og tilbake fungerer.

## 28. Samle-spredningskart øverst i Statistikk-fanen

Inspirert av en annen disc golf-app ([Discloggen](https://krissern97.github.io/discloggen/), sett på etter brukerens forespørsel — se mockup-runden i samtalen), som leder statistikksiden sin med ett spredningskart for *alle* kast i stedet for en diskliste først.

- Ny seksjon øverst i Statistikk-fanen (`renderOverviewSpread()`), rett under dev-banneret og over diskliste/knapperaden: alle kast fra alle disker plottet i samme korridor-SVG, farget per disk, med en legend under (siden farge alene ikke skiller 6 disker fra hverandre — jf. dataviz-prinsipper).
- Egne filtre (vindretning/kasttype/presisjon), samme kollapsbare `.filter-group`-mønster som disk-detalj, men med egne state-variabler (`overviewWindFilter` osv.) — helt separat fra disk-detaljens filter, ingen kryssforurensning.
- Ny `allThrowsAcrossDiscs()` — som `throwsForDisc()`, men uten discId-filter.
- **Bevisst holdt utenfor**: disklisten og drill-ned til per-disk-detalj er uendret, kun lagt til over — avklart med bruker for å unngå en større omlegging av navigasjonen.
- Verifisert i preview: dev-mode-isolasjon fungerer (ekte/falske disker vises riktig avhengig av `devMode`), filtrering fungerer (96→16 kast ved hyzer-filter i testdata), tomt-filter-tilstand håndteres uten feil (tom legend/statrad, korridor uten prikker), ingen konsollfeil.

## 29. Presisjonsmodus — egen måleflyt for putting/nærspill

Andre halvdel av mockup-runden inspirert av [Discloggen](https://krissern97.github.io/discloggen/), som har en egen "presisjonsøkt" ved siden av lengdemåling: du velger et mål og logger hvor nære hvert kast lander, i stedet for avstand langs en siktelinje. Brukeren ba eksplisitt om hele måleflyten, ikke bare en stat-widget på eksisterende data.

- **Ny inngang i "Ny runde"-fanen**: tapper du fanen uten en aktiv økt av noe slag, møtes du nå av et modusvalg (`screen-mode-choice`) — «Start lengdeøkt» (uendret flyt) eller «🎯 Start presisjonsøkt» (ny). `showTab('round')` ruter nå til riktig skjerm avhengig av `sessionActive`/`precisionSessionActive`.
- **Egen, enklere måleflyt**: presisjon bruker kun ÉN disk per økt (typisk en putter, valgt i `screen-precision-disc`), ikke en diskkø som lengdemodus — matcher hvordan puttetrening faktisk fungerer (mange kast på rad med samme disk). Deretter ett enkelt GPS-punkt for målet (`screen-precision-target`, ingen sikteretning trengs siden bom-avstand er retningsuavhengig), og en løkke for å logge kast (`screen-precision-throw` → «Nytt kast →» → gjenta).
- **Helt separat datamodell**: `precisionRounds` (persistert via `Store`, egen `dgfw:precisionRounds`-nøkkel) med `{id, date, discId, target:{lat,lon}, throws:[{missDistance,x,y,lat,lon,accuracy}]}` — ingen gjenbruk av `rounds`/`results`/`distance`/`lateral`, for å unngå enhver risiko for kryssforurensning med lengdemodus sine tall. `x`/`y` (lokale meter fra målet, via `toLocalXY()`) lagres sammen med `missDistance` slik at en radial mål-visning kan tegnes senere.
- **`buildTargetSVG()`** — ny visualisering ved siden av `buildCorridorSVG()`: konsentriske ringer rundt et sentrert mål i stedet for en rett siktelinje, siden presisjon er retningsuavhengig. Punktene plottes med faktisk retning+avstand fra målet, ikke bare avstanden som et tall.
- **Presisjonsstatistikk** (`screen-precisionstats`, nås via ny «Presisjon →»-knapp i Statistikk-fanen): snitt-bom, beste kast, andel innenfor 3 m (fast terskel, `PRECISION_WITHIN_M`), filtrert per disk. Respekterer dev-mode-isolasjonen fullt ut (egen `devPrecisionRounds`, `activePrecisionRounds()`-hjelper, eget testdata-bånd) — `generateDevData()` utvidet til å lage 8 falske presisjonsrunder med en av de falske putter-diskene.
- Simulert GPS (dev-mode, punkt 22) utvidet med en ny `simMode: 'precision-throw'` — genererer kast tett rundt målet (mest under 3 m, sjeldne bommer lenger unna) i stedet for lengdemodusens 60-115 m, siden de to måleflytene har helt ulik geometri.
- Verifisert i preview: hele flyten (modusvalg → velg disk → sett mål → logg 3 kast → avslutt) med simulert GPS, riktig lagret datastruktur (`missDistance`/`x`/`y`/`lat`/`lon` per kast, `target` per runde), dev-mode-isolasjon bekreftet begge veier (tomt med ekte testøkt mens dev var på, riktige tall etter `generateDevData()`, riktige ekte tall med dev av igjen — snitt/beste/andel stemte eksakt med de 3 loggede kastene), disk-filter i statistikken fungerer, og lengdemodus sin uendrede flyt (`beginNewSession()` fra det nye modusvalget) fortsatt fungerer som før. Ingen konsollfeil.

## 30. Fullført «Skaler UI til skjermhøyden» — inkludert kart/diagram-visningen

En tidligere økt (commit `99ed49d`) la allerede inn et `@media (max-height)`-basert skaleringssystem (CSS-variabler som `--header-pad`, `--screen-pad`, `--pin-size` osv., krympet i to trinn ved 700px/560px), men TODO-punktet ble aldri fjernet. Testet empirisk i preview (måler `scrollHeight` vs. `clientHeight` for hver skjerm) i stedet for å anta at punktet allerede var dekket eller fortsatt var ugjort.

**Funn:** hovedknappen på hver skjerm er allerede alltid synlig/trykkbar (`.sticky-footer{position:sticky; bottom:0;}`) — det reelle problemet var at kart/korridor-visningen (lagt til i en senere økt, punkt 26-28) bruker faste høyder (`220px`/`180px`) som aldri ble koblet til skaleringssystemet. På en liggende telefon (667×375) ga dette 93px overflow i kastregistrering; selv på en vanlig høy telefon (390×844, over 700px-grensen og dermed uskalert) var det 173px overflow siden kart/veksleknapp-innholdet aldri fantes da skaleringssystemet ble bygget.

- Ny delt `--viz-h`-variabel styrer både korridor-SVG-en (`.corridor-wrap svg{height:var(--viz-h);}`, overstyrer SVG-ens `height`-presentasjonsattributt) og Leaflet-kartet (`.throw-map-inner`) — samme mønster som de andre skjermhøyde-variablene.
- Nytt, mildere `@media (max-height: 900px)`-trinn (`--viz-h`, `--throwcard-pad`, `--distance-fs` litt strammere) for vanlige moderne telefoner som er for høye til å trigge 700px-trinnet, men likevel trengte litt hjelp etter at kart-innholdet kom til.
- Nytt, strengere `@media (max-height: 420px)`-trinn for ekte liggende-telefon-høyder, som de to opprinnelige trinnene (700/560px) ikke dekket godt nok.
- Kortet ned kastregistreringens diagram-bildetekst («Loddrett = kastelengde, vannrett = sideavvik (V/H)», droppet «fra utgangspunktet»/«fra siktelinjen») for å matche den already-kortere varianten brukt i samle-spredningskartet, og strammet marginen over kasttype-velgeren — begge rene tekst/mellomrom-kutt, ingen funksjonalitet fjernet.
- **Bevisst IKKE «fikset»**: Alle runder, disk-detalj og Statistikk-forsiden scroller fortsatt mye på korte skjermer — det er naturlig innhold (lister som vokser med mengden data) der scroll er forventet og riktig, ikke et resultat av dårlig skjermhøyde-tilpasning. Å tvinge disse til å vise alt uten scroll ville krevd en helt annen løsning (virtualiserte lister e.l.), ikke CSS-variabel-skalering — avklart med bruker før arbeidet startet.
- Verifisert i preview på flere høyder: kastregistrering gikk fra 93px→0px overflow (667×375) og 173px→25-43px overflow (390×844, resten krever å fjerne mer innhold enn det er verdt for en skjerm som uansett er godt over normal telefon-høyde). Sammenlign disker gikk fra 13px→2px. Bekreftet at en ekte, urealistisk ekstrem høyde (375×320, kortere enn nesten alle ekte telefoner) fortsatt bare scroller (ikke knekker) siden `.screen{overflow-y:auto}` og sticky-footeren uansett holder knappen nåbar. Ingen konsollfeil.