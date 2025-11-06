# Dokument wymagań produktu (PRD) - DreamInsight

## 1. Przegląd produktu

DreamInsight to webowa aplikacja do journalingu snów wspierana przez AI. Umożliwia użytkownikom bezpieczne zapisywanie snów, tagowanie emocji i symboli, generowanie interpretacji przez model AI na żądanie oraz przeglądanie prostych wizualizacji trendów (dashboard). MVP ma być minimalistyczne, w domyślnym ciemnym motywie, z autoryzacją przez Supabase Auth i zapisem danych w Supabase DB. Interpretacje AI generowane są manualnie (na żądanie) i zapisywane w bazie, aby uniknąć ponownych kosztów zapytań.

Główne cele projektu:

- Szybkie i bezpieczne dodawanie wpisów snów (CRUD).
- AI-driven interpretacje snów zapisywane jako część wpisu.
- Dashboard z agregacjami emocji i tagów za okresy 7/30 dni.
- Prosty przepływ użytkownika: rejestracja → lista snów → dodaj sen → poproś o interpretację → oceń.
- Przygotowanie do E2E testu (login → add dream → interpret → rate) oraz deploymentu na Vercel.

## 2. Problem użytkownika

Użytkownicy, którzy chcą prowadzić dziennik snów, napotykają następujące problemy:

- Chaotyczność i brak systematyczności: brak przypomnień i łatwe zapomnienie szczegółów snów.
- Brak narzędzi do analizy: trudno dostrzec wzorce w emocjach, symbolach i trendach.
- Trudność w interpretacji: użytkownicy potrzebują sugestii i kontekstu, które mogą pomóc w refleksji.
- Koszty i prywatność: użytkownicy chcą, aby interpretacje były przechowywane lokalnie w ich profilu, a nie generowane za każdym odczytem.

DreamInsight rozwiązuje te problemy przez: prosty formularz wpisu, przypomnienia, interpretacje AI generowane na żądanie i zapisane w DB, oraz dashboard ułatwiający obserwację trendów.

## 3. Wymagania funkcjonalne

3.1 Autoryzacja i zarządzanie kontem

- Rejestracja i logowanie przez Supabase Auth (email + password).
- Resetowanie hasła (supabase built-in flow).
- Wylogowanie.
- Profile użytkownika (opcjonalne pola, ustawienia przypomnienia).

  3.2 CRUD wpisów snów

- Formularz dodawania snu: data, tytuł (opcjonalny), opis (limit 500–1000 znaków), 1–3 emocje (wybrane z listy), 1–5 tagów (wybrane z listy).
- Walidacja po stronie klienta: wymagane pola, limity opisów, limity ilości emocji/tagów.
- Lista snów sortowana malejąco po dacie (najnowsze na górze).
- Edycja i usuwanie wpisów (ikonki edit/delete bezpośrednio na liście).
- Soft cap: informacja dla użytkownika o limicie 30 wpisów; limit nie blokuje (można dalej dodawać), ale powinien wyświetlać komunikat ostrzegawczy.

  3.3 AI: generowanie interpretacji

- Użytkownik może na żądanie wygenerować interpretację snu (przycisk "Poproś o interpretację" lub "Interpretuj").
- Model: OpenAI — wywołanie synchroniczne z widocznym loaderem.
- Wynik interpretacji zapisany w kolumnie `interpretation` tabeli `dreams` w DB; interpretacja nie jest generowana ponownie przy odczycie.
- Możliwość ponownego wygenerowania interpretacji przez użytkownika (nadpisuje istniejącą interpretację) — wymaga potwierdzenia.

  3.4 Ocena interpretacji

- Po wygenerowaniu interpretacji użytkownik może ocenić ją w skali 1–5 gwiazdek.
- Ocena zapisywana jest jako część rekordu interpretacji (lub tabela `interpretation_ratings`).

  3.5 Sugestie emocji i tagów

- Przycisk "Zaproponuj tagi/emocje" analizuje opis snu i proponuje 1–3 emocje i 1–5 tagów.
- Propozycje generowane lokalnie po stronie frontu lub z użyciem API AI (do decyzji implementacyjnej); wyniki edytowalne przez użytkownika.

  3.6 Dashboard i wizualizacje

- Strona `/dashboard` dostępna po zalogowaniu.
- Wizualizacje: rozkład emocji i tagów w ujęciu 7 dni i 30 dni (słupki/koło/procenty).
- Prostą agregację (liczba wystąpień) wykonuje frontend z danych pobranych z DB.

  3.7 Przypomnienia (opcjonalne)

- Domyślna godzina przypomnienia: 21:00; użytkownik może zmienić godzinę w ustawieniach.
- Powiadomienia realizowane przez service worker (web push) lub e-mail (w MVP: web push lub lokalny przypomnienie — implementacja zależna od infrastruktury).

  3.8 Onboarding (opcjonalne)

- Opcjonalne 3-ekranowe intro: intro → dodaj sen → interpretuj. Oznaczone jako P3 (opcjonalne na później).

  3.9 CI/CD i testy

- Pipeline: lint/test → build → deploy to Vercel.
- Minimum 1 test E2E (Playwright): login → add dream → interpret → rate.

  3.10 Audyt i bezpieczeństwo

- Wszystkie operacje wymagające autoryzacji muszą być zabezpieczone (Supabase RLS / auth checks).
- Interpretacje przechowywane w DB i dostępne tylko dla właściciela rekordu.

## 4. Granice produktu

4.1 Zakres MVP (co obejmuje)

- Rejestracja i logowanie użytkowników (Supabase Auth).
- Pełen CRUD dla pojedynczych wpisów snów.
- Generowanie i zapis interpretacji AI na żądanie.
- Dashboard z podsumoewaniem 7/30 dni.
- Walidacja formularzy i dark theme UI.
- CI/CD i jeden test E2E.

  4.2 Out of scope (nie wchodzi w MVP)

- Przypomnienia i soft cap 30 wpisów.
- Wizualizacje - dashboard
- Zaawansowane modele ML czy custom training.
- Integracje z urządzeniami (wearables) i automatyczne importy snu.
- Funkcje społecznościowe (udostępnianie/dyskusje).
- Eksport (PDF, kalendarz), natywne aplikacje mobilne, voice input.
- Wersjonowanie wpisów (MVP nadpisuje wpisy przy edycji).

  4.3 Ograniczenia i założenia

- Interpretacje AI będą zapisywane w DB i nie będą ponownie generowane przy odczycie.
- Użycie Supabase SDK do zarządzania danymi zamiast własnych API endpointów (upraszcza stos technologiczny).
- Domyślny język interfejsu: do decyzji (PL/EN) — w MVP można ustawić PL.
- Limit zapisu: soft cap 30 wpisów na użytkownika, ale bez mechanicznego blokowania.

## 5. Historyjki użytkowników

Poniżej znajdują się wszystkie historyjki użytkowników niezbędne do zbudowania pełnej aplikacji MVP, włączając scenariusze podstawowe, alternatywne i skrajne. Każda historia ma unikalny ID i kryteria akceptacji.

US-001
Tytuł: Rejestracja nowego użytkownika
Opis: Użytkownik może zarejestrować konto używając adresu e-mail i hasła.
Kryteria akceptacji:

- Formularz rejestracji przyjmuje e-mail i hasło (min 8 znaków).
- Po pomyślnej rejestracji użytkownik otrzymuje sesję zalogowaną.
- Błędy (np. email już użyty, słabe hasło) wyświetlane są użytkownikowi zrozumiale.
- Dane użytkownika tworzone są w Supabase Auth i tabela użytkowników.

US-002
Tytuł: Logowanie użytkownika
Opis: Zarejestrowany użytkownik może się zalogować (email + password).
Kryteria akceptacji:

- Poprawne dane logowania tworzą sesję i przekierowują do listy snów.
- Niepoprawne dane pokazują komunikat o błędnych danych.
- Sesja wygasa zgodnie z konfiguracją Supabase.

US-003
Tytuł: Reset hasła
Opis: Użytkownik może zainicjować reset hasła przez e-mail.
Kryteria akceptacji:

- Możliwość wysłania requestu resetu hasła (Supabase flow).
- Użytkownik otrzymuje potwierdzenie wysłania linku resetującego.

US-004
Tytuł: Wylogowanie
Opis: Użytkownik może się wylogować i zostanie przeniesiony do strony logowania.
Kryteria akceptacji:

- Wylogowanie niszczy lokalną sesję i przekierowuje na stronę logowania.

US-005
Tytuł: Ustawienia przypomnienia (opcjonalne)
Opis: Użytkownik może ustawić godzinę przypomnienia (domyślnie 21:00).
Kryteria akceptacji:

- Ustawienie godziny zapisuje preferencję w profilu użytkownika.
- Domyślna wartość to 21:00 dla nowych kont.
- Zmiana godziny natychmiast wpływa na logikę wysyłania przypomnień (po stronie backend/service worker).

US-006
Tytuł: Widok listy snów
Opis: Użytkownik widzi listę swoich snów posortowaną malejąco po dacie.
Kryteria akceptacji:

- Lista pokazuje tytuł, datę, krótki fragment opisu, emocje i tagi.
- Tylko wpisy zalogowanego użytkownika są widoczne.
- Paginacja lub ładowanie leniwe jeśli > 20 wpisów.

US-007
Tytuł: Dodanie nowego snu
Opis: Użytkownik dodaje nowy wpis snu poprzez formularz.
Kryteria akceptacji:

- Formularz wymaga daty i opisu, ogranicza opis do 500–1000 znaków.
- Emocje: 1–3 wybory; Tagi: 1–5 wybory; klient waliduje limity.
- Po zatwierdzeniu wpis jest zapisany i wyświetlony na liście.
- Gdy użytkownik przekroczy 30 wpisów, wyświetlany jest komunikat soft cap.

US-008
Tytuł: Edycja wpisu snu
Opis: Użytkownik może edytować istniejący wpis i zapisać zmiany.
Kryteria akceptacji:

- Edycja aktualizuje pola: data, tytuł, opis, emocje, tagi.
- Zmiany są nadpisywane w DB; brak wersjonowania.
- Walidacja tych samych ograniczeń co przy tworzeniu.

US-009
Tytuł: Usunięcie wpisu snu
Opis: Użytkownik może usunąć wpis; wymagana jest potwierdzenie.
Kryteria akceptacji:

- Użytkownik widzi modal potwierdzający usunięcie.
- Po potwierdzeniu wpis jest trwale usuwany z DB i z listy.
- Usunięcie możliwe tylko dla właściciela wpisu.

US-010
Tytuł: Prośba o interpretację AI
Opis: Użytkownik może zażądać interpretacji dla danego wpisu.
Kryteria akceptacji:

- Po kliknięciu "Interpretuj" pokazuje się loader; przycisk disabled, by uniknąć duplikatów.
- Wywołanie API AI (gpt-4o-mini) odbywa się synchronicznie; odpowiedź zapisywana w polu `interpretation` rekordu snu.
- Po zapisaniu interpretacji użytkownik widzi wynik i przycisk do oceny.
- Jeśli API zwraca błąd, użytkownik otrzymuje czytelny komunikat i opcję ponownej próby.

US-011
Tytuł: Ponowne wygenerowanie interpretacji
Opis: Użytkownik może ponownie wygenerować interpretację (nadpisanie).
Kryteria akceptacji:

- Akcja wymaga potwierdzenia, ponieważ nadpisuje istniejący tekst.
- Po potwierdzeniu proces jest identyczny jak w US-010 i nadpisuje `interpretation`.

US-012
Tytuł: Ocena interpretacji (opcjonalne)
Opis: Użytkownik ocenia interpretację w skali 1–5.
Kryteria akceptacji:

- Ocena zapisywana jest wraz z identyfikatorem wpisu i użytkownika.
- Użytkownik może zaktualizować swoją ocenę (zmiana zapisuje nową wartość).
- Widoczna jest średnia ocena (jeśli planujemy agregację) — w MVP wystarczy własna ocena.

US-013
Tytuł: Propozycja tagów i emocji (opcjonalne)
Opis: Użytkownik może kliknąć "Zaproponuj tagi/emocje" i otrzymać sugestie.
Kryteria akceptacji:

- System proponuje listę rekomendowanych emocji (1–3) i tagów (1–5) na podstawie opisu.
- Użytkownik może zaakceptować, odrzucić lub edytować proponowane wartości.
- Propozycje są wygenerowane deterministycznie (AI lub prosty parser) i nie zapisują się automatycznie bez potwierdzenia.

US-014
Tytuł: Dashboard — widok 7/30 dni (wizualizacje - opcjonalne)
Opis: Użytkownik ogląda wizualizacje emocji i tagów dla 7 i 30 dni.
Kryteria akceptacji:

- Dashboard dostępny pod `/dashboard` po zalogowaniu.
- Widgety: liczba wpisów, najczęściej występujące emocje, ciąg tagów, wykresy dla 7 dni i 30 dni.
- Dane są poprawnie agregowane z wpisów użytkownika.

US-015
Tytuł: Onboarding (opcjonalne)
Opis: Nowy użytkownik widzi 3-ekranowe wprowadzenie: intro → dodaj sen → interpretuj.
Kryteria akceptacji:

- Onboarding można pominąć i oznaczyć jako ukończony.
- Onboarding nie jest wymuszony i jest P3 (nie blokuje MVP).

US-016
Tytuł: Soft cap 30 wpisów — komunikat
Opis: Po osiągnięciu 30 wpisów użytkownik otrzymuje komunikat informacyjny.
Kryteria akceptacji:

- Komunikat informuje o zalecanym limicie, ale nie blokuje dodawania dalszych wpisów.
- Komunikat pokazuje aktualną liczbę wpisów.

US-017
Tytuł: Ochrona dostępu do zasobów
Opis: Tylko uwierzytelnieni użytkownicy mogą uzyskać dostęp do zapisanych wpisów i interpretacji.
Kryteria akceptacji:

- Nieautoryzowany dostęp redirectuje do strony logowania.
- API/DB używa reguł dostępu (RLS) zapewniających, że dane nie są widoczne dla innych użytkowników.

US-018
Tytuł: E2E: scenariusz krytyczny
Opis: Automatyczny test E2E sprawdza: logowanie → dodanie snu → wygenerowanie interpretacji → ocena.
Kryteria akceptacji:

- Test uruchamia się w CI i przechodzi na przykładowej instancji testowej.
- Test uwzględnia podstawowe assertions: widoczność wpisu, tekst interpretacji, zapis oceny.

US-019
Tytuł: Obsługa błędów AI i retry
Opis: System obsługuje błędy zwrotu z API AI i pozwala użytkownikowi na retry.
Kryteria akceptacji:

- W przypadku błędu AI użytkownik widzi czytelny komunikat i może wybrać ponowną próbę.
- Aplikacja nie zapisuje pustej interpretacji.

US-020
Tytuł: Ustawienia prywatności i eksport danych (informacyjne)
Opis: Użytkownik ma dostęp do informacji, że jego dane są prywatne i zapisane w Supabase. Eksport nie jest dostępny w MVP.
Kryteria akceptacji:

- Widoczna informacja o prywatności i miejscu przechowywania danych.
- Brak opcji eksportu w MVP.

US-021
Tytuł: Zabezpieczenie UI przed wielokrotnymi submitami
Opis: Formularze i akcje AI mają safeguardy przed wielokrotnym kliknięciem.
Kryteria akceptacji:

- Po wysłaniu formularza przycisk jest disabled do zakończenia requestu.
- Duplikaty nie tworzą wielu wpisów.

US-022
Tytuł: Obsługa długich opisów i edge-case tekstowych
Opis: System daje czytelne walidacje dla zbyt długich opisów i specjalnych znaków.
Kryteria akceptacji:

- Przekroczenie limitu znaków blokuje submit z komunikatem.
- Tekst z emoji i symbolami jest poprawnie przechowywany i wyświetlany.

US-023
Tytuł: Zgłaszanie błędów i feedback (opcjonalnie)
Opis: Użytkownik może zgłosić problem lub przesłać feedback.
Kryteria akceptacji:

- Formularz feedback wysyła wiadomość do logów/systemu ticketowego (MVP: zapis do DB lub prosty mail webhook).

(Każda z powyższych historyjek jest testowalna — zawiera konkretne warunki i wynik oczekiwany.)

## 6. Metryki sukcesu

- Aktywność użytkowników: 80% użytkowników dodaje ≥3 sny tygodniowo.
- Przydatność interpretacji: 70% interpretacji ocenionych ≥3/5 gwiazdek.
- Stabilność: brak krytycznych błędów w testach E2E i deployie.
- Zaangażowanie: średni czas sesji > 5 minut.
- Operacyjność: pipeline CI/CD uruchamia test E2E przed deployem i przechodzi pomyślnie.

Dodatkowe metryki (opcjonalne):

- Liczba wygenerowanych interpretacji dziennie/tygodniowo.
- Procent użytkowników zmieniających ustawienie przypomnienia.
- Retencja 7/30 dni.

---

Plik ten jest gotowy do użycia jako PRD MVP. Jeśli chcesz, mogę:

- Dodać sekcję techniczną: schemat DB (tabele `users`, `dreams`, `interpretations`, `ratings`).
- Przygotować przykładowe API prompts dla `gpt-4o-mini` i przykładowy payload.
- Wygenerować listę E2E testów w Playwright w formacie executable.
