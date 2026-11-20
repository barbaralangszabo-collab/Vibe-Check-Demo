# Vibe Check – Csibe-foglalkozások gyakorlatbankja

Egyfájlos webapp (index.html), ami Supabase-ből olvassa a gyakorlatokat, a témákat és a nyomtatnivalókat.
Ha az index.html elején üres a Supabase-beállítás, **demó módban** fut a beépített adatokkal.

## A csomag tartalma

| Fájl | Mire való |
|---|---|
| `index.html` | Maga az app. Ezt kell feltölteni GitHub Pagesre (vagy bármilyen tárhelyre). |
| `schema.sql` | Táblák, jogosultságok (RLS) és fájltárolók létrehozása. |
| `schema_meghivo.sql` | Meghívó linkek és jelszavas belépés, levélküldés nélkül. |
| `schema_naplo.sql` | Napló, naptár és évfolyam-terv: patrónusi csoportok, közös évfolyam-foglalkozások, megtartott alkalmak. |
| `seed.sql` | A 17 téma (10 közös + 7 évfolyamsávos), 145 gyakorlat és 23 nyomtatnivaló betöltése. Újrafuttatható. |
| `data.json` | Ugyanezek az adatok JSON-ban (biztonsági mentésnek). |
| `tema-fotok/` | Mind a 17 téma fotója WebP-formátumban, három méretben (640, 800 és 1280 px). Az `index.html` mellé kell feltölteni. |
| `manifest.webmanifest`, `sw.js`, `icon-*.png` | A telefonra telepíthető változathoz. Az `index.html` mellé kell feltölteni. |

## Beállítás lépésről lépésre

### 1. Supabase-projekt
Készíts egy új projektet a supabase.com-on (az ingyenes csomag elég).
A Learning Hub projektjébe is teheted, de akkor előbb nézd meg, hogy nincs-e már `themes`, `activities` vagy `printables` nevű tábla.

### 2. Adatbázis
SQL Editor → New query:
1. Másold be a `schema.sql` teljes tartalmát → **Run**.
2. Másold be a `schema_naplo.sql` tartalmát → **Run**. (Ha a gyakorlatbank már fut, elég csak ezt lefuttatni; újrafuttatható, a meglévő adatokat nem bántja.)
3. Másold be a `seed.sql` tartalmát → **Run**.
4. Állítsd be magad adminnak (a saját címeddel):
   ```sql
   insert into public.admins (email) values ('sajat.cimed@example.com');
   ```
Az admin tud témafotót feltölteni, és szerkesztheti az adatokat. Mindenki más csak olvasni tud.

### 3. Nyomtatnivalók feltöltése
Storage → `nyomtatnivalok` tároló → **Upload files** → húzd be az 54 PDF-et (a zip `nyomtatnivalok` mappájából). Utána futtasd újra a `seed.sql`-t, hogy az új nyomtatnivalók és kapcsolásaik bekerüljenek az adatbázisba.
A fájlnevek maradjanak változatlanok (pl. `22_Narancs_feladat_instrukciok.pdf`), mert az app ezek alapján keresi őket.
Ez a tároló privát: csak bejelentkezett patrónusok tudnak letölteni belőle, 10 percig érvényes linkkel.

### 4. Meghívók és jelszavas belépés
SQL Editor → New query → másold be a `schema_meghivo.sql` tartalmát → **Run**.

Authentication:
- **Sign In / Providers → Email:** legyen bekapcsolva, a **„Confirm email”** pedig legyen **kikapcsolva**.
- **Sign In / Providers → „Allow new users to sign up”:** legyen **bekapcsolva**. Az adatbázis gondoskodik róla, hogy csak meghívott (vagy admin) e-mail-címmel lehessen regisztrálni.

Így a Supabase egyetlen levelet sem küld, ezért SMTP-t sem kell beállítani.

### 5. Saját belépés és a patrónusok meghívása
- **Te (admin):** Authentication → Users → **Add user → Create new user**, a saját címeddel és egy jelszóval (az „Auto confirm user” legyen bepipálva).
- **Patrónusok:** az appban, adminként: Csibe-alkalmak → lent: **Patrónusok, összesítés, visszajelzések → 👥 Patrónusok meghívása**. Másold be az e-mail-címeket, kattints a **Meghívók létrehozása** gombra, és küldd el mindenkinek a saját linkjét (✉ E-mail gomb, vagy 📋 Link / Az összes link másolása).
- A patrónus a linken jelszót állít be, és az első belépéskor maga választja ki az évfolyamát és a csoportját.
- **Elfelejtett jelszó:** kattints az illető mellett az **↻ Új link** gombra, és küldd el neki: a linken új jelszót állíthat be.

### 6. Az app összekötése
Project Settings → API: másold ki a **Project URL**-t és az **anon public** kulcsot, és írd be az `index.html` elején:
```js
const SUPABASE_URL      = 'https://xxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';
```
Az anon kulcs nyilvános lehet: a jogosultságokat az adatbázis (RLS) védi.

### 7. Feltöltés GitHub Pagesre
Új repó (pl. `Vibe-Check-Gyakorlatbank`) → töltsd fel az `index.html`-t, a `tema-fotok` mappát, valamint a `manifest.webmanifest`, `sw.js`, `icon-192.png`, `icon-512.png` és `icon-maskable-512.png` fájlokat (a nevek maradjanak változatlanok) → Settings → Pages → Branch: `main`. A `supabase` mappát nem kell feltölteni.
**A PDF-eket ne töltsd fel a GitHub-repóba**, mert az nyilvános. A PDF-ek helye a Supabase-tároló.

## Használat

- **Keresés:** ékezet nélkül is működik, és rokon szavakra is talál (pl. „veszekedés” → konfliktuskezelés). A „10 perc” típusú keresés időre szűr.
- **Szűrők:** téma, idő, évfolyam.
- **Témaoldal:** az alkalom menete időbeosztással; a teljes alkalom egy gombbal betervezhető.
- **Foglalkozás tervezése:** a gyakorlatok `+` gombjával vagy a Javaslat „Betervezés” gombjával lehet összerakni a következő foglalkozást. Tervezés közben alul egy sáv mutatja, hány gyakorlat és hány perc van benne; a „Folytatás” gombbal lehet átrendezni és betervezni.

## Csibék (a heti patrónusi alkalmak)

Az app három részből áll: **Kezdőlap** (a következő Csibe nagyban: dátum, kép, gyakorlatok; ▶ Indítás · 🖨 Nyomtatás · ✏️ Módosítás, alatta a következő Csibék), **🐣 Csibék** (a tanév összes Csibéje listában, havi nézettel) és **📚 Gyakorlatbank** (évfolyam és téma szerinti böngészés, keresés, Csibe összeállítása). Az Indítás végén jelölöd, hogy megtartottátok a Csibét, és értékeled a gyakorlatokat.

## Évfolyam-terv, napló és naptár

A legegyszerűbb a naptáron keresztül dolgozni: a tanév minden Csibéjére be van osztva egy téma a gyakorlataival (ezt egyszer, a **🗓 Tanévi ütemterv** gombbal kell létrehozni évfolyamonként; alapból a következő szerdától indul). A napra kattintva megnyílnak a gyakorlatok, a **✏️ Módosítás** gombbal pedig a terv átrakható a foglalkozástervezőben.


Évfolyamonként 4 patrónus van, és **mind a 4 ugyanazt a gyakorlatot tartja**. Ezért a foglalkozásokat évfolyamszinten tervezik.

- **Csoport:** első belépéskor a kezdőlap tetején mindenki megadja a saját csoportját és az évfolyamot. A csoport kaphat fiktív nevet (pl. Iránytű, Szikra, Horizont, Üstökös); a 🎲 gomb névötletet ad.
- **Betervezés:** a megtervezett foglalkozást az évfolyam bármelyik patrónusa betervezheti egy napra (alapból a legutóbbi foglalkozás után egy héttel). A foglalkozás ezután mind a 4 patrónusnál megjelenik a kezdőlapon („A 9. évfolyam következő foglalkozása”) és a naptárban (szaggatott keretes jelölés).
- **Megtartottuk:** mindenki a saját csoportjánál jelöli, ha megtartotta a foglalkozást. Az évfolyam patrónusai csak annyit látnak egymásról, hogy ki tartotta már meg (✓ Szikra ○ Horizont …), a jegyzeteket nem. Az app szól, ha egy korábbi foglalkozást még nem jelöltél.
- **Ki mit lát:** minden patrónus a saját évfolyamának terveit és állapotát látja; más évfolyamét nem. Az admin mindent lát, és kap egy évfolyamonkénti összesítést: hány közös foglalkozás volt, a csoportok hány százalékban tartották meg, melyik témánál tart az évfolyam, és melyek a leggyakrabban használt gyakorlatok.

### Témakövető javaslat

A kezdőlapon az évfolyam következő foglalkozásához mindig megjelenik egy javaslat (20, 45 vagy 90 percre), alatta a témahaladással. A haladást az évfolyam közös tervei alapján számolja, a betervezett foglalkozásokat is beleértve:
- amíg egy téma gyakorlatainak nincs vége, **ugyanazt a témát javasolja folytatni** („Ez lesz a 3. foglalkozás ebben a témában a 9. évfolyamon…”);
- a sorrendben következő gyakorlatokat ajánlja, és ha belefér, a téma ráhangolódó és lezáró körét is (ismétlésként is, mert ezek rituálénak is jók);
- ha a téma kész, a **sorban következő** témát ajánlja, mert a témák egymásra épülnek;
- ha egy félbehagyott témáról szeretnétek másik témára váltani, az app jelzi és rákérdez, és tervezéskor is jelzi, ha egy foglalkozás túl sok témát érint. Akkor is szól, ha arra a napra már van terv. A döntés mindig a patrónusoké.

## Évfolyamsávok

A 10 közös téma mind a hat évfolyamon fut. Mellettük 7 évfolyamsávos téma van, ezek csak a megadott évfolyamoknál jelennek meg:

| Téma | Évfolyam |
| --- | --- |
| Az új osztály | 7. |
| Barátság és kirekesztés | 7–8. |
| Digitális élet | 9–10. |
| Kortárs nyomás és nemet mondás | 9–10. |
| Stressz és érettségi | 11–12. |
| Döntések és jövő | 11–12. |
| Búcsú és lezárás | 12. |

A 10 közös témában évfolyamsávonként (7–8., 9–10., 11–12.) egy saját fő gyakorlat és egy saját feldolgozás is van (összesen 60), így a diákok kétévente más anyagot kapnak; a ráhangolódás, a bemelegítő és a lezárás közös. A Javaslat és az ütemterv a sávos gyakorlatot ajánlja előbb, a közös változat alternatívaként marad.

A kezdőlap, a Javaslat, a témahaladás és a tanévi ütemterv csak az adott évfolyamnak való témákat és gyakorlatokat mutatja. 17 meglévő gyakorlatnál sávos változat is van (pl. „Változat a 7–8. évfolyamnak”): a gyakorlat ablakában és Foglalkozásvezetés módban a saját évfolyamhoz illő változat kiemelve látszik.

**Meglévő adatbázisnál:** futtasd újra a `schema.sql` végét (az „Évfolyamsávos kiegészítés” részt), majd a `seed.sql`-t.

A demóban mind a hat évfolyam szerepel, évfolyamonként 4 csoporttal; a fejléc legördülő menüjével lehet váltani közöttük.

## Patrónusi értékelések

A gyakorlat kártyáján rögtön értékelhető, hogyan működött (👍 / 🤔 / 👎, tipp a kollégáknak). Ha a csoportnak még nincs olyan alkalma, amelyben a gyakorlat szerepelt, az értékelés mai alkalomként kerül a naplóba.

Foglalkozás után a patrónus gyakorlatonként jelöli, hogyan működött (👍 működött / 🤔 részben / 👎 nem), és írhat egy rövid tippet a kollégáknak. Ez a „Megtartottuk” alatt vagy a foglalkozásvezetés végén lehetséges.
- **Ki mit lát:** az értékeléseket és a tippeket minden patrónus látja, **név nélkül**, csak az évfolyam látszik. Módosítani mindenki csak a sajátját tudja.
- **Összesítés:** a fejlécben az **📊 Értékelések** oldal mutatja, hány százalékban működött az egyes gyakorlatok (a „részben” fél pontot ér), évfolyamsávonként bontva (7–8., 9–10., 11–12.). Látszik a „Legjobban bevált” és az „Érdemes átgondolni” lista (60% alatt, legalább 3 értékelésből), és a legutóbbi tippek.
- **A gyakorlatoknál is látszik:** a kártyán egy 👍 % jelvény (legalább 3 értékelésnél), a részleteknél a bontás és a tippek.

## Foglalkozásvezetés mód

Az évfolyam következő foglalkozásánál és a naptárban is van egy **▶ Foglalkozás indítása** gomb. Ez teljes képernyőn, nagy betűvel mutatja a lépéseket gyakorlatonként, visszaszámlálóval (−1′/+1′, hangjelzés a végén) és a „Mire figyelj” tanáccsal. Billentyűzettel is irányítható: szóköz = indítás/szünet, nyilak = előző/következő. Foglalkozás közben nem kapcsol le a kijelző. A foglalkozás végén egy képernyőn lehet értékelni és menteni, hogy megtartottátok.

## Előkészítés

A **🧰 Előkészítés és nyomtatás** gomb megmutatja, mit kell kinyomtatni és milyen eszközök kellenek a foglalkozáshoz, az évfolyam csoportszámával felszorozva (pl. „× 4”). Pipálható ellenőrzőlistaként működik, a **Mind egy PDF-be** gombbal pedig az összes nyomtatnivaló egyetlen fájlba fűzhető. Az **🖨 Foglalkozásvázlat nyomtatása** gomb a lépéseket és a „Mire figyelj” tanácsokat nyomtatja ki.

## Tanévi ütemterv

A Naptár oldalon a **🗓 Ütemterv készítése** gombbal egy évfolyam egész évét be lehet tervezni. Meg kell adni az első foglalkozás napját, az utolsó napot, a foglalkozás hosszát (25, 45 vagy 90 perc) és a szüneteket (soronként egy dátum vagy időszak), és ki kell választani a témákat. Az app a témák gyakorlatait sorrendben foglalkozásokra bontja: 90 percben témánként 1 foglalkozás, 45 percben kb. 3 foglalkozás. Ahol belefér, a nyitó és záró kört is beteszi; az egyszeri gyakorlatokat (kezdő kérdőív, csoportszerződés, záróprojekt-választás) nem ismétli. Előnézet után egy gombbal létrehozza az évfolyam-terveket, és szól, ha valami nem fér bele, vagy ha marad tartalék hét. Azokat a napokat kihagyja, amelyekre már van terv.

## Telefonos változat

Telefonon a képernyő alján egy menüsor van: 🏠 Kezdőlap · 📅 Naptár · 📊 Értékelések.

GitHub Pagesen az app telepíthető, és utána saját ikonnal, böngészősáv nélkül nyílik.
- **Androidon (Chrome):** a kezdőlapon megjelenő **📲 Telepítés a telefonra** gombbal, vagy a menüben a „Hozzáadás a kezdőképernyőhöz” ponttal.
- **iPhone-on (Safari):** Megosztás → „Főképernyőhöz adás”.
- **Gyenge wifi mellett** az app a telefonon tárolt példányból is betölt, és a legutóbb letöltött adatokat mutatja. Erről egy „Offline mód” sáv szól. Menteni ilyenkor nem lehet.

## Témafotók

Mind a 17 témához tartozik fotó a `tema-fotok` mappában. A `seed.sql` ezekre az útvonalakra hivatkozik (pl. `tema-fotok/t07-eroszakmentes-kommunikacio-1280.webp`). Minden képből két méret van: a `-640.webp` a kártyákhoz, a `-1280.webp` a nagy képhez, ezért a mappának az `index.html` mellett kell lennie.
- **Csere:** adminként a témaoldalon a **Fotó cseréje** gombbal tölthetsz fel másikat (a Supabase `tema-fotok` tárolójába kerül). Feltöltés előtt az app automatikusan 1280 px szélesre kicsinyíti és WebP-be tömöríti, így a nagy telefonos fotók sem lassítják az oldalt. A `seed.sql` újrafuttatása nem írja felül a feltöltött fotót.
- **Törlés:** a **Törlés** gombbal visszaáll a helyőrző. Ha egy kép nem található, az app automatikusan a helyőrzőt mutatja.
- Ha a képek stockfotók (pl. Canvából), nézd meg a licencüket: nyilvános weboldalon való közzététel előtt érdemes ellenőrizni, hogy szabad-e. Ha diákok is látszanak egy fotón, ahhoz a szülők hozzájárulása kell.

## Tartalom szerkesztése

Table Editor → `activities`: itt javíthatod a gyakorlatok szövegét. Új gyakorlatnál a `type_order` a foglalkozás menetében elfoglalt helyet jelenti (1 = ráhangolódás … 5 = lezárás).
Nyomtatnivaló hozzárendelése: `activity_printables` tábla (gyakorlat ID + nyomtatnivaló ID).

---
Vibe Check – Csibe-foglalkozások gyakorlatbankja · Fejlesztő: Láng-Szabó Barbara


## Csibe-alkalmak: idő szerinti választás és jogosultság (2026. november)

- Minden Csibe alkalom rögzített csomag: a téma fő gyakorlata és a különböző hosszúságú kísérő gyakorlatok, legfeljebb 120 perc.
- A patrónus a Csibénél kiválasztja, mennyi ideje van (30 / 45 / 60 / 90 perc / Mind). Az app ebből mutatja meg, mi fér bele: a fő gyakorlat mindig benne van, utána a feldolgozás, a ráhangolódás, a lezárás és a bemelegítő játék. A tartalom nem változik, a választás csak az adott patrónus gépén tárolódik. Az Indítás és a Nyomtatás a kiválasztott gyakorlatokkal dolgozik.
- A Csibe-alkalmakat (beosztás, szünetek, módosítás, törlés, gyakorlat hozzáadása) csak az admin szerkesztheti. Ehhez futtasd újra a `schema_naplo.sql` fájlt a Supabase SQL Editorban: az új szabályok szerint a patrónusok csak olvassák az évfolyam-terveket.


## Nyomtatnivalók és vetítenivalók (2026. november)

- 54 nyomtatható PDF a `nyomtatnivalok` mappában (23 eredeti + 31 új: helyzet- és szerepkártyák, munkalapok, a Holdon-feladat szakértői kulccsal, kitalált csoportchat stb.). 78 gyakorlathoz tartozik legalább egy.
- 38 gyakorlathoz kivetíthető diák tartoznak (📽 Vetítés): a gyakorlatkártyán, az Indítás közben a Segédanyagok között és a Módosítás panelen. A diák az appba vannak beépítve, nem kell hozzájuk fájl. Léptetés: nyilak, szóköz; kilépés: Esc.
- Demó módban a PDF-ek az `index.html` melletti `nyomtatnivalok/` mappából nyílnak meg, ezért a mappát is fel kell tölteni a GitHub Pagesre.

### Nyomtatási jelölések (15 fős csoport)

Minden nyomtatnivaló jelölést kapott: **🖨 Nyomtatni kell**, **Opcionális** (kivetíthető, vagy üres lapra / füzetbe is elkészíthető) és **👩‍🏫 Csak neked** (felolvasod). Mellette a példányszám egy 15 fős csoporthoz (pl. „15 példány”, „4 készlet”). A Csibe kártyáján egy sor mutatja, mit kell aznap nyomtatni a kiválasztott időhöz, és nagyjából hány lap. Az éles adatbázishoz futtasd le a `schema.sql` végén lévő `alter table public.printables …` sorokat, majd a `seed.sql`-t.

### Pótlás: kimaradt fő gyakorlat

Ha egy csoport a Csibén nem tartotta meg a fő gyakorlatot (az Indítás végén a „Megtartottuk” gomb elmenti, mi volt meg), a következő Csibénél ennél a patrónusnál megjelenik: „↪ Kimaradt a múlt alkalomról: … [Beteszem] [Kihagyom]”. A pótlás csak a saját csoportot érinti, a közös terv nem változik. Csak fő gyakorlat pótlódik, csak egyszer, és csak ha a tanév hátralévő részében nem kerül úgyis sorra. A „Mennyi időtök van?” választásnál a mostani fő gyakorlat után következik, egy Csibe továbbra is legfeljebb 120 perc.
