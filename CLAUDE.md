# Składki Klasowe — kontekst projektu dla Claude Code

## Czym jest ten projekt

Aplikacja webowa dla **Marthy** — skarbnika klasy 1 (rok szkolny 2025-2026, 23 dzieci).
Pozwala rodzicom samodzielnie sprawdzać stan składek swojego dziecka, a Marcie zarządzać zbiorkami, płatnościami i uczniami.

Hosting: **GitHub Pages** (pliki statyczne, bez serwera).
Backend: **Firebase** (Firestore + Google Auth).
Frontend: vanilla JS + Tailwind CSS CDN, zero frameworków, zero bundlera.

---

## Stack techniczny

- Firebase 10.12.2 (compat SDK: `firebase-app-compat`, `firebase-auth-compat`, `firebase-firestore-compat`)
- Google Sign-In (OAuth popup)
- Tailwind CSS via CDN
- Vanilla JavaScript (ES2020+)
- GitHub Pages (static hosting)

---

## Struktura plików

```
Skarbnik/
├── firebase-config.js   # konfiguracja Firebase (wypełniona przez użytkownika)
├── index.html           # strona logowania + flow rejestracji rodzica
├── parent.html          # panel rodzica (widok składek swojego dziecka)
├── admin.html           # panel skarbnika (macierz płatności, zarządzanie)
├── firestore.rules      # reguły bezpieczeństwa Firestore
├── INSTRUKCJA.md        # instrukcja konfiguracji po polsku
└── CLAUDE.md            # ten plik
```

---

## Kolekcje Firestore

### `students/{id}`
```
{
  firstName: string,
  lastName: string,
  parentEmails: string[],   // adresy Google, po których rodzic jest identyfikowany
  parentName: string,
  parentPhone: string,
  enrolledFrom: string,     // data ISO
  active: boolean,
  createdAt: Timestamp
}
```

### `contributions/{id}`
```
{
  name: string,             // np. "Artykuły klasowe"
  transferTitle: string,    // tytuł przelewu widoczny dla rodzica
  description: string,
  amount: number,           // zł
  dueDate: Timestamp,
  schoolYear: string,       // np. "2025-2026"
  active: boolean,
  createdAt: Timestamp
}
```

### `payments/{studentId}_{contributionId}`
```
{
  studentId: string,
  contribId: string,
  schoolYear: string,
  paid: boolean,
  paidDate: Timestamp | null,
  note: string,
  amount: number,
  updatedAt: Timestamp
}
```
> Klucz dokumentu to zawsze `${studentId}_${contributionId}`.
> Istnienie dokumentu = uczeń jest uczestnikiem składki.
> `paid: false` = w składce, ale nie zapłacił.

### `requests/{uid}`
```
{
  uid: string,
  email: string,
  childFirstName: string,
  childLastName: string,
  status: 'pending' | 'approved' | 'rejected',
  studentId: string,         // uzupełniane przy zatwierdzeniu
  createdAt: Timestamp,
  approvedAt: Timestamp
}
```

### `users/{uid}`
```
{
  isAdmin: boolean           // true tylko dla Marthy
}
```

### `classes/{id}`
```
{
  label: string,             // np. "Kl.1" (może być wyliczone automatycznie)
  schoolYear: string,        // np. "2025-2026"
  createdAt: Timestamp
}
```
> Kolekcja `classes` przechowuje roczniki. Zakładki lat w panelu admina są budowane z tej kolekcji.
> Przy pierwszym uruchomieniu Martha musi dodać klasę przez ⚙️ → zakładka Klasa → formularz.

---

## Stałe (w admin.html i parent.html)

```js
const STARTING_YEAR  = "2025-2026";  // Kl.1 = ten rok szkolny
const STARTING_CLASS = 1;
```

Funkcja `getClassLabel(schoolYear)` wylicza etykietę klasy na podstawie różnicy lat od `STARTING_YEAR`.

---

## Główne przepływy biznesowe

### Flow logowania (index.html)
1. Klik „Zaloguj przez Google" → popup OAuth
2. `onAuthStateChanged`:
   - Admin (`users/{uid}.isAdmin === true`) → redirect do `admin.html`
   - Zatwierdzony rodzic (`students` z `parentEmails` zawierającym email) → redirect do `parent.html`
   - Wniosek w `requests/{uid}`:
     - `pending` → widok oczekiwania (pendingView)
     - `approved` → reload (synchronizacja)
     - `rejected` → widok odrzucenia
   - Brak wniosku → formularz rejestracji (registerView)

### Rejestracja rodzica (samoobsługa)
1. Rodzic podaje imię i nazwisko dziecka → tworzy dokument `requests/{uid}` z `status: 'pending'`
2. Martha widzi wniosek w panelu ⚙️ → zakładka Uczniowie → sekcja „Oczekujące wnioski"
3. Martha klika „Zatwierdź" → wybiera ucznia z listy → batch write:
   - `arrayUnion(email)` do `students/{id}.parentEmails`
   - `status: 'approved'` do `requests/{uid}`

### Panel rodzica (parent.html)
- Wyszukuje ucznia po `parentEmails array-contains user.email`
- Zakładki lat budowane z kolekcji płatności tego ucznia (distinct schoolYears)
- Karty składek: zielona = OPŁACONA, czerwona = ZALEGA

### Panel admina (admin.html)

**Stan globalny `G`:**
```js
let G = {
  year: currentSchoolYear(),    // aktywny rok szkolny
  view: 'main',                 // 'main' | 'newContrib' | 'settings'
  contribId: 'ALL',             // wybrany chip: 'ALL' lub ID składki
  edit: false,                  // tryb edycji w widoku składki
  settingsTab: 'uczniowie',     // 'uczniowie' | 'klasa'
  students: [],
  contribs: [],
  payments: {},                 // key: studentId_contribId
  classes: [],
  pendingReqs: [],
  pendingDel: null,
  pendingApprove: null
};
```

**Widoki:**
- `viewMain` — chipsy zbiórek + tabela/macierz płatności
- `viewNewContrib` — formularz nowej składki
- `viewSettings` — ustawienia z dwoma zakładkami:
  - 👥 Uczniowie: oczekujące wnioski + tabela uczniów + formularz dodawania
  - 🏫 Klasa: lista klas + formularz dodawania nowej klasy/rocznika

**Funkcje kluczowe:**
- `loadData()` — ładuje wszystko równolegle (Promise.all), wywołuje `buildYearTabs()` i `showView()`
- `buildYearTabs()` — buduje zakładki z `G.classes` (jeśli pusta kolekcja, pokazuje bieżący rok)
- `showView(v)` — przełącza widoki, wywołuje render dla danego widoku
- `renderMain()` → `renderChips()` + `renderMatrix()` lub `renderContribDetail()`
- `renderSettings()` → `switchSettingsTab(G.settingsTab)`
- `switchSettingsTab(tab)` — przełącza między zakładkami Uczniowie/Klasa
- `renderClassesTab()` — lista klas z oznaczeniem aktywnej
- `openPmtModal(sId, cId, currentlyPaid)` — modal oznaczania płatności
- `openApproveModal(reqId)` — modal zatwierdzania wniosku rodzica

---

## Reguły bezpieczeństwa Firestore

| Kolekcja      | Odczyt                         | Zapis         |
|---------------|--------------------------------|---------------|
| users         | właściciel (uid)               | tylko admin   |
| students      | admin lub własny rodzic        | tylko admin   |
| contributions | każdy zalogowany               | tylko admin   |
| payments      | admin lub rodzic właściciela   | tylko admin   |
| requests      | właściciel lub admin           | create: właściciel; update/delete: admin |
| classes       | każdy zalogowany               | tylko admin   |

Admin = `users/{uid}.isAdmin === true`

---

## Co jest zrobione

- [x] Logowanie Google + auto-redirect (admin/rodzic/nowy)
- [x] Samoregistracja rodziców z przepływem zatwierdzania przez admina
- [x] Panel rodzica: karty składek (zielone/czerwone), zakładki lat, podsumowanie
- [x] Panel admina:
  - [x] Macierz płatności (ALL)
  - [x] Widok per-składka z tabelą uczniów
  - [x] Chipsy zbiórek (kolor = status płatności)
  - [x] Tryb EDYTUJ/ZAPISZ (dodawanie/usuwanie uczniów ze składki)
  - [x] Modal oznaczania płatności (z datą i notatką)
  - [x] Formularz nowej składki (z checklistą uczniów)
  - [x] Ustawienia → Uczniowie: tabela + dodawanie/usuwanie uczniów
  - [x] Ustawienia → Klasa: lista klas + dodawanie nowych roczników
  - [x] Zatwierdzanie wniosków rodziców (z auto-podpowiedzi ucznia)
- [x] Reguły Firestore dla wszystkich kolekcji
- [x] Instrukcja konfiguracji (INSTRUKCJA.md)

## Co można jeszcze rozwinąć (potencjalne zadania)

- [ ] Odrzucanie wniosków rodziców (przycisk „Odrzuć" obok „Zatwierdź")
- [ ] Eksport do CSV / wydruk listy płatności
- [ ] Powiadomienia email do rodziców o nowej składce
- [ ] Edycja istniejącej składki (zmiana kwoty, terminu)
- [ ] Historia zmian płatności (kto i kiedy oznaczył)
- [ ] Widok zbiorczy przez wszystkie lata (dla rodziców — zakładki Kl.1, Kl.2...)

---

## Uwagi developerskie

- `firebase-config.js` NIE jest w repozytorium (zawiera klucze) — użytkownik wypełnia go ręcznie
- Composite ID dla payments: zawsze `${studentId}_${contributionId}` — nigdy nie używaj `add()`, używaj `set()`
- `schoolYear` to string w formacie `"RRRR-RRRR"` — używany jako klucz wszędzie
- Martha ma `active: false` zamiast fizycznego usuwania dokumentów (soft delete)
- `parentEmails` to tablica — jeden uczeń może mieć kilka kont rodziców
- Przy zatwierdzaniu rodzica: użyj `arrayUnion` — nie nadpisuj całej tablicy
