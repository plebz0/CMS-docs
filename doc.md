# Mini-CMS — dokumentacja
 ## 1. Ogólny opis

**Mini-CMS** to projekt napisany w **Node.js** z użyciem frameworka **Express.js** i szablonów **EJS**.  

System nie korzysta z bazy danych — artykuły są zapisywane w pliku `.json`.  
Całość jest podzielona na kilka warstw, co ułatwia przejżystość i dalszy rozrós kodu:

- **Models** – odpowiadają za zapis i odczyt danych z pliku  
- **Services** – przetwarzają dane i łączą modele z logiką aplikacji  
- **Controllers** – obsługują żądania użytkownika (np. wyświetlenie artykułów)  
- **Routes** – definiują adresy URL i przekazują żądania do kontrolerów  
- **Views (EJS)** – generują strony HTML, które widzi użytkownik  

Aplikacja pozwala na:
- przeglądanie listy artykułów,  
- wyświetlenie pojedynczego artykułu po jego *slug’u*,  
- dodawanie nowych artykułów przez prosty formularz,  
- obsługę błędów, jeśli artykuł nie istnieje.

## 2. Schemat zależności


- **routes** używa controllers (każda trasa korzysta z funkcji kontrolera)

- **controllers** używa **services**  (kontrolery wywołują logikę biznesową)

- **services** używa **models**  (serwisy pobierają i zapisują dane przez model)

- **controllers** renderują **views** (na końcu zwracają gotową stronę EJS użytkownikowi)

A wszystko startuje od **server.js**, który uruchamia całą aplikację.

## 3. Główne moduły aplikacji

---

###  `server.js`
**Ścieżka pliku:**  
`/server.js`

**Rola w aplikacji:**  
Uruchamia serwer Express i łączy wszystkie elementy aplikacji.  

---


**Używane funkcje:**  
- `require()` – import modułów  
- `app.listen()` – uruchamia serwer na porcie  
---


**Eksportowane elementy:**  
Brak – uruchamiany bezpośrednio.
---


**Przykład użycia:**  
```js
const app = require('./app');
const PORT = 3000;
app.listen(PORT, () => console.log(`Serwer działa na porcie ${PORT}`));
```

### `app.js`

**Ścieżka pliku:**  
`/app.js`

**Rola w aplikacji:**  
Odpowiada za główną konfigurację aplikacji Express.  
Ustawia silnik szablonów EJS, dodaje obsługę plików statycznych z folderu `public`  
oraz rejestruje trasy z `routes/articles.js`.  
Jest centralnym punktem, który łączy wszystkie moduły.
---


**Używane funkcje:**  
- `express()` – tworzy instancję aplikacji  
- `app.set()` – konfiguracja silnika widoków  
- `app.use()` – rejestracja middleware i tras  
- `path.join()` – określa ścieżki do folderów  
---


**Eksportowane elementy:**  
- `module.exports = app;` – eksport instancji Express, którą uruchamia `server.js`

- `module.exports = app;` – eksport instancji Express, którą uruchamia `server.js`
---


**Przykład użycia:**  
```js
const express = require('express');
const app = express();
const path = require('path');
const articleRoutes = require('./routes/articles');
```


### `controllers/articlesController.js`

**Ścieżka pliku:**  
`/controllers/articlesController.js`

**Rola w aplikacji:**  

Odbiera żądania z tras (`routes/articles.js`), korzysta z serwisu (`services/articleService.js`)  
do pobierania lub tworzenia danych, a następnie renderuje odpowiednie widoki (`views/articles/...`).  

Odpowiada za wyświetlanie listy artykułów, pojedynczego wpisu, tworzenie nowego artykułu  
oraz obsługę błędów walidacji.

---


**Używane funkcje:**  
- `articleService.getAllArticles()` – pobiera wszystkie artykuły  
- `articleService.getArticlesBySlug(slug)` – pobiera artykuł po jego slug  
- `articleService.createArticle(data)` – tworzy nowy artykuł  
- `res.render()` – renderuje widok EJS  
- `res.redirect()` – przekierowuje po zapisaniu artykułu  
- `res.status()` – ustawia kod odpowiedzi HTTP  

---


**Eksportowane elementy:**  
- `index(req, res)` – renderuje listę artykułów  
- `show(req, res)` – pokazuje pojedynczy artykuł na podstawie slug  
- `newForm(req, res)` – wyświetla formularz dodawania nowego artykułu  
- `create(req, res)` – tworzy nowy artykuł i waliduje dane  


---

**Przykład użycia:**  
```js
const articleService = require('../services/articleService');
```


### `models/storage.js`

**Ścieżka pliku:**  
`/models/storage.js`

**Rola w aplikacji:**  
Ten moduł pełni funkcję prostego magazynu danych — działa jak mini baza danych w pliku JSON.  
Odpowiada za odczytywanie i zapisywanie artykułów do pliku `data/articles.json`.  
Wykorzystuje wbudowany moduł `fs` do operacji na plikach i zapewnia prosty interfejs do użycia w serwisach.

---

**Używane funkcje:**  
- `fs.readFileSync()` – odczytuje dane z pliku JSON  
- `fs.writeFileSync()` – zapisuje dane do pliku JSON  
- `path.join()` – łączy ścieżki w systemie plików  
- `JSON.parse()` – zamienia dane tekstowe na obiekt JavaScript  
- `JSON.stringify()` – konwertuje dane z powrotem do formatu JSON  

---

**Eksportowane elementy:**  
- `readArticles()` – odczytuje dane z pliku JSON i zwraca je jako tablicę obiektów
- `writeArticles()` – zapisuje tablicę obiektów do pliku JSON 

---

**Przykład użycia:**  
```js
const fs = require('fs');
const path = require('path');
const DATA_PATH = path.join(__dirname, '..', '..', 'data', 'articles.json');
async function readArticles() {
    const data = await fs.readFile(DATA_PATH, 'utf-8');
    return JSON.parse(data || '[]');
}

async function writeArticles(articles) {
    await fs.writeFile(DATA_PATH, JSON.stringify(articles, null, 4), 'utf-8');
}
```

### `routes/articles.js`

**Ścieżka pliku:**  
`/routes/articles.js`

**Rola w aplikacji:**  
Definiuje wszystkie trasy związane z artykułami (`/articles`, `/articles/:slug`, `/articles/new`) i przekazuje obsługę żądań do funkcji w kontrolerze `articlesController.js`.

**Używane funkcje:**  
- `express.Router()` – tworzy router dla grupy tras  
- `router.get()` – obsługa tras GET  
- `router.post()` – obsługa tras POST  
- funkcje z `articlesController` – np. `index()`

**Eksportowane elementy:**  
`module.exports = router;`

**Przykład użycia:**  
```js
const router = require('express').Router();
const articlesController = require('../controllers/articlesController');

router.get('/', articlesController.index);        
router.get('/new', articlesController.newForm);   
router.get('/:slug', articlesController.show);    
router.post('/', articlesController.create);      

module.exports = router;
```

### `articleService.js`
**Ścieżka pliku:**  
`/services/articleService.js`

**Rola w aplikacji:**  
Łączy kontrolery z modelem `storage.js`.  
Przetwarza dane artykułów, tworzy nowe wpisy i zwraca informacje potrzebne kontrolerom.

**Używane funkcje:**  
- `getAllArticles()` – pobiera wszystkie artykuły z modelu  
- `getArticlesBySlug(slug)` – pobiera artykuł po jego `slug`  
- `createArticle(data)` – tworzy nowy artykuł, generuje slug i zapisuje przez model

**Eksportowane elementy:**  
- `getAllArticles`  
- `getArticlesBySlug`  
- `createArticle`



## 4. Opis wyglądu

### `views/articles/list.ejs`
**Opis:**  
Wyświetla listę wszystkich artykułów w formie prostego spisu tytułów z linkami.  
Każdy tytuł prowadzi do osobnej podstrony artykułu.  

**Zawartość:**  
- Nagłówek z tytułem strony  
- Pętla `forEach` renderująca listę artykułów  
- Linki do `/articles/:slug`  


### `views/articles/show.ejs`
**Opis:**  
Wyświetla pojedynczy artykuł – jego tytuł, treść i autora.
To widok, który użytkownik widzi po kliknięciu w tytuł z listy.

**Zawartość:**  

- Nagłówek z tytułem artykułu

- Paragraf z treścią

- Podpis z autorem



**Zawartość:**

- Formularz z metodą POST

- Pola input i textarea

- Obsługa błędów (np. walidacja tytułu i treści)

**Wygląd:**
Czysty formularz z trzema polami i prostym przyciskiem.
W przypadku błędów, komunikaty są wyświetlane nad formularzem.

### `views/articles/error.ejs`

**Opis:**
Widok odpowiedzialny za wyświetlanie komunikatu błędu — np. gdy użytkownik próbuje otworzyć artykuł, który nie istnieje.
Pokazuje prosty komunikat o braku strony lub danych.

**Zawartość:**

-Nagłówek z tytułem błędu (np. „Nie znaleziono”)

-Krótka informacja dla użytkownika




### `views/articles/new.ejs`

**Opis:**
Formularz do tworzenia nowego artykułu.
Pozwala użytkownikowi wpisać tytuł, treść oraz autora, a następnie zapisać artykuł w systemie.
Obsługuje również błędy walidacji (np. zbyt krótki tytuł lub treść).

**Zawartość:**

- Formularz z metodą POST wysyłający dane na /articles

- Pola input (tytuł, autor) i textarea (treść artykułu)

- Wyświetlanie komunikatów błędów w przypadku niepoprawnych danych

- Przycisk Zapisz do przesłania formularza


### `views/partials/header.ejs`
**Opis:**
Część wspólna wszystkich stron – nagłówek z tytułem aplikacji i prostym menu.
Dołączany w innych widokach za pomocą include.

**Zawartość:**

- Tytuł strony

- Nawigacja z linkami


### `views/partials/footer.ejs`

**Opis:**
Część wspólna wszystkich stron — stopka aplikacji, dołączana na końcu każdej podstrony za pomocą include.
Zawiera proste informacje o autorze lub prawach autorskich.

**Zawartość:**

- Tekst ze stopką, np. nazwa aplikacji lub informacja o autorze

- Może zawierać linki do dodatkowych podstron (np. kontakt, o nas)



## `public/css/style.css`
**Opis:**
Plik stylów odpowiada za wygląd aplikacji.
Nadaje podstawowe kolory, marginesy i czcionki, zachowując minimalistyczny charakter.

**Wygląd:**
Jasne tło
Ciemny tekst

Koniec - Dawid Kubik