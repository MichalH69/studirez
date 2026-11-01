# STUDIO.flp – rezervace studia

Rezervační kalendář pro naše studio ve stylu FL Studia. Je to jeden soubor `index.html`.
Data jsou ve Firebase Firestore, stránku hostuje GitHub Pages.

## Jak to funguje
- **Týden / Den:** přepínač pohledu v horní liště (klávesy T / D).
- **Nová session:** táhni v prázdné stopě nebo klikni na ● REC (klávesa R).
- **Otevřená session:** ostatní se můžou přidat. **Soukromá:** studio je jen pro tebe.
- **Detail session:** klikni na klip. Najdeš tam členy a odkazy na Google Disk.
- **Mixer (F9):** statistiky týdne.
- **Channel rack (F6):** výběr, kdo jsi.
- LED u „TÝDEN“: zelená = online, oranžová = lokální režim, červená = chyba spojení.

## Nasazení
- Každý push do větve `main` se sám nasadí přes GitHub Pages.
- Konfigurace Firebase je v `index.html` (`FIREBASE_CONFIG`). Není tajná, ochranu zajišťují pravidla.
- Pravidla databáze jsou ve `firestore.rules`. Po změně je vlož do Firebase konzole: Firestore → Rules → Publish.

## Přidání člověka
Doplň jméno a barvu do `PEOPLE` v `index.html` a jméno do `names()` ve `firestore.rules`.
