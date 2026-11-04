# STUDIO.flp – rezervace studia ve stylu FL Studia

Rezervační kalendář pro jedno sdílené nahrávací studio a čtyři kamarády (Herb, Olb, Žaludos, Beky).
Celé UI napodobuje FL Studio: kalendář je „Playlist / Arrangement“, statistiky jsou „Mixer“, výběr uživatele je „Channel rack“
a bouncy z Google Disku jsou „Playlist – Tracky“. Uživatel (Herb = Michal) píše česky; UI, komentáře v kódu i commity jsou česky.

- Živá stránka: https://michalh69.github.io/studirez/ (GitHub Pages z větve `main`, repo `MichalH69/studirez`)
- Firebase projekt: `studio-rezervace` (Firestore, žádný login)

## Soubory
- `index.html` – celá aplikace: CSS, HTML i JS v jednom souboru, bez build kroku a bez závislostí kromě Firebase SDK
  (ES moduly z gstatic, načítané dynamicky) a Google Fonts.
- `firestore.rules` – pravidla databáze. **Nenasazují se samy**: po změně je musí uživatel vložit do Firebase konzole
  (Firestore → Rules → Publish). Vždy mu to připomeň.
- `README.md` – návod pro lidi (ovládání, nasazení, zapnutí Drive API).

**Proč jeden soubor:** hostuje se zdarma na GitHub Pages, upravuje se často z telefonu přes Claude, žádný toolchain.
Nezaváděj bundler, framework ani npm. Kód drž v existujících sekcích (`/* ====== NÁZEV ====== */`).

## Pohledy a okna (FL metafora)
| FL Studio | Tady | Kód |
|---|---|---|
| Playlist – Arrangement | Týdenní kalendář: stopy = dny, pravítko = hodiny, klipy = session | `renderPlaylist()`, drag & drop `#plScroll` |
| (vlastní) | Denní pohled na výšku, hlavně pro mobil (klávesa D) | `renderDay()`, `.m-*` |
| Mixer (F9) | Statistiky týdne: fader = hodiny ve studiu na člověka. V titulku tlačítko **TEST** | `renderMixer()` |
| Channel rack (F6) | „Kdo jsi“: armneš svůj kanál, kroky = dny se session | `renderRack()` |
| Playlist – Tracky (F5) | Všechny odkazy na Google Disk ze všech session jako songy s waveformem a přehrávačem | sekce `TRACKY`, objekt `tr` |
| Hint bar vlevo nahoře | Nápověda pro prvek pod myší (`data-hint="Titulek|text"`) a hlášky `flash()` | `setHint`, `flash` |
| Transport | ⏪/⏩ týden, ▶ skok na teď, ● REC = najdi nejbližší volné 2 h | `goWeek`, `goNow`, `quickRec` |

Klávesy: ←/→ týden, mezerník = teď (nebo play/pauza, když je otevřený playlist tracků), R = nová session,
T/D = týden/den, Delete = smazat vybranou vlastní session (s potvrzením), F5/F6/F9 = okna, Esc = zavřít.
F5 záměrně přebíjí reload (jako v FL); Ctrl/Shift+F5 reload nechává.

## Data
### Firestore `bookings/{auto-id}`
```
day: 'YYYY-MM-DD'      start, end: int minuty od půlnoci dne `day` (480–1800; >1440 = po půlnoci, „+1“)
owner: jméno z PEOPLE  type: 'open' | 'private'    members: [1–4 jmen, vždy včetně owner]
note: ≤ 60 znaků       links: [{ id, url, label, by, at }] (≤ 30)
```
Časová osa: 08:00 až 06:00 dalšího dne (`MIN_START`, `MAX_END`), výchozí zobrazení 10–24, snap 30 min, týdny podle ISO.
Otevřená session = ostatní se můžou přidat; soukromá = studio jen pro majitele. Překryvy hlídá `validate()`.
Budoucí session upravuje/maže jen majitel (editor); proběhlou může smazat kdokoli z `members` (tlačítko v detailu),
s ní zmizí i její odkazy z Tracky.

### Firestore `waves/{driveFileId}` – spočítané waveformy
`name, mime, size, sum (md5 nebo size), dur (s), ch, peak, rms (dBFS), w, v:1, at`.
`w` = base64 z 600 bajtů maxim + 600 bajtů |minim| (0–255). Počítá je prohlížeč, který soubor uvidí první;
`sum` pozná, že se soubor na Disku změnil, a pak se analyzuje znovu.

### Bez přihlášení – proč a jak
Čtyři kamarádi, žádné účty: číst/zapisovat může kdokoli s odkazem, ale pravidla pustí jen správně tvarovaná data
s jmény z `names()`. Bezpečnost tedy stojí na `firestore.rules`, ne na UI. Kdo jsi, si pamatuje jen prohlížeč.
**Přidání člověka:** `PEOPLE` v `index.html` (jméno + barva) **a** `names()` ve `firestore.rules`.
**Změna tvaru bookingu:** `clean()` v Firebase backendu, `validBooking()` v pravidlech, lokální backend.

### Backend abstrakce
Všechny zápisy jdou přes objekt `api` (`create, update, remove, join, leave, addLink, removeLink, saveWave`)
a všechna data se vrací jedinou cestou přes `onData(list)` (Firestore živě přes `onSnapshot`).
`initBackend()` vybere jeden ze tří režimů (`api.mode`):
- `firebase` – ostrý provoz, LED u „TÝDEN“ zelená.
- `local` – když se Firebase nenačte; data v `localStorage['studio-daw-v2']`, LED oranžová.
- `test` – **testový režim** (tlačítko TEST v titulku mixeru): data v `localStorage['studio-daw-test-v1']`,
  nasazená z `testSeed()`; do Firestore se nic nezapíše. Přepnutí = `setTestMode()` + reload.
  Seed jsou proběhlé session s opravdovými bouncy, které poslal uživatel (vlastník souboru pepppler@ = Olb,
  martin.zaloudek@ = Žaludos, Herb u všech). Do seedu nedávej nic z osobních souborů, které uživatel výslovně neposlal.

Ostatní `localStorage`: `studio-daw-user` (kdo jsem), `studio-daw-waves` (cache waveformů, max 150),
`studio-daw-wavebad` (soubory, které tenhle prohlížeč neumí dekódovat), `studio-daw-test` (příznak testu).

## Playlist – Tracky (Google Disk)
1. `trCollect()` posbírá odkazy na `drive.google.com` / `docs.google.com` ze všech session (jen http/https),
   sloučí stejné soubory (podle Drive ID) a seřadí nejnovější nahoře. Složky se dají rozbalit na audio soubory.
2. `trMeta()` načte metadata přes Drive API v3 (`fields`, u složek i výpis obsahu).
3. `trPump()` / `trAnalyze()` stáhne audio (`alt=media`, s průběhem), dekóduje přes `OfflineAudioContext.decodeAudioData`,
   `waveOf()` spočítá 600 sloupců + peak/RMS a výsledek uloží lokálně i do Firestore (`api.saveWave`).
   Jeden soubor naráz; na počítači běží na pozadí, na mobilu jen při otevřeném okně; limity 200 MB / 80 MB.
4. Přehrávání: jeden `Audio` element se `src` přímo na Drive API (Range = funguje přetáčení),
   playhead přes `requestAnimationFrame`, Media Session (ovládání z lockscreenu), auto-přechod na další track.
5. Ikonka 🔗 v rohu klipu (`trLinks`) otevře popover s odkazem (kopírovat/otevřít) a seznamem session, kde byl přidaný
   (`refs` z `trCollect`); „Otevřít“ skočí na týden session a otevře její detail.
6. Záloha: bez klíče, u neveřejných souborů nebo formátů, které prohlížeč neumí (AIFF v Chrome),
   hraje vložený přehrávač Google Disku (`/file/d/ID/preview`) v doku nad seznamem.

**Proč v prohlížeči:** stránka je statická (GitHub Pages), žádný server. Drive API podporuje CORS a Range,
takže stažení i streamování jde přímo z prohlížeče; cache ve Firestore zajistí, že se každý soubor analyzuje jednou.

**Drive API klíč:** Firebase klíč má Drive API zablokované (`API_KEY_SERVICE_BLOCKED`), proto vlastní `DRIVE_API_KEY`
(GCP Credentials: „Drive API - playlist tracku (studirez)“, omezený na Google Drive API + referrer `https://michalh69.github.io/*`).
Kdyby byl `null`, běží jen záloha přes přehrávač Disku. Fatální chyby klíče (`FATAL`) přepnou celé okno do zálohy.
Soubory musí být sdílené „Kdokoli s odkazem“.

Rozložení: klipy mají šířku v % podle délky songu vůči nejdelšímu (`tr.span`), mřížka přes `--st` v %,
takže se nic nepřepočítává při změně velikosti okna. Zobrazení waveformu je normalizované (max +16 dB).

## Design
- Tmavé FL Studio 20/21: proměnné v `:root` (`--bg0`, `--win`, `--grid-*`), akcent **oranžová** `--orange`
  (playhead, LCD, aktivní prvky), **zelená** `--green` (LED, play, online), `--red` (REC, chyby).
- Písma: Barlow (UI), Barlow Condensed (logo), Share Tech Mono (LCD, čísla, štítky v kapitálkách).
- Stavební kostky: `.module` (blok v horní liště), `.win` + `.win-title` + `.win-toolbar` (FL okno),
  `.lcd`, `.tbtn`/`.tool` (tlačítka, `aria-pressed` = rozsvícené), `.led`, `.chip`, `.seg`, `.btn`,
  `.clip` (barevná hlavička podle člověka + tělo s waveformem), dialogy `.backdrop > .dialog`.
- Každý člověk má barvu v `PEOPLE`; klipy, LED a stopy ji přebírají přes `--c` / `--m`.
- Waveformy v klipech kalendáře jsou jen dekorace (`wavePath()` ze seedu); skutečné jsou jen v Tracky.
- Mobil (`max-width: 760px`, `isMobile()`): denní pohled, FAB „REC“, dialogy jako spodní sheet, okna přes celou obrazovku,
  dotykové cíle ≥ 40 px, `env(safe-area-inset-*)`. Každou změnu UI kontroluj i na mobilu.
- Texty: krátké, česky, správné plurály (`plural(n, 'track', 'tracky', 'tracků')`), spojovník „–“, čísla s čárkou.

## Konvence v kódu
- Malé helpery: `$` = querySelector, `esc()` **vždy** na uživatelská data v `innerHTML`, `fmtT/fmtTL/fmtH/fmtDate/fmtS/fmtDb`.
- Stav v `state` (kalendář) a `tr` (tracky); render funkce přepisují `innerHTML` a znovu se volají po změně dat.
  Časté věci (playhead, průběh stahování) se kreslí cíleně bez celého renderu (`trHeadPaint`, `trJobPaint`).
- Hlášky přes `flash(titulek, text, 'ok'|'err')`, nápověda přes `data-hint`.
- Žádné nové závislosti; moderní CSS (`color-mix`, container queries) je v pořádku.

## Nasazení
- `git push origin main` = nasazení na GitHub Pages během pár minut. Commit messages česky, krátce.
- Změny `firestore.rules` musí uživatel ručně publikovat ve Firebase konzoli.
- Ostrou databázi při testech jen čti, nikdy do ní nezapisuj (na zkoušení je testový režim).

## Testování (Windows stroj uživatele)
- Není tu Node. Je tu Python 3.10, ffmpeg a nainstalovaný Chrome. Playwright: `python -m venv` v `%TEMP%`,
  `pip install playwright`, spouštět `chromium.launch(channel='chrome')` (bez `playwright install`).
- Stránku servíruj přes `page.route('https://michalh69.github.io/studirez/', …fulfill(index.html))`, ať sedí origin.
  Lokální režim vynutíš zablokováním `https://www.gstatic.com/firebasejs/**`; testový režim přes tlačítko TEST.
- Drive API mockuj routou na `https://www.googleapis.com/drive/v3/files**` (metadata JSON, `alt=media` s podporou Range).
  Mockovaná velká média se v headless Chrome rozjíždějí pomalu; to je artefakt mocku, ne chyba.
- **Skutečné soubory přes Drive API opakovaně nestahuj.** Po pár desítkách stažení z jedné IP vrací Google na `alt=media`
  stránku „Sorry… automated queries“ (403 bez CORS) na minuty až hodiny. Stránka to ukáže jako „DISK TEĎ NEDÁ STÁHNOUT“
  a hraje přes přehrávač Disku. Pravidla ověřuj nanečisto přes Firebase Rules API `projects/studio-rezervace:test`.
- Testovací audio generuj ffmpegem s neutrálními názvy. Výstup Pythonu pouštěj s `PYTHONIOENCODING=utf-8`.
- Po testu smaž dočasné složky (hlavně stažené kopie songů).

## Google Cloud / Firebase z příkazové řádky
- Na stroji uživatele je `gcloud` v `%LOCALAPPDATA%\Google\Cloud SDK\google-cloud-sdk\bin\gcloud.cmd`, přihlášený účtem,
  který projekt vlastní (výchozí projekt `studio-rezervace`). V Git Bash se cesta s mezerou rozbije, když přesměruješ
  stdout/stderr do souboru; spolehlivější je PowerShell (`& $g ...`).
- Pravidla jde publikovat bez konzole přes Firebase Rules API s tokenem z `gcloud auth print-access-token`
  (hlavička `x-goog-user-project: studio-rezervace`): `POST /v1/projects/studio-rezervace/rulesets` se zdrojem
  a pak `PATCH /v1/projects/studio-rezervace/releases/cloud.firestore` s novým `rulesetName`.
  Před publikací vždy stáhni nasazený ruleset a porovnej ho s gitem, ať nepřepíšeš ruční úpravu z konzole.
- Klíč je omezený na referrer stránky, takže v Playwright testech přes `page.route` na skutečnou URL funguje i opravdové Drive API.
