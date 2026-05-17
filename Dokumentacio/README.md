# Szakmai és Infrastrukturális Dokumentáció: GitHub Actions CI/CD Pipeline
**Készítette:** Henczes Gábor Tas (JSQL3E)  
**Intézmény:** Eszterházy Károly Katolikus Egyetem  
**Kurzus:** Felhőalapú számítástechnika gyakorlat  

---

## 1. Elméleti alapok: Hogyan működik a CI/CD a gyakorlatban?

A hagyományos webfejlesztés során a kész fájlokat a fejlesztő manuálisan, valamilyen FTP klienssel vagy távoli asztali kapcsolattal másolja fel a szerverre. Ez a módszer lassú és hordozza a kockázatot: ha a feltöltött kódban benne marad egy elgépelés vagy egy hibás HTML tag, az éles oldal azonnal összeomlik.

Ezt a manuális folyamatot váltottam ki a **CI/CD (Continuous Integration / Continuous Deployment)** automatizált futószalaggal.

```text
[Helyi kód] --(git push)--> [GitHub Repó] --> [Automata Teszt (CI)] --> [Automatikus Élesítés (CD)] --> [Élő Weboldal]
```

* **Continuous Integration (CI - Folyamatos Integráció):** Amikor feltöltöm a kódot, a rendszer azonnal lefuttat egy ellenőrző fázist. Átnézi, hogy a kódom megfelel-e a technikai követelményeknek (szintaxis, szabályok). Ha hibát talál, leállítja a folyamatot, így a hiba nem kerülhet ki az élő oldalra.
* **Continuous Deployment (CD - Folyamatos Élesítés):** Ha a tesztek sikeresen lefutottak és zöld jelzést adtak, a pipeline önműködően becsomagolja a projektemet, átmásolja a célszerverre, és élesíti a módosításokat.

---

## 2. A háttér-infrastruktúra: Mi történik a felhőben a `git push` után?

A GitHub nem csak egy online tárhely, ahol a fájljaim állnak, a háttérben egy komoly felhőalapú infrastruktúra üzemel (a Microsoft Azure alapjain).

### Miért és mikor indul el a `deploy.yml`?
Amikor a helyi számítógépemen kiadom a `git push` parancsot, vagy a GitHub felületén egy másik ágról **Pull Request**-et (beolvasztási kérelmet) indítok a `main` ág felé, a következő folyamat zajlik le:
1. A GitHub szervere fogadja a Git csomagot.
2. A GitHub belső eseménykezelője (Webhook) észreveszi a változást, és átvizsgálja a `.github/workflows/` mappát.
3. Mivel a `deploy.yml` fájlom tetején beállítottam az `on: push` és `on: pull_request` triggereket a `main` ágra, a GitHub Actions ütemezője tudja, hogy feladat van, így sorba rendezi és elindítja a pipeline-t.

### A Runner (Virtuális Gép) életciklusa

A konfigurációba beírt `runs-on: ubuntu-latest` sor határozza meg a környezetet. Amikor a pipeline elindul, a háttérben a következő virtuális folyamatok futnak le:

1. **Gépkérés a felhőből (Provizionálás):** A GitHub elkülönít a saját szerverparkjában egy erőforrást, és másodpercek alatt felhúz egy teljesen tiszta, üres **Ubuntu Linux** operációs rendszert futtató virtuális gépet (konténert). Ezen a gépen az indítás pillanatában semmi nincs, még az én kódom sem.
2. **Környezet izoláció (Headless működés):** Ez a virtuális gép grafikus felület nélkül (headless módban) működik, csak parancssorból vezérelhető. Első lépésként letölti a GitHub repómból a fájljaimat (ezt végzi az `actions/checkout@v4` lépés), majd pontról pontra végrehajtja a YAML fájlban megadott parancsokat.
3. **Azonnali megsemmisülés (Efemér jelleg):** Amint a pipeline lefutott (akár sikerrel, akár hibával állt le), ez a virtuális gép a másodperc törtrésze alatt teljesen letörlődik és megsemmisül a felhőből. Ez garantálja, hogy a következő futásnál egy teljesen szűz környezetet kapok, és semmilyen korábbi futásból maradt fájl nem zavarhatja be a teszteket.

---

## 3. A pipeline felépítése és az új védelmi funkciók

A pipeline-t úgy alakítottam ki, hogy modellezze az ipari környezetben elvárt biztonsági szűrőket. Két egymásra épülő fő fázist (Job) hoztam létre:

### A) Validációs fázis (Code Validation)
Mielőtt a kódom kikerülne az internetre, a virtuális gép két szigorú szűrőn futtatja át a feltöltést:
* **Commit üzenet formátum ellenőrzése (Commit Linting):** A rendszer egy reguláris kifejezéssel (Regex) ellenőrzi az utolsó commit szövegét. Elvárja a nemzetközi *Conventional Commits* szabványt (pl. `feat:`, `fix:`, `docs:` kezdetű üzenetek). Ha csak annyit írok be, hogy *"kész"* vagy *"módosítás"*, a gép megszakítja a folyamatot és hibát ad.
* **HTML Szintaktikai ellenőrzés (HTML Linting):** A gép a `html-validate` motor segítségével átvizsgálja az `index.html` tartalmát. Ha lezáratlan tag-et, elgépelt attribútumot vagy nem szabványos szerkezetet talál, a pipeline elhasal.

### B) Deployment fázis (Deployment)
Ez a fázis szigorú függőségben van az elsővel (`needs: test`), tehát csak akkor indul el, ha a tesztek hibátlanul lefutottak.
* **Védelem a Pull Requestek ellen:** Beépítettem az `if: github.event_name == 'push'` szabályt. Ez azt jelenti, hogy ha egy teszt ágról nyitok egy beolvasztási kérelmet (Pull Request) a main-re, a tesztek lefutnak, de az éles weboldal még nem frissül. Az oldal csak és kizárólag akkor módosul, ha a kód ténylegesen beolvad és pusholódik a main ágra.

---

## 4. Hogyan működik a weboldal szerver és domain név nélkül?

A feladat során nem kellett saját szervert bérelnem vagy domaint vásárolnom, az oldal mégis elérhető bárki számára az interneten a **GitHub Pages** technológiának köszönhetően.

* **Statikus Webhoszting:** Mivel a projektem egy egyszerű statikus oldal (HTML/CSS), nincs szüksége háttérben futó adatbázis-motorra vagy szerveroldali nyelvre. A fájlok közvetlenül, változtatás nélkül kiszolgálhatók a tárhelyről.
* **DNS és URL Maszkolás:** A GitHub üzemeltet egy globális gyűjtődomaint (`github.io`). Amikor a CD pipeline sikeresen lefut, a GitHub belső DNS kiszolgálói automatikusan leképeznek egy egyedi webcímet a projektemhez a következő séma alapján:  
  `https://<github-felhasználónév>.github.io/<repó-neve>/`
* **CDN (Content Delivery Network):** A fájljaim nem egyetlen merevlemezen ülnek, hanem a GitHub szétmásolja őket a világ különböző pontjain található gyorsítótár-szervereire (CDN). Így ha valaki megnyitja az oldalt, a hozzá legközelebbi szerver fogja kiszolgálni azt, maximális sebességgel és biztonságos, titkosított HTTPS protokollon keresztül.

---

## 5. Implementációs eredmények és screenshotok

A működést igazoló képernyőképeket a `Dokumentacio` mappába mentettem el:

### Pipeline futási eredmény (GitHub Actions)
A sikeresen lefutott pipeline folyamat, amely mutatja a Validációs és Deployment lépéseimet is:

![GitHub Actions sikeres futás (01_actions_success.png)](Dokumentacio/01_actions_success.png)

### GitHub Pages konfiguráció
A GitHub Pages szolgáltatás beállításai és az aktív, élő URL címem:

![GitHub Pages beállítások (02_pages_config.png)](Dokumentacio/02_pages_config.png)

### Élő, publikus weboldal
A felhőből kiszolgált, böngészőben ellenőrzött éles weboldal, rajta az adataimmal:

![Éles weboldal (03_live_site.png)](Dokumentacio/03_live_site.png)
