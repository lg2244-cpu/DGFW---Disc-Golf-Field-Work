# Kastlinje – fra prototype til app

Dette er en frittstående PWA (Progressive Web App) versjon av Kastlinje-prototypen.
Den bruker vanlig nettleser-GPS og vanlig `localStorage`, som fungerer normalt
her siden siden ikke lenger kjører inni en Claude-forhåndsvisning.

## Filer
- `index.html` — hele appen (HTML/CSS/JS i én fil)
- `manifest.json` — gjør appen installerbar på hjemskjerm
- `service-worker.js` — enkel offline-cache av app-skallet
- `icons/` — app-ikoner (192px og 512px)

## 1. Test lokalt
Åpne en terminal i denne mappen og kjør en enkel lokal server (må være HTTPS
eller `localhost` for at GPS skal fungere):

```
python3 -m http.server 8000
```

Åpne `http://localhost:8000` på telefonen (samme wifi-nett som datamaskinen,
bruk datamaskinens lokale IP i stedet for `localhost`), eller bare i
datamaskinens nettleser for å se på design/flyt.

## 2. Publiser med ekte HTTPS (nødvendig for GPS på ekte telefon utenfor localhost)
Enklest gratis-alternativer:

**GitHub Pages**
1. Lag et nytt repo, last opp alle filene i denne mappen til rot av repoet.
2. Settings → Pages → velg branch `main` / mappe `/ (root)`.
3. Du får en URL som `https://dittbrukernavn.github.io/repo-navn/`.

**Netlify (drag-and-drop, ingen konto med Git nødvendig)**
1. Gå til app.netlify.com/drop
2. Dra hele mappen inn i nettleservinduet.
3. Du får en URL med en gang.

## 3. Installer på telefonen
Åpne URL-en fra steg 2 i telefonens nettleser:
- **Android/Chrome:** meny → "Legg til på Hjem-skjerm"
- **iPhone/Safari:** del-knappen → "Legg til på Hjem-skjerm"

Da får du et app-ikon som åpner appen i fullskjerm uten nettleserramme, og
GPS-tilgangen fungerer som i en vanlig app (du blir spurt om tillatelse første
gang).

## 4. Neste steg: ekte nativ app (valgfritt)
Hvis du senere vil ha appen i App Store / Google Play, eller trenger bedre
GPS-tilgang i bakgrunnen:

- **Capacitor** (anbefalt) — pakker denne samme HTML/JS-koden i et nativt
  app-skall. Minimal omskriving. `npm install @capacitor/core @capacitor/cli`,
  se https://capacitorjs.com/docs/getting-started
- Fra da av kan du bruke native plugins for mer nøyaktig GPS, bakgrunnssporing,
  og publisere til app-butikkene.

## Kjente begrensninger å være obs på
- Nettleser-GPS har typisk ±3–8 m nøyaktighet — helt greit for kastelengder,
  men ikke landmåler-presist.
- `localStorage` er per nettleser/enhet. Skal du bruke appen på flere
  telefoner med delt historikk, må data synkroniseres via en backend
  (f.eks. en enkel database) — si gjerne til meg om dette blir aktuelt, det er
  en naturlig neste utbygging.
