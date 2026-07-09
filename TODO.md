# Kastlinje – TODO

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

## 4. Git + publisering — delvis ferdig
- ✓ Lokalt git-repo initialisert, initial commit gjort.
- ✓ Privat GitHub-repo opprettet og pushet: https://github.com/lg2244-cpu/DGFW---Disc-Golf-Field-Work
- ⏸ **GitHub Pages er ikke aktivert** — krever enten at repoet gjøres offentlig, eller GitHub Pro/Team (Pages støtter ikke private repos på gratis-plan). Avventer beslutning fra bruker om hvordan appen skal hostes for faktisk telefonbruk (GPS krever HTTPS/ekte hosting, fungerer ikke fra `file://`).
