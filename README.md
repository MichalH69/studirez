# STUDIO.flp – rezervace studia

Rezervační kalendář pro naše studio ve stylu FL Studia. Je to jeden soubor `index.html`.
Data jsou ve Firebase Firestore, stránku hostuje GitHub Pages.

## Jak to funguje
- **Týden / Den:** přepínač pohledu v horní liště (klávesy T / D).
- **Nová session:** táhni v prázdné stopě nebo klikni na ● REC (klávesa R). Jde zapsat i zpětně do minulosti. V editoru vybereš, kdo dál jde. Pod **+ Někdo jiný** napíšeš jméno hosta, který nemá kanál, a ten bude šedý.
- **Otevřená session:** ostatní se můžou přidat. **Soukromá:** nikdo další se nepřidá.
- **Detail session:** klikni na klip. Najdeš tam členy a odkazy na Google Disk. Odkaz může přidat kdokoli ke kterékoli session. U nového odkazu musíš vybrat kategorii (beat, track, jiné nebo vlastní), u svých odkazů ji jde kliknutím změnit. Budoucí session upravuje jen majitel, proběhlou může upravit kdokoli. Proběhlou session může smazat kdokoli, kdo na ní byl (zmizí s ní i její odkazy z playlistu tracků).
- **Mixer (F9):** statistiky týdne. Tlačítko **TEST** v jeho titulku zapne testový režim: proběhlé session s tracky z Disku, data jen v tomhle prohlížeči, do databáze se nic nezapíše. **↺ OBNOVIT** vrátí testová data do původního stavu.
- **Channel rack (F6):** výběr, kdo jsi.
- LED u „TÝDEN“: zelená = online, oranžová = lokální režim, červená = chyba spojení.

## Playlist – tracky (F5)
- První ikona v nástrojích (nebo F5) otevře playlist se všemi odkazy na Google Disk ze všech session, nejnovější nahoře.
- Audio soubory se samy stáhnou a spočítá se jejich waveform, peak a RMS. Výsledek se uloží do Firestore (kolekce `waves`), takže se každý soubor analyzuje jen jednou pro všechny. Na mobilu se analyzuje jen při otevřeném playlistu.
- ▶ nebo klik do waveformu = přehrávání přímo na stránce, mezerník = play/pauza, po konci tracku hraje další. Složka se rozbalí na jednotlivé audio soubory.
- **Filtr** nad seznamem: podle kategorie a podle toho, kdo odkaz nahrál. Poslední položka **+ Nová kategorie** v menu kategorií přidá vlastní kategorii s barvou z palety (uvidí ji všichni).
- Proužek vlevo u tracku: horní polovina = kdo to nahrál, spodní polovina = ostatní, kdo byli na session.
- 🔗 v rohu klipu ukáže odkaz na Disk (kopírovat, otevřít) a ve kterých session byl track přidaný. Tlačítko **Otevřít** skočí rovnou na session.
- Soubory musí být sdílené „Kdokoli s odkazem“. Co prohlížeč neumí (třeba AIFF v Chrome) nebo co není sdílené veřejně, pustí přehrávač Google Disku.
- Okno jde na počítači přesouvat za titulek a zvětšovat za pravý dolní roh. Hudba hraje dál i po zavření okna (zelená tečka na ikoně).

### Drive API
Je zapnuté a klíč je v `DRIVE_API_KEY` v `index.html` (v Google Cloud pod názvem „Drive API - playlist tracku (studirez)“).
Klíč smí jen Google Drive API a jen ze stránky `https://michalh69.github.io/*`, takže nevadí, že je veřejně v kódu.
Kdyby bylo potřeba ho vyměnit:
1. [Credentials](https://console.cloud.google.com/apis/credentials?project=studio-rezervace) → **Create credentials → API key**.
2. U klíče: *Application restrictions* → **Websites** → `https://michalh69.github.io/*`. *API restrictions* → **Restrict key** → jen **Google Drive API**.
3. Nový klíč vlož do `DRIVE_API_KEY` a starý smaž.

## Nasazení
- Každý push do větve `main` se sám nasadí přes GitHub Pages.
- Konfigurace Firebase je v `index.html` (`FIREBASE_CONFIG`). Není tajná, ochranu zajišťují pravidla.
- Pravidla databáze jsou ve `firestore.rules`. Po změně je vlož do Firebase konzole: Firestore → Rules → Publish.

## Přidání člověka
Doplň jméno a barvu do `PEOPLE` v `index.html` a jméno do `names()` ve `firestore.rules`.
