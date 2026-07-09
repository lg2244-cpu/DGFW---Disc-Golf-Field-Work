# Kastlinje – TODO

## 1. Statistikk å hente ut fra kastene
Utover det som finnes i dag (lengst/snitt/kortest per disk, kronologisk kastliste med enkelt stolpediagram), vil vi bygge ut med:

- **Konsistens/spredning** — standardavvik (eller min–maks-range) i kastelengde per disk. Sier noe om hvor forutsigbar disken er, ikke bare hvor langt den går i snitt.
- **Sideveis tendens (fade/turn-bias)** — gjennomsnittlig sideavvik per disk over mange kast. Avslører om en disk systematisk drar venstre (turn) eller høyre (fade) mer enn forventet.
- **Utvikling over tid** — trend i kastelengde per disk over uker/måneder, ikke bare en rå kronologisk liste. Går man lenger med denne disken enn for en måned siden?
- **Sammenligning mellom disker** — rangere disker mot hverandre, gjerne gruppert på type (driver/midrange/putter), på avstand og treffsikkerhet.
- **Forhold under kastet** — registrere vind (med/mot/sidevind) og kasttype (hyzer/anhyzer/flatt) per kast, slik at statistikken over kan filtreres/brytes ned på disse. F.eks: "Destroyer i medvind, hyzer" vs "Destroyer i motvind, flatt".

### Avklarte beslutninger
- **Valgfritt**: vind og kasttype er begge valgfrie — man skal fortsatt kunne registrere raskt i felt uten å fylle ut alt.
- **Vind settes én gang per runde** (på startpoint- eller aim-skjermen, før kastregistreringen starter) — vinden endrer seg sjelden mye i løpet av en runde. Kasttype (hyzer/anhyzer/flatt) velges per kast, i den aktive kast-registreringsvisningen.
- **Vind = retning + styrke**: retning (med/mot/side) og styrke (svak/middels/sterk).

### Datamodell-konsekvenser
- Runde-objektet (`rounds[i]`) får et `wind` felt: `{direction: 'med'|'mot'|'side'|null, strength: 'svak'|'middels'|'sterk'|null}`, satt én gang per runde.
- Hvert kast (`results[i]` / `rounds[i].throws[j]`) får et `throwType` felt: `'hyzer'|'anhyzer'|'flatt'|null`.
- Eksisterende lagrede runder i `localStorage` mangler disse feltene — statistikk og filtrering må håndtere `undefined`/manglende verdier på gamle kast uten å krasje (behandle som "ukjent/ikke satt", ikke som en egen kategori i filtre).

## 2. Diskregister (bag) — rediger/slett
I dag kan man kun legge til disker, ikke redigere eller slette dem.
- Legg til rediger-knapp per disk-kort i baggen (endre navn/type/farge)
- Legg til slett-knapp med bekreftelse (påvirker historikk — avklar om historiske kast med slettet disk skal beholdes eller skjules)

## 3. Historikk — slett runde
I dag kan man ikke fjerne en feilregistrert runde.
- Legg til slett-knapp per runde (evt. i en ny "alle runder"-visning, siden dagens historikk kun viser per-disk)
- Bekreftelsesdialog før sletting

## 4. Git + publisering
- Init git-repo i prosjektmappen
- Publiseres til **GitHub Pages** (avklart med bruker) — krever GitHub-repo, push, og aktivering av Pages i repo-settings
