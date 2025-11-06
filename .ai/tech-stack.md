# Tech stack — DreamInsight (MVP)

Krótka decyzja: używamy Next.js jako głównego frameworka (frontend + serverless API), Supabase jako backend-as-a-service i Openrouter do integracji z modelami AI. Taka kombinacja daje najszybszą ścieżkę do MVP przy zachowaniu bezpieczeństwa i możliwości skalowania.

## Frontend

- Next.js (app router / Server Actions where useful) + React 19
- TypeScript 5
- Tailwind CSS 4
- shadcn/ui dla gotowych komponentów i szybkiego UI
- E2E: Playwright

Dlaczego: pełny React + server-side/edge API w jednym frameworku — prostsze sesje, SSR/ISR dla list i dashboardu, łatwy deploy na Vercel.

## Backend / DB / Auth

- Supabase (Postgres + Auth + RLS)
- Schemat minimalny: `users`, `dreams` (kolumna `interpretation`), opcjonalnie `interpretation_ratings`
- Wymuszenie bezpieczeństwa: RLS — rekordy widoczne tylko dla właściciela (owner = auth.uid())

## AI

- Openrouter jako warstwa pośrednia (możliwość zamiany providerów)
- Model domyślny: `gpt-4o-mini` dla pełnych interpretacji (zgodnie z PRD)
- Tańszy model (np. gpt-3.5) lub lokalny heurystyczny parser do propozycji tagów/emocji
- Wszystkie wywołania AI wykonywane na serwerze (Next API Route lub Supabase Edge Function) — klucze API w secrets, nigdy w przeglądarce
- Obsługa błędów: retry, timeout, circuit-breaker; rate-limiting na endpointach AI

## Hosting / CI

- Frontend/API: Vercel (preferowane dla MVP — zero-config Next deploys)
- DB/Auth: Supabase (hosted)
- CI: GitHub Actions — lint/test → build → deploy
- Opcjonalnie: DigitalOcean droplet / worker jeśli potrzebujesz stałych background workerów (np. kolejka interpretacji)

## Bezpieczeństwo & operacje

- RLS + auth checks obligatoryjne
- Żadne klucze AI w frontendzie; wrapper server-side
- Store interpretacje w `dreams.interpretation` (PRD — oszczędza koszty)
- Rate-limits i monitoring kosztów AI (alerty przy progach)
- Sanitizacja promptów i ograniczenie długości wejść (prompt-injection mitigation)

## Skalowalność / Koszty

- Supabase + Postgres skaluje do fazy PMF; AI to główny koszt — kontrolować przez:
  - cache/ zapis interpretacji,
  - domyślnie tańsze modele do sugestii,
  - quota/rate-limits per user
- Przy wzroście: dodać worker/queue (Redis / Sidekiq-like / Supabase + external worker)

## Dev / env

- Dev env vars (minimum):
  - `SUPABASE_URL`, `SUPABASE_ANON_KEY` (public), `SUPABASE_SERVICE_ROLE_KEY` (server-only)
  - `OPENROUTER_API_KEY` (server-only)
  - `NEXTAUTH_URL` / inne jeśli używasz dodatkowych auth flows
- Local dev: `vercel dev` or `next dev` + supabase local emulator / hosted test instance

## Minimalne implementacyjne ustalenia do startu

- Implementować AI wrapper jako Next API route lub Supabase Edge Function (ukrycie klucza + retry + rate-limit)
- Włączyć RLS: `createPolicy('is_owner', ... using (user_id = auth.uid()))`
- Interpretacje zapisujemy i nigdy nie generujemy on-read
- Propozycje tagów: domyślnie client-side heuristics; fallback do tańszego modelu
