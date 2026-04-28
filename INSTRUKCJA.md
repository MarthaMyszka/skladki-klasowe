# 📋 Instrukcja konfiguracji – Składki Klasowe

## Co dostaniesz po konfiguracji?

- **Strona logowania** – każdy rodzic loguje się swoim kontem Google
- **Panel rodzica** – widzi tylko składki swojego dziecka z bieżącego roku
- **Panel skarbnika** – Ty widzisz wszystko: macierz płatności, zarządzanie uczniami i składkami
- **Bezpieczeństwo** – dane w Firebase, rodzice nie mogą widzieć danych innych dzieci

---

## KROK 1 – Utwórz projekt Firebase (5 min)

1. Wejdź na [console.firebase.google.com](https://console.firebase.google.com)
2. Kliknij **„Dodaj projekt"** → wpisz nazwę np. `skladki-klasowe` → kliknij dalej
3. Możesz wyłączyć Google Analytics (nieobowiązkowe) → **Utwórz projekt**

---

## KROK 2 – Włącz logowanie przez Google (2 min)

1. W bocznym menu wybierz **Authentication** → **Sign-in method**
2. Kliknij **Google** → włącz → podaj e-mail kontaktowy → **Zapisz**

---

## KROK 3 – Utwórz bazę danych Firestore (3 min)

1. W bocznym menu wybierz **Firestore Database** → **Utwórz bazę danych**
2. Wybierz tryb: **Tryb produkcyjny** → **Dalej**
3. Wybierz lokalizację: **europe-west1 (Belgium)** → **Gotowe**

### Wgraj reguły bezpieczeństwa:

1. W Firestore przejdź do zakładki **Reguły**
2. Zaznacz wszystko i wklej zawartość pliku `firestore.rules` (otwórz go w Notatniku)
3. Kliknij **Opublikuj**

---

## KROK 4 – Dodaj aplikację webową i skopiuj konfigurację (3 min)

1. W bocznym menu kliknij ikonę koła zębatego → **Ustawienia projektu**
2. Przewiń do sekcji **„Twoje aplikacje"** → kliknij **</>** (Web)
3. Wpisz nazwę aplikacji → kliknij **Zarejestruj aplikację**
4. Zobaczysz blok kodu z `firebaseConfig`. Skopiuj wartości i wklej do pliku `firebase-config.js`:

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "skladki-klasowe.firebaseapp.com",
  projectId:         "skladki-klasowe",
  storageBucket:     "skladki-klasowe.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abc123"
};
```

---

## KROK 5 – Nadaj sobie uprawnienia skarbnika (2 min)

Musisz jednorazowo dodać swoje konto jako admina w Firestore.

1. Wejdź na [console.firebase.google.com](https://console.firebase.google.com) → wybierz projekt
2. Po lewej kliknij **Authentication** → zakładka **Użytkownicy**
3. **Zaloguj się raz na stronie** (np. otwórz `index.html`) → wróć do Authentication
4. Skopiuj swoje **UID** (długi ciąg znaków obok e-maila)
5. Wróć do **Firestore Database** → kliknij **Rozpocznij kolekcję**
   - ID kolekcji: `users`
   - ID dokumentu: wklej swoje UID
   - Dodaj pole: `isAdmin` → Typ: `boolean` → Wartość: `true`
6. Kliknij **Zapisz**

Od teraz po zalogowaniu na stronie automatycznie trafisz do panelu skarbnika.

---

## KROK 6 – Opublikuj stronę na GitHub Pages (10 min)

### 6a. Utwórz konto na GitHub (jeśli nie masz)
Wejdź na [github.com](https://github.com) → **Sign up** → zarejestruj się

### 6b. Utwórz repozytorium
1. Kliknij **+** (prawy górny róg) → **New repository**
2. Nazwa: `skladki-klasowe`
3. Ustaw na **Public** (wymagane dla darmowego GitHub Pages)
4. Kliknij **Create repository**

### 6c. Wgraj pliki
1. Na stronie repozytorium kliknij **uploading an existing file**
2. Przeciągnij wszystkie pliki (oprócz `firestore.rules` i `INSTRUKCJA.md`):
   - `firebase-config.js`
   - `index.html`
   - `parent.html`
   - `admin.html`
3. Kliknij **Commit changes**

### 6d. Włącz GitHub Pages
1. Przejdź do **Settings** → **Pages** (w bocznym menu)
2. W sekcji „Branch" wybierz **main** → **/root** → **Save**
3. Po chwili pojawi się link do strony, np.:
   `https://twoj-login.github.io/skladki-klasowe/`

### 6e. Dodaj domenę do Firebase (wymagane!)
1. Wróć do **Firebase Console** → **Authentication** → **Ustawienia** → **Autoryzowane domeny**
2. Kliknij **Dodaj domenę** → wpisz `twoj-login.github.io` → **Dodaj**

---

## KROK 7 – Skonfiguruj pierwszą klasę, uczniów i składkę

### 7a. Dodaj pierwszą klasę (wymagane!)

Zanim dodasz uczniów lub składki, musisz utworzyć klasę w systemie.

1. Otwórz stronę → zaloguj się → trafisz do **Panelu Skarbnika**
2. Kliknij ikonę ⚙️ (prawy górny róg) → otworzą się Ustawienia
3. Przejdź do zakładki **🏫 Klasa**
4. W polu **„Rok szkolny"** wpisz rok w formacie `2025-2026`
5. Pole „Nazwa" możesz zostawić puste – zostanie wyliczona automatycznie (np. „Kl.1")
6. Kliknij **+ Dodaj klasę**

Po dodaniu klasy pojawi się w górnym pasku jako zakładka. Kliknij ją, żeby ją aktywować.

> Każdy kolejny rok szkolny (awans do klasy 2, 3...) dodajesz tak samo –  
> nowa zakładka pojawi się automatycznie w panelu.

### 7b. Dodaj uczniów

1. W Ustawieniach przejdź do zakładki **👥 Uczniowie** → kliknij **EDYTUJ**
2. Wypełnij formularz na dole: imię, nazwisko, e-mail rodzica (konto Google)
3. Kliknij **+ Dodaj ucznia** – powtórz dla każdego dziecka (23 uczniów)
4. Kliknij **ZAPISZ**

### 7c. Dodaj pierwszą składkę

1. Kliknij **+** (niebieski przycisk w prawym górnym rogu)
2. Wypełnij: nazwa składki, tytuł przelewu, kwota, termin
3. Zaznacz uczniów objętych składką (domyślnie wszyscy)
4. Kliknij **Zapisz**

W panelu głównym zobaczysz macierz płatności – klikaj komórki, żeby oznaczać wpłaty.

---

## Jak rodzice korzystają ze strony?

### Opcja A – rodzic dodany ręcznie przez Ciebie
1. Wyślij im link do strony (np. na czacie klasowym)
2. Klikają **„Zaloguj się przez Google"** swoim kontem Gmail
3. Widzą od razu dane swojego dziecka

> ⚠️ E-mail wpisany w systemie musi być **dokładnie tym samym** kontem Google, którym się logują.

### Opcja B – samorejestracja rodzica
1. Rodzic wchodzi na stronę i loguje się Google
2. Widzi formularz – wpisuje imię i nazwisko dziecka → klika **„Wyślij wniosek"**
3. **Ty** dostajesz powiadomienie w Panelu Skarbnika → ⚙️ → zakładka **Uczniowie**  
   (sekcja „Oczekujące wnioski" pojawi się na górze)
4. Klikasz **Zatwierdź** → wybierasz ucznia z listy → akceptujesz
5. Rodzic może teraz wejść na stronę i zobaczyć składki swojego dziecka

---

## Pytania? Problemy?

- Upewnij się, że `firebase-config.js` ma wypełnione wszystkie wartości
- Sprawdź, czy domena GitHub Pages jest dodana w Firebase Authentication
- Upewnij się, że reguły Firestore zostały wgrane i opublikowane
