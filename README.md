# Felhőalapú számítástechnika gy. - Féléves beadandó

**Készítette:** Henczes Gábor Tas (Neptun-kód: JSQL3E)  
**Tárgykód:** LBT_PI255G3  
**Intézmény:** Eszterházy Károly Katolikus Egyetem  

🌐 **Weboldal megtekintése:** [Kattints ide az oldal megnyitásához](https://tas-h.github.io/Felhoalapu-szamitastechnika-Beadando-JSQL3E/)

---

## Választott feladat

**GitHub Actions és GitHub Pages alapú CI/CD pipeline témakör részletes kifejtése és működő megvalósítása.**

### A projekt felépítése

* **`README.md`**: Ez a fő összefoglaló dokumentum a projekt gyökerében.
* **`Dokumentacio/`**: A részletes elméleti háttéranyagot, a szakmai leírást és a futási eredményeket tartalmazó mappa.
  * **`README.md`**: A Dokumentacio mappán belüli részletes szakmai és infrastrukturális leírásom.
  * **`kepek/`**: A működést és az implementációt igazoló screenshotok helye.
* **`.github/workflows/deploy.yml`**: A CI/CD folyamatot vezérlő automatizációs szkriptem.

---

## A pipeline működése és felépítése

A futószalagot úgy alakítottam ki, hogy modellezze az ipari környezetben elvárt biztonsági és minőségi szűrőket. Az automatizáció két fő fázisból (Job) áll:

1. **Validációs fázis (Code Validation / CI):** Minden feltöltéskor vagy Pull Request nyitásakor automata környezetben ellenőrzi a commit üzenetek formátumát (Commit Linting) és a HTML kód szintaxisát (HTML Linting).
2. **Élesítési fázis (Deployment / CD):** A sikeres validáció után a rendszer önműködően publikálja a statikus webalkalmazást a GitHub Pages felhőinfrastruktúrájára. A pipeline-ba épített védelem biztosítja, hogy a tényleges élesítés kizárólag a `main` ágra való push vagy merge után hajtódjon végre.

---

## Részletes dokumentáció és eredmények

A projekt kimerítő elméleti hátterét, a felhőalapú virtuális gépek (Runnerek) életciklusának magyarázatát, a szerver nélküli működés (CDN, DNS maszkolás) technikai részleteit, valamint a működést igazoló screenshotokat a belső **[Dokumentacio/README.md](Dokumentacio/README.md)** fájlban vezettem le.
