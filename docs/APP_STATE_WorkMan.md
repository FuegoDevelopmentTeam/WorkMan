# APP_STATE_WorkMan.md — WorkMan Active Project State

Version: 0.1.0  
Date: 2026-06-14  
Current Phase: **Phase 0: Bootstrap & Core Schema**  
Status: IN_PROGRESS (Not started yet)  

Ez a fájl a WorkMan modul **aktív rendszer- és fejlesztési állapotát (Source of Truth)** rögzíti. Minden ide belépő Agentnek kötelező ezt elsőként beolvasnia, hogy lássa, hol tart a projekt, és mik a soron következő lépések.

---

## 🧭 1. Jelenlegi Fázis és Mérföldkövek (Phase & Milestones)

A WorkMan jelenleg a **Phase 0** (legelső lépések) kapujában áll. 

```
[►] Phase 0: Bootstrap & Core Schema  <-- JELENLEG AKTÍV
[ ] Phase 1: Basic Operations & CRUD
[ ] Phase 2: Automated & IoT Integration
[ ] Phase 3: Financial & Revenue Engine
[ ] Phase 4: Succession & Marketplace
```

---

## 🎯 2. Aktív Backlog (Soron következő feladatok - Next Up)

### Task 0.1: Repozitórium Inicializálása & Könyvtárszerkezet
*   **Státusz:** `[PENDING]`
*   **Leírás:** Hozd létre a standard Next.js / Supabase könyvtárszerkezetet.
*   **Kimenet:** `/app` mappa Next.js boilerplate-tel, `/supabase` mappa konfigurációval.

### Task 0.2: Adatbázis Inicializálása (Core Schema migrations)
*   **Státusz:** `[PENDING]`
*   **Leírás:** Hozd létre az alapvető Supabase PostgreSQL táblákat (`work_orders`, `projects`, `time_logs`, `cash_advances`).
*   **Kimenet:** Első Supabase migrációs fájlok (`0001_initial_schema.sql`).

### Task 0.3: Környezeti változók & Auth konfiguráció
*   **Státusz:** `[PENDING]`
*   **Leírás:** Állítsd be a Next.js és Supabase összeköttetést (.env.local), és konfiguráld az alapvető autentikációt.

---

## 🧊 3. Elnapolt / Jövőbeli Feladatok (Deferred Backlog)

A magasabb fázisok specifikációit biztonságosan eltároltuk:
*   `docs/BACKLOG_PROMPT_CACHE_WorkMan.md`

---

## 📜 4. Fejlesztési Napló (Changelog)
*   **2026-06-14 (v0.1.0):** Első rendszerállapot leírás, koncepció rögzítése. (Szőnyi Levente / DANA Stratéga)