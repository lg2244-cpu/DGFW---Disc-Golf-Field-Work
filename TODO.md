# DG Field Work – TODO

## 1. Statistikk å hente ut fra kastene — ferdig ✓
Utover det opprinnelige (lengst/snitt/kortest per disk, kronologisk kastliste), er følgende implementert i disk-detaljvisningen:

- **Konsistens/spredning** — standardavvik i kastelengde per disk (4. stat-boks).
- **Sideveis tendens (fade/turn-bias)** — gjennomsnittlig sideavvik per disk, vist som chip.
- **Utvikling over tid** — trend siste 5 kast vs. tidligere kast (krever ≥6 kast for å vises).
- **Sammenligning mellom disker** — egen "Sammenlign disker"-skjerm, gruppert på type (driver/midrange/putter), rangert på snittavstand.
- **Forhold under kastet** — vind (retning + styrke) settes én gang per runde på utgangspunkt-skjermen; kasttype (hyzer/anhyzer/flatt) velges per kast. Disk-detaljvisningen har filter-chips for begge, slik at statistikken over kan brytes ned på f.eks. "Destroyer i motvind".

Datamodell: `rounds[i].wind = {direction, strength}`, `rounds[i].throws[j].throwType`. Begge valgfrie; gammel lagret data uten disse feltene håndteres som "ikke satt", ikke egen filterkategori.

## 2. Diskregister (bag) — rediger/slett — ferdig ✓
Rediger- og slett-knapp per disk-kort. Sletting er ikke-destruktivt: disken forsvinner fra bag/historikk/sammenlign, men rådata for tidligere kast med disken ligger fortsatt i `localStorage`.

## 3. Historikk — slett runde — ferdig ✓
Ny "Alle runder"-visning (nås fra Historikk-fanen) med slett-knapp + bekreftelse per runde.

## 4. Git + publisering — ferdig ✓
- ✓ Lokalt git-repo initialisert.
- ✓ GitHub-repo opprettet og pushet, gjort offentlig (bruker bekreftet — ingen hemmeligheter i koden): https://github.com/lg2244-cpu/DGFW---Disc-Golf-Field-Work
- ✓ GitHub Pages aktivert og live: **https://lg2244-cpu.github.io/DGFW---Disc-Golf-Field-Work/**. Oppdateres automatisk ved push til `master`.

## 5. Fargedesign for sollys-lesbarhet — ferdig ✓
Hele appen byttet fra mørkt til lyst, høykontrast-tema, siden appen alltid brukes utendørs i felt (aldri innendørs) — samme prinsipp som turgps-er/sportsklokker.
- Bakgrunn/kort: lys kremhvit (`--chalk`/`--chalk-dim`), mørk skoggrønn ink-tekst (`--forest-dark`/`--forest`) som primærfarge for all lesetekst.
- Aksentfargene oransje og gull fikk egne mørkere "ink"-varianter (`--disc-orange-ink`, `--turf-yellow-ink`) til bruk som *tekst* (stat-tall, overskrifter, chips), mens de opprinnelige lyse variantene beholdes til *fyll* (knapper, prikker, progresjon) — nødvendig fordi samme lyse farge ikke har nok kontrast både som bakgrunnsfarge-med-mørk-tekst og som tekstfarge-på-lys-bakgrunn.
- Korridor-/bane-visualiseringen (kast-registrering, oversikt) var først bevisst mørkegrønn som et tematisk "fairway"-panel, men ble senere gjort lys som resten av appen etter tilbakemelding om at den fortsatt var vanskelig å lese i sol — det finnes nå ingen bevisst mørke områder igjen.
- **Viktig funn:** `service-worker.js` cachet gammel `index.html` uten at endringer noensinne ble hentet på nytt, siden `CACHE_NAME` aldri endret seg (service worker-skriptet så uendret ut for nettleseren → ingen oppdatering ble trigget). Fikset ved å bumpe `CACHE_NAME`. **Husk å bumpe `CACHE_NAME` ved hver fremtidig endring i cachede filer**, ellers ser ikke brukere som allerede har besøkt siden noen oppdateringer.

## 6. Navnebytte: "Kastlinje" → "DG Field Work" — ferdig ✓
Appnavnet "Kastlinje" var dårlig, byttet overalt: `<title>`, synlig merkevaretekst i header, `manifest.json` (name/short_name), README, denne filen. `localStorage`-nøklene ble også byttet fra `kastlinje:*` til `dgfw:*` — en engangs migrering (`migrateOldStorageKeys()`) kopierer over evt. data lagret under det gamle navnet, slik at ingen mister lagrede disker/runder. `service-worker.js` sitt `CACHE_NAME` byttet til `dgfw-v1`.

## 7. Diskkategorier: 4 nye typer — ferdig ✓
Byttet ut de 3 gamle kategoriene ("Driver"/"Midrange"/"Putter") med 4 nye: **Distance Driver**, **Fairway Driver**, **Mid-range**, **Putt and Approach**. Endret i type-select i legg til/rediger-disk-modalen, standardverdi for nye disker, seed-diskene, og gruppe-rekkefølgen i "Sammenlign disker". Lagt til `migrateDiscTypes()` (kjører i `initApp()`) som mapper gamle lagrede disker over til nye kategorier én gang (`Driver→Distance Driver`, `Midrange→Mid-range`, `Putter→Putt and Approach`) og persisterer, slik at ingen disker forsvinner fra Sammenlign-grupperingen.

## 8. Bedre GPS-nøyaktighet: watchPosition + nøyaktighets-sperre — ferdig ✓
Byttet fra `getCurrentPosition` (tar det aller første, ofte upresise fiksen) til `watchPosition` på alle tre målepunkter (utgangspunkt, sikteretning, kast). Ny delt mekanisme `startPositionWatch()`/`stopPositionWatch()`:
- Strømmer posisjonsoppdateringer og viser live nøyaktighet i statusteksten og et fargekodet nøyaktighetsmerke (gult mens den søker, grønt når den er god).
- Hovedknappen ("Sett utgangspunkt"/"Registrer sikteretning"/"Registrer posisjon") er deaktivert helt til nøyaktigheten er ≤ `ACCURACY_GOOD_M` (7 m).
- En "Bruk nåværende posisjon likevel"-knapp dukker opp så snart det finnes ett fiks (uansett kvalitet), som en manuell nødluke slik at man aldri kan bli sittende fast i tett skog der signalet aldri blir bra nok.
- GPS-pin-ikonet pulserer mens det søkes, roer seg når signalet er godt.
- `stopPositionWatch()` kalles ved bekreftet måling, ved "← Velg annen disk", ved "Hopp over"-sikteretning, og som sikkerhetsnett øverst i `showTab()` — ingen watch skal fortsette å kjøre i bakgrunnen og tappe batteri etter at brukeren har forlatt måleskjermen.
- `getCurrentPosition`-baserte `getPosition()`-helperen er fjernet (ubrukt etter omleggingen). `describeGeoError`s timeout-tekst oppdatert til 15 sekunder (matcher nytt `timeout`-alternativ i `watchPosition`).
