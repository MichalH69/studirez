# STUDIO.flp – rezervace studia

Rezervační kalendář pro naše studio ve stylu FL Studia. Je to jeden soubor `index.html`.
Data jsou ve Firebase Firestore, stránku hostuje GitHub Pages.

## Jak to funguje
- **Týden / Den:** přepínač pohledu v horní liště (klávesy T / D).
- **Nová session:** táhni v prázdné stopě nebo klikni na ● REC (klávesa R).
- **Otevřená session:** ostatní se můžou přidat. **Soukromá:** studio je jen pro tebe.
- **Detail session:** klikni na klip. Najdeš tam členy a odkazy na Google Disk. Proběhlou session může smazat kdokoli, kdo na ní byl (zmizí s ní i její odkazy z playlistu tracků).
- **Mixer (F9):** statistiky týdne. Tlačítko **TEST** v jeho titulku zapne testový režim: proběhlé session s tracky z Disku, data jen v tomhle prohlížeči, do databáze se nic nezapíše. **↺ OBNOVIT** vrátí testová data do původního stavu.
- **Channel rack (F6):** výběr, kdo jsi.
- LED u „TÝDEN“: zelená = online, oranžová = lokální režim, červená = chyba spojení.

## Playlist – tracky (F5)
- První ikona v nástrojích (nebo F5) otevře playlist se všemi odkazy na Google Disk ze všech session, nejnovější nahoře.
- Audio soubory se samy stáhnou a spočítá se jejich waveform, peak a RMS. Výsledek se uloží do Firestore (kolekce `waves`), takže se každý soubor analyzuje jen jednou pro všechny. Na mobilu se analyzuje jen při otevřeném playlistu.
- ▶ nebo klik do waveformu = přehrávání přímo na stránce, mezerník = play/pauza, po konci tracku hraje další. Složka se rozbalí na jednotlivé audio soubory.
- Soubory musí být sdílené „Kdokoli s odkazem“. Co prohlížeč neumí (třeba AIFF v Chrome) nebo co není sdílené veřejně, pustí přehrávač Google Disku.
- Okno jde na počítači přesouvat za titulek a zvětšovat za pravý dolní roh. Hudba hraje dál i po zavření okna (zelená tečka na ikoně).

### Zapnutí Drive API (jednou)
Bez toho se tracky jen vypíšou a hrají přes přehrávač Google Disku, bez waveformů.
1. [Google Drive API](https://console.cloud.google.com/apis/library/drive.googleapis.com?project=studio-rezervace) → **Enable**.
2. [Credentials](https://console.cloud.google.com/apis/credentials?project=studio-rezervace) → **Create credentials → API key**.
3. U nového klíče: *Application restrictions* → **Websites** → `https://michalh69.github.io/*`. *API restrictions* → **Restrict key** → jen **Google Drive API**. Uložit.
4. Klíč vlož do `DRIVE_API_KEY` v `index.html`.
5. Nová pravidla z `firestore.rules` vlož do Firebase konzole (Firestore → Rules → Publish). Bez nich se waveformy nesdílí a každý prohlížeč si je počítá sám.

## Nasazení
- Každý push do větve `main` se sám nasadí přes GitHub Pages.
- Konfigurace Firebase je v `index.html` (`FIREBASE_CONFIG`). Není tajná, ochranu zajišťují pravidla.
- Pravidla databáze jsou ve `firestore.rules`. Po změně je vlož do Firebase konzole: Firestore → Rules → Publish.

## Přidání člověka
Doplň jméno a barvu do `PEOPLE` v `index.html` a jméno do `names()` ve `firestore.rules`.
