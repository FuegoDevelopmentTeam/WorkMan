# BACKLOG_PROMPT_CACHE_WorkMan.md — WorkMan Local Prompt Cache

Version: 1.0.0 (Deferred Tech Tasks)  
Date: 2026-06-14  
Status: DEFERRED (Pending Phase 3 Completion)  

Ez a fájl a WorkMan modul **helyi prompt váróterme (Prompt Cache)**. Ide kerülnek azok a magasabb szintű technikai specifikációk és fejlesztési promptok, amelyek a jelenlegi fázisban még nem végrehajthatóak, megakadályozva a kódoló Agentek túl korai implementációs kísérleteit.

---

## 📌 Aktív Fázis Ellenőrzés
*   **Jelenlegi projektfázis:** **Phase 0 (Bootstrap)** — Lásd: `docs/APP_STATE_WorkMan.md`.
*   **Ez a backlog legkorábban itt aktiválható:** **Phase 3 (Financial) / Phase 4 (Marketplace)**.

*TILOS ezen feladatok kódolásába kezdeni, amíg az `docs/APP_STATE_WorkMan.md` szerint a projekt el nem éri a megfelelő fázist!*

---

## 📥 Eltárolt Fejlesztői Promptok (Cached Prompts)

### 🔴 PR-003 — WorkMan Pénzügyi Elszámoló Motor és Felelősség-követés (D062, D046, D073)
```markdown
[TARGET_AGENT: WorkMan Tech Lead & Full-Stack Developer]
[DEPENDENCY: Phase 2 Completion in WorkMan]
[SPEC_LINK: docs/MASTER_CONCEPT_WorkMan.md]

MŰSZAKI IMPLEMENTÁCIÓS PROMPT:
Kérlek, valósítsd meg a WorkMan modul pénzügyi és elszámolási motorját a MASTER_CONCEPT_WorkMan.md specifikációi alapján:

1. Cash & Advance Ledger (Adatbázis):
   - Építsd ki a készpénzes, előleg és kölcsön (cash advances, loans) nyilvántartási Supabase táblákat (White/Grey/Black támogatással).
2. Valós idejű fizetési feed (Real-Time Earnings):
   - Készítsd el a dashboard komponenst, amely a `time_logs` (órarendi), `fixed_fee` és levont `advances` alapján valós időben mutatja a munkavállaló (önkéntes) mai és jövőbeli várható bevételét.
3. Garanciális (Decay / Szavatosság) Rendszer:
   - Valósíts meg egy Edge Functiont, ami a feladat lezárása (completed) után elindít egy visszaszámlálót. A letétbe helyezett fizetés / bizalmi pont csak a szavatossági idő lejárta után szabadul fel véglegesen.
```
