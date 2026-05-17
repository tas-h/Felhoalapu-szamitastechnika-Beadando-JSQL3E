# Szakmai Dokumentáció: GitHub Actions & Pages CI/CD Pipeline
**Készítette:** Henczes Gábor Tas (JSQL3E)

## 1. Elméleti háttér: Mi az a CI/CD?
A **CI/CD** a modern DevOps módszertan alapköve, ami a szoftverek kiadásának automatizálására szolgál.
* **Folyamatos Integráció (CI):** A fejlesztők kódmódosításai automatikusan beépülnek a központi ágba. Ekkor automatikus tesztek futnak le, kiszűrve a hibákat még a korai fázisban.
* **Folyamatos Kézbesítés/Élesítés (CD):** A sikeresen integrált és ellenőrzött kódot a rendszer automatikusan kiküldi az éles felhőalapú környezetbe (jelen esetben a GitHub Pages statikus CDN hálózatára).

## 2. A GitHub Actions működési elve
A GitHub Actions lehetővé teszi, hogy eseményvezérelt (Event-driven) munkafolyamatokat (Workflows) hozzunk létre.
1. **Event (Esemény):** A folyamatunk indítója (`on: push`), ami figyeli a `main` branchen történő változásokat.
2. **Runner (Futtató környezet):** A GitHub biztosít egy izolált virtuális gépet (`ubuntu-latest`), amin a feladataink végrehajtódnak.
3. **Permissions & Security:** A felhőbiztonság érdekében a pipeline OpenID Connect (OIDC) tokent használ (`id-token: write`), így nem kell fix hozzáférési jelszavakat tárolni a rendszerben. A `contents: read` biztosítja a minimális jogosultság elvét.

## 3. Megvalósítás lépései és futási eredmények
*(Ide fognak kerülni a screenshotok a következő lépés után)*

### Szükséges képek helye:
* **Pipeline futási eredmény az Actions fülön:** `[Screenshot: 01_actions_success.png]`
* **A beállított GitHub Pages környezet:** `[Screenshot: 02_pages_config.png]`
* **Az élő, felhőben futó weboldal:** `[Screenshot: 03_live_site.png]`