# Felhőalapú számítástechnika gy. - Féléves beadandó
**Készítette:** Henczes Gábor Tas (Neptun-kód: JSQL3E)
**Tárgykód:** LBT_PI255G3
**Intézmény:** Eszterházy Károly Katolikus Egyetem

## Választott feladat
**GitHub Actions és GitHub Pages alapú CI/CD pipeline témakör részletes kifejtése és működő megvalósítása.**

### A projekt felépítése
* `README.md`: Ez az összefoglaló dokumentum.
* `Dokumentacio/`: Az elméleti háttéranyagot és a futási screenshotokat tartalmazó mappa.
* `.github/workflows/deploy.yml`: A CI/CD folyamatot vezérlő automatizációs szkript.

A pipeline minden `main` branch-re történő feltöltéskor automatikusan lefut, teszteli a környezetet, buildeli a statikus webalkalmazást, majd publikálja a GitHub Pages felhőinfrastruktúrájára.