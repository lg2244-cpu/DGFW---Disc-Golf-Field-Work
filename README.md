# DG Field Work

Frittstående PWA (Progressive Web App) for feltmåling i disc golf. Registrer diskene dine i en "bag", start en økt (session) med ett fast utgangspunkt og én fast sikteretning, og mål kastelengde + sideavvik per disk over så mange runder du vil — uten å måtte gå GPS-runden på nytt for hver runde. Bruker vanlig nettleser-GPS og `localStorage`, ingen backend.

**Live:** https://lg2244-cpu.github.io/DGFW---Disc-Golf-Field-Work/

## Filer
- `index.html` — hele appen (HTML/CSS/JS i én fil, ingen build-steg)
- `manifest.json` — gjør appen installerbar på hjemskjerm
- `service-worker.js` — offline-cache av app-skallet. **Bump `CACHE_NAME` ved enhver endring i cachede filer**, ellers ser ikke brukere som allerede har besøkt siden noen oppdatering.
- `icons/` — app-ikoner (192px og 512px)
- `TODO.md` — prioritert backlog over planlagt/foreslått arbeid
- `implementert.md` — logg over hva som allerede er bygget og hvorfor

## Funksjonsstatus
Se [implementert.md](implementert.md) for hva som er bygget, og [TODO.md](TODO.md) for hva som står for tur.

## Kjør lokalt
Åpne en terminal i denne mappen og kjør en enkel lokal server (må være HTTPS
eller `localhost` for at GPS skal fungere):

```
python -m http.server 8080
```

(Port 8000 kan være blokkert av Windows på enkelte oppsett — 8080 er tryggere.)

Åpne `http://localhost:8080` på telefonen (samme wifi-nett som datamaskinen,
bruk datamaskinens lokale IP i stedet for `localhost` — merk at Windows-brannmuren
må tillate innkommende tilkoblinger til `python.exe` for at dette skal fungere fra
en annen enhet), eller bare i datamaskinens nettleser for å se på design/flyt.

## Publisering
Live-versjonen kjører på GitHub Pages og oppdateres automatisk ved push til
`master` (kan ta et par minutter). Ønsker du et eget speil:

**GitHub Pages**
1. Lag et nytt repo (må være offentlig for gratis Pages-hosting, med mindre du har GitHub Pro/Team), last opp alle filene i denne mappen til rot av repoet.
2. Settings → Pages → velg branch `master` / mappe `/ (root)`.
3. Du får en URL som `https://dittbrukernavn.github.io/repo-navn/`.

**Netlify (drag-and-drop, ingen konto med Git nødvendig)**
1. Gå til app.netlify.com/drop
2. Dra hele mappen inn i nettleservinduet.
3. Du får en URL med en gang.

## Installer på telefonen
Åpne appens URL i telefonens nettleser:
- **Android/Chrome:** meny → "Legg til på Hjem-skjerm"
- **iPhone/Safari:** del-knappen → "Legg til på Hjem-skjerm"

Da får du et app-ikon som åpner appen i fullskjerm uten nettleserramme, og
GPS-tilgangen fungerer som i en vanlig app (du blir spurt om tillatelse første
gang).

## Kjente begrensninger
- Nettleser-GPS har typisk ±3–8 m rå nøyaktighet. Appen strømmer posisjoner
  (`watchPosition`) og venter med å låse opp registrer-knappen til signalet er
  ≤7 m, med en manuell nødluke hvis signalet aldri blir bedre — men den fysiske
  presisjonstaket til enhetens GPS-brikke kan ikke fjernes helt, bare kompenseres for.
- `localStorage` er per nettleser/enhet — ingen synkronisering mellom telefoner
  i dag, og ingen automatisk sikkerhetskopi. Se "Sikkerhetskopi av alt" i
  [TODO.md](TODO.md) for planlagt løsning (full JSON-eksport/import).
