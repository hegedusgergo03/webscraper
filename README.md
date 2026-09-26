# Socio.hu Web Scraper - AI Fejlesztői Specifikáció

## A Projekt Célja
A cél egy Python alapú web scraper készítése, amely a [socio.hu archívumából](https://socio.hu/index.php/so/issue/archive) kigyűjti az összes eddig megjelent lapszámot és az azokban található cikkek/tanulmányok metaadatait. Az eredményt egy pontosan meghatározott szerkezetű Excel fájlba kell menteni.

## A Forrás
- **Kezdő URL:** `https://socio.hu/index.php/so/issue/archive`
- A kódnak fel kell térképeznie az archívumban található összes évfolyamot és lapszámot, majd minden lapszám oldaláról ki kell nyernie a benne publikált cikkek adatait.

## Elvárt Kimenet (Kimeneti Adatstruktúra)
A végeredményt egy pandas DataFrame-en keresztül egy Excel fájlba (pl. `socio_scraped_data.xlsx`) kell exportálni. A fájl oszlopainak **pontosan** a következőnek kell lenniük (a `rögzítő_társszerző_network.xlsx` minta alapján):

1. `Szerző neve`: A kinyert szerző(k) neve.
2. `Szerző affiliáció`: A szerző intézményi kötődése (ha elérhető a cikk részleteinél; ha nem, maradjon üres string/None).
3. `Tanulmány címe`: A megjelent tanulmány pontos címe.
4. `Folyóirat`: Fix érték: `"Socio.hu"`.
5. `ÉV`: A lapszámhoz tartozó megjelenési év.
6. `Tanulmány ID`: Egyedi azonosító a cikkhez (ez lehet a cikk linkje/URL-je, a kinyert DOI szám, vagy a rendszerük által használt belső ID).
7. `Téma`: A lapszámon belüli rovat, amelybe a cikk tartozik (pl. "Tanulmányok", "Recenzió", "Vita", stb.).

## Speciális Üzleti Logika: A Többszerzős Cikkek Kezelése (Kritikus!)
A legfontosabb elvárás az adatbázis normalizálása a hálózatkutatási (network) elemzésekhez.
- Ha egy cikknek/tanulmánynak **több szerzője van**, akkor a kimeneti Excelben **minden egyes szerzőnek egy külön sort kell kapnia**.
- Ezekben a sorokban a személyhez kötődő adatokon (`Szerző neve` és `Szerző affiliáció`) kívül **minden más mezőnek pontosan meg kell egyeznie** (`Tanulmány címe`, `Folyóirat`, `ÉV`, `Tanulmány ID`, `Téma`).
- *Példa:* Ha egy cikket Háp Károly és Gipsz Jakab írtak együtt, a táblázatban két sor jön létre. Mindkét sorban ugyanaz lesz a címe, éve és ID-ja a cikknek, de az első sorban Háp Károly, a másodikban Gipsz Jakab lesz a `Szerző neve` oszlopban. Ennek leprogramozásához figyelj a szerzők neveit elválasztó karakterek (vessző, "és", "and") helyes darabolására (split).

## Technikai Elvárások az AI Kódoló Agent Felé
1. **Alkalmazott technológiák:** Python 3, `requests` és `BeautifulSoup` (bs4) a webes adatgyűjtéshez. Az adatok manipulációjához és az Excel mentéshez használd a `pandas` és `openpyxl` könyvtárakat.
2. **Bejárási logika (Crawling):**
   - Kezdd az archívum oldallal.
   - Keresd meg és kövesd a href linkeket a dedikált lapszámok URL-jeire.
   - A lapszámok oldalán iterálj végig a rovatokon (`Téma`) és a cikkeken, majd mentsd le a metaadatokat (Szerzők, Cím, Cikk linkje mint ID).
3. **Kivételkezelés (Error Handling):** A scraper ne omoljon össze hiányzó adatok (pl. hiányzó szerző, vagy nem létező rovatcím) esetén. Használj try-except blokkokat és default értékeket.
4. **Adattisztítás:** A kinyert szövegekből (.text) minden esetben futtass `.strip()` metódust a sortörések (`\n`, `\r`, `\t`) és a felesleges whitespace karakterek eltávolítására.

## Fejlesztési Lépések
1. Készítsd el a hálózati kéréseket végrehajtó és a HTML-t feldolgozó (BeautifulSoup) alapstruktúrát.
2. Írd meg a szerzők sztringjét szétválasztó (parsing) függvényt.
3. Építsd fel az adatszerkezetet úgy, hogy iterációnként generálja a duplikált sorokat a többszerzős cikkekhez.
4. Írd meg a kódot, ami létrehozza a pandas DataFrame-et és exportálja az Excel fájlt.
