# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Hva dette er

En frittstående vanilla JS Progressive Web App for å måle kastelengde og sideavvik i disc golf i felt, via nettleser-GPS. Ingen backend, ingen build-steg, ingen rammeverk. UI-tekst, kodekommentarer og dokumentasjon (README.md, TODO.md, implementert.md) er på norsk — hold ny tekst konsistent med det, inkludert i denne filen.

## Kommandoer

Det finnes ingen build-, lint- eller testverktøy (ingen package.json). For å kjøre og verifisere endringer:

```
python -m http.server 8080
```

Åpne `http://localhost:8080` (må være `localhost` eller HTTPS — Geolocation-API-et nekter over vanlig HTTP på LAN). Foretrekk 8080 fremfor 8000 (8000 kan være blokkert på Windows). For å teste GPS/PWA-oppførsel på telefon: åpne datamaskinens lokale IP fra telefonen på samme wifi-nett (Windows-brannmuren må tillate innkommende tilkoblinger til `python.exe`).

Det finnes ingen automatiserte tester — verifiser endringer ved å teste flyten i en nettleser.

## Git

Kjør alltid `git status` (og `git fetch && git log HEAD..origin/master --oneline` for å se om remote har commits du ikke har lokalt) **før** du begynner å kode. Brukeren kjører av og til flere Claude Code-økter parallelt mot dette repoet uten å huske det selv — det har allerede ført til en reell merge-konflikt (2026-07-10, to økter endret `index.html`/`TODO.md`/`service-worker.js` samtidig). Oppdager du at remote har endringer du ikke har, `git pull`/merge før du fortsetter i stedet for å bygge videre på et utdatert grunnlag.

## Arkitektur

Alt ligger i `index.html`: CSS i én `<style>`-blokk, all markup, deretter all JS i én `<script>`-blokk nederst. Ingen moduler, ingen imports, ingen build-artefakter å holde synkronisert — rediger filen direkte.

- **Navigasjon**: `showTab(name)` bytter mellom de fire fanene i bunn-navigasjonen (bag/round/summary/history) ved å toggle en `.screen.active`-klasse. `showScreen(id)` viser en spesifikk skjerm direkte, brukt i den lineære GPS-målingsflyten (utgangspunkt → sikteretning → kast) og andre visninger utenfor fanene (disk-detalj, sammenlign disker, alle runder). Enhver ny skjerm i GPS-flyten bør kalle `stopPositionWatch()` når man navigerer bort fra den — `showTab()` gjør allerede dette som et sikkerhetsnett, men et nytt `showScreen`-kall bør også gjøre det, ellers kan en `watchPosition` fortsette å kjøre og tappe batteri etter at brukeren har forlatt skjermen.
- **State**: enkle globale `let`-variabler øverst i scriptet (`discs`, `rounds`, `sessionActive`, `startpoint`, `aimVector`, `selectedForRound`, `throwQueue`, `results`, osv.). Ingen reaktivitet — etter at state endres må den aktuelle `render*`-funksjonen (`renderBag`, `renderSummary`, `renderHistoryList`, `renderDiscDetail`, ...) kalles eksplisitt.
- **Domenemodell**: en *session* (`sessionActive` + `startpoint` + `aimVector`) låser ett fast GPS-utgangspunkt og én fast sikteretning på tvers av så mange *runder* brukeren vil ha — det vanlige er å teste samme disker gjentatte ganger fra samme sted. Hver runde registrerer `throws` for diskene som er valgt inn i den. Kastelengde/sideavvik beregnes ved å projisere GPS-fiks til lokale øst/nord-meter med `toLocalXY()`, relativt til session-utgangspunktet.
- **GPS-nøyaktighetssperre**: `startPositionWatch()` / `stopPositionWatch()` pakker inn `navigator.geolocation.watchPosition`. Hovedknappen på en måleskjerm er deaktivert helt til nøyaktigheten er ≤ `ACCURACY_GOOD_M` (7 m); en «bruk nåværende posisjon likevel»-knapp er alltid tilgjengelig som manuell nødluke så snart det finnes ett fiks, slik at brukeren aldri sitter fast hvis GPS-signalet aldri blir bedre (f.eks. i tett skog).
- **Persistering**: `Store`-objektet pakker inn `localStorage` under et `dgfw:`-prefiks — gå via det i stedet for å kalle `localStorage` direkte. `initApp()` kjører engangs-migreringer (`migrateOldStorageKeys()` for det gamle `kastlinje:`-prefikset, `migrateDiscTypes()` for de gamle 3 diskkategoriene) som må fortsette å fungere for brukere med gammel lagret data — ikke fjern disse uten videre.
- **Tema**: bevisst lyst, høykontrast-tema (CSS-variabler som `--chalk`, `--forest`) for lesbarhet i sollys — appen brukes alltid utendørs. Ikke innfør mørke paneler/tema.

## PWA-cache

`service-worker.js` cache-først-serverer app-skallet (`ASSETS`-lista) under `CACHE_NAME`. **Bump `CACHE_NAME` ved enhver endring i en cachet fil** (`index.html`, `manifest.json`, ikoner) — ellers fortsetter nettlesere som allerede har installert/besøkt appen å servere den gamle cachede versjonen på ubestemt tid, siden et uendret service-worker-script aldri trigger en oppdateringssjekk. Dette har forårsaket en reell bug tidligere (se `implementert.md` punkt 5).

Samtidig med denne bumpen: oppdater også `APP_VERSION` og `APP_UPDATED_AT` (øverst i `<script>`-blokken i `index.html`) til samme tall og pushe-tidspunktet (lokal tid, ISO med tidssone). Disse vises nederst i Innstillinger-fanen som «Versjon N · Sist oppdatert dd.mm.åååå kl. tt:mm», så brukeren kan se om telefonen har hentet siste versjon.

## Repo-dokumenter som må holdes oppdatert

- `TODO.md` — prioritert backlog. Hvert punkt følger `- **Tittel**` på egen linje med beskrivelsen indentert på neste linje, blanklinje mellom punkter. Følg dette formatet ved nye punkter.
- `implementert.md` — nummerert endringslogg over hva som er bygget og hvorfor; legg til et punkt etter at en ikke-triviell funksjon er landet.
- `README.md` — brukervendt oversikt, lokal kjøring og publiseringsinstruksjoner.
