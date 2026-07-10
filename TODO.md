# DG Field Work – TODO

Samlet backlog, organisert etter prioritet. Ferdigstilte punkter ligger i [implementert.md](implementert.md).

## Prioritet 1 – Analyse og visualisering

- **Utvikling over tid (utvidet)**
  Dagens trend-badge (siste 5 kast vs. tidligere) kan bygges videre ut med: gjennomsnitt siste 10 kast, sammenligning mot en valgt tidligere periode, personlig rekord per disk, beste måned, egen trendgraf (linje over tid) i tillegg til dagens stolpediagram.

- **Median i tillegg til snitt**
  Median er mer robust mot GPS-uteliggere enn gjennomsnitt, og ville gitt et mer troverdig "typisk kast"-tall per disk i disk-detaljvisningen.

- **Merk usikre målinger**
  Bruker man "Bruk nåværende posisjon likevel" ved dårlig GPS-nøyaktighet, lagres kastet i dag uten spor av at målingen var usikker. Lagre nøyaktigheten (`coords.accuracy`) per kast, vis et lite merke i historikken/detaljvisningen, og vurder mulighet til å filtrere bort upresise kast fra statistikken.

- **Slett/korriger enkeltkast**
  I dag kan man kun slette en hel runde, ikke ett enkelt feilmålt kast. Én dårlig GPS-måling forurenser statistikken permanent inntil hele runden slettes.

- **Kastkart**
  Vis alle kast for en disk ovenfra i ett samlet kart (utvidelse av dagens korridor-SVG, som i dag kun viser én runde av gangen), for å se spredning, høyre/venstre-tendens og konsistens visuelt på tvers av alle runder. Kan bygges som SVG, samme mønster som `buildCorridorSVG()`.

- **Kartvisning i bakgrunnen (siktepunkt vs. landingspunkt)**
  Vis telefonens native kart (Apple Maps for iOS / Android SDK for Android, for å unngå lisenskostnader og forenkle oppsett) bak målepunktene, sentrert og zoomet (bounding box) etter kastets GPS-koordinater. Plasser to unike markører — siktepunkt og landingspunkt — og tegn en linje (polyline) mellom dem for å visualisere avviket.

- **Automatisk analyse av disker**
  La appen selv beskrive hvordan en disk oppfører seg, f.eks. "mest stabil i motvind", "størst spredning", "minst sideavvik", "lengst i medvind", "mest konsistent". Bygger videre på eksisterende vind/kasttype-filtrering i disk-detalj.

- **Sammenligning mellom disker (videreutvikling)**
  Utvid dagens "Sammenlign disker"-skjerm (som i dag kun viser snittavstand) med maks, standardavvik/konsistensscore, og et forslag til anbefalt bruksområde per disk.

- **Personlige anbefalinger**
  Basert på historiske kast, f.eks. "Explorer er i snitt 6 meter lengre enn Teebird i motvind" eller "Ballista er den mest konsistente distance-driveren". Forutsetter at punktene over (spredning, vind-nedbrutt statistikk) er på plass først — dette er i praksis et sammendrag av dem. Målet er at appen blir en treningsassistent, ikke bare et måleverktøy.

## Prioritet 2 – Registrering

- **Flere kastparametere**
  Del opp dagens "kasttype" (hyzer/anhyzer/flatt) i to uavhengige valg: kastestil (backhand/forehand/roller/overhand) og frislipp (hyzer/flat/anhyzer). Mer presist, men også flere valg å ta per kast — vurder om begge fortsatt skal være valgfrie som i dag.

- **Automatisk vær**
  Hent vindretning/-styrke og temperatur automatisk fra GPS-posisjon (værtjeneste-API) i stedet for manuell chip-registrering, med mulighet for brukeren til å korrigere. Krever ekstern API-nøkkel/tjeneste (f.eks. met.no sitt gratis API siden appen allerede er norsk) — egen vurdering av hvilken tjeneste før implementasjon.

- **Baner og hull**
  Knytt en session til bane/hull/teepad i stedet for bare fri GPS-posisjon, for bedre sammenligning over tid på samme hull. Overlapper med dagens session-konsept (utgangspunkt+sikteretning låst per økt) — dette ville i så fall bli en navngitt/gjenbrukbar variant av samme idé, ikke noe helt nytt system.

## Prioritet 3 – Brukervennlighet

- **Skaler UI til skjermhøyden**
  Tilpass layout/skriftstørrelse/avstander dynamisk til tilgjengelig skjermhøyde (f.eks. med `vh`/`dvh`-enheter eller `ResizeObserver`), slik at alle knapper og elementer på en skjerm er synlige uten scrolling — spesielt viktig på mindre telefoner og i landskapsmodus.

- **Søk i bag**
  Ved mange disker blir listen lang. Legg til søkefelt, favoritt-merking, og filtrering (favoritter + de fire disktypene) i Bag-fanen.

- **Varsel om ny versjon**
  Cache-first service worker betyr at man i dag ser oppdateringer først ved *andre* besøk etter en deploy. En liten "Ny versjon tilgjengelig — trykk for å oppdatere"-melding (lytt på `updatefound`/`controllerchange` på service worker-registreringen) ville fjernet forvirringen.

- **Små forbedringer:**
  - Vis antall valgte disker på "Start runde →"-knappen
  - Sortering av disker i Bag (f.eks. alfabetisk, etter type, etter snittavstand)
  - Tydeligere fargekoding per disktype (utover brukervalgt swatch-farge)
  - Eget GPS-statusikon i header, ikke bare tekst på måleskjermene
  - Haptisk tilbakemelding (vibrasjon) ved vellykket GPS-registrering, der nettleseren støtter det
  - Flere filtermuligheter i historikken (utover dagens vind/kasttype-filter — f.eks. dato-intervall)
  - Vis vind/dato på Oversikt-skjermen, og vind per rad i kastlisten i disk-detalj

## Prioritet 4 – Data

- **Sikkerhetskopi av alt (eksporter/importer runder+disker)**
  All historikk ligger i dag kun i `localStorage` på én telefon. Mistes telefonen eller ryddes nettleserdata, er alt borte for godt. En "Eksporter alt"/"Importer alt"-funksjon (JSON, full runde-for-runde-data — samme mønster som dagens disk-import/eksport) ville gitt en reell sikkerhetskopi og mulighet til å flytte data mellom enheter. Bør trolig prioriteres høyt til tross for at den står under "Prioritet 4" i denne lista — lav innsats, høy risiko ved å ikke ha den.

- **Eksporter kastdata som CSV**
  Enklere, read-only variant/supplement til punktet over: CSV-eksport av alle registrerte kast for åpning i regneark/ekstern analyse. Ikke nødvendigvis egnet for reimport (i motsetning til JSON-sikkerhetskopien over), men raskere å få øye på for enkel manuell gjennomgang.

## Langsiktig

- **Capacitor / nativ app**
  Pakk dagens HTML/JS-kode i et nativt app-skall (`@capacitor/core` + `@capacitor/geolocation`) hvis nettleser-GPS fortsatt ikke er godt nok etter watchPosition-omleggingen, eller hvis man trenger stabil bakgrunnssporing mens skjermen er låst (iOS dreper ofte GPS-sporing i bakgrunnsfaner). Gir tilgang til app-butikkene, men er et større steg (bygge-pipeline, plattform-spesifikk publisering) — kun aktuelt hvis de nettleser-baserte forbedringene viser seg utilstrekkelige i praksis. Nevnt som fremtidig mulighet i README.md.

- **Cloud-synkronisering**
  Synkroniser historikk mellom flere enheter (f.eks. Supabase eller Firebase). Forutsetter en beslutning om backend/konto-løsning — en betydelig arkitekturendring fra dagens rene frontend-app uten server.

- **AI-analyse**
  Når brukeren har registrert mange kast, la appen foreslå hvilke disker som overlapper i bruksområde, hvilke som bør brukes mer, hvilke kast som bør trenes, og hvilke disker som kan tas ut av bagen. Forutsetter et betydelig datagrunnlag (mange runder) for å gi mening, og sannsynligvis ekstern AI-tjeneste eller egen regelmotor.
