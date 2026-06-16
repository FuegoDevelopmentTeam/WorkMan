# MASTER_CONCEPT_WorkMan.md — WorkMan Technical Specification

Version: 1.3.0 (Work-Order, Resource & Value-Flow Engine + DANA v1.34.0 platform-elv adoptáció + DANA v1.36.0 Kulturális Mozgalom adopció: D097/D100/D104/D109 — §7 + DANA v1.37.0 Kooperatív Szakmai Ív adopció: D112/D118 — §8)  
Date: 2026-06-16  
Status: RELEASED  

> **DANA Platform-elv Adoptáció (v1.34.0):** **P52** (a `cash_advances`, előleg, kölcsön = **ledger-számlák** egy tier-dimenziós naplóban, nem külön rendszer; White/Grey/Black egy `ledger_tier` mező); **P53** (a Settlement Matrix [hourly/fixed/revenue-share/combined] a közös **elosztó-vízesés** primitíven, Money value object, determinisztikus kerekítés); **P60** (a szavatossági/Decay reputáció a **közös reputáció-gráfba** csatlakozik, nem külön); **P56/D095** (külső/privát projektek = meta-org-buborékon kívüli vagy scope-olt tételek); **P55** (a szavatossági idő valid-time). Cross-modul: a WorkMan kimenete DrBill-be (P44 esemény) és a DANA ledgerbe (D029) projektál.
> **Optimalizációs és Skálázási Paradigmaváltás (P63-P70):** **P66 (Durable Execution):** A szavatossági letét (Decay) és a work-order→warranty→settlement saga folyamatokat egy Durable Execution Motor (pl. Temporal.io) vezérli a törékeny status oszlopok és cron jobok helyett. **P64 (Event Sourcing):** A valós idejű fizetési kimutatások és előlegek eseménynaplókból épülnek fel.
> **Senior Review II Adoptáció (DANA v1.35.0 — P71–P82, D096):** **P72/P78** (komplexitás-költségvetés): a P66 Temporal és a blanket Event Sourcing **alapból KIKAPCSOLT** — a warranty/settlement MVP-ben **sima Postgres status + scheduled job**; ES csak a pénzügyi posztolásnál; Temporal csak mért triggerre. **P74/P75** (FinTech-mag): a `cash_advances`, a settlement-tételek és a warranty-letét **nem** önálló érték-táblák, hanem a **kanonikus Számlatükörre + Party–Account magra** posztolnak a központi ledgeren (double-entry, P03) — egy dolgozó teljes nettó pozíciója (előleg − ledolgozott + letét) egyetlen egyenleg-lekérdezés. **P76** (suspense): a work-order→warranty→settlement saga lépései függő számlára posztolnak (a letét = explicit suspense-egyenleg). **P71** (Shapley): a revenue-share/projekt-elosztás a közös Shapley-elosztóra kerül (P53 waterfall fölött). **P81** (jurisdikció-pluggolható adó): a munkadíj/anyag bizonylat adó-kezelése a közös rule-packen át. **P80** (invariáns-guardrail): folytonos trial-balance a cash_advance ledgeren.
Tech Stack: Next.js (Frontend), Supabase/PostgreSQL (Backend), Tailwind CSS (UI)

---

## 1. Rendszer-architektúra és Filozófia

A **WorkMan (Work Manager)** a DANA ökoszisztéma dedikált munka-, erőforrás- és értékáramlás-követő modulja. Míg a BeatPass a *terekre és időre* (foglalások), a KineLex a *tudásra* (oktatás) fókuszál, a WorkMan a **fizikai és szellemi munka végrehajtását** menedzseli. 

### A Magprobléma
A meta-szervezetekben dolgozó karbantartók, építők, gondnokok és önkéntesek számára a munkavállalás ma **rizikós és átláthatatlan**. A dolgozó nem tudja:
*   Melyik feladatba kezdjen bele?
*   Hol vannak a szerszámok, alapanyagok (festék, kilincs)?
*   Vannak-e függőségek (pl. le kell kapcsolni az áramot, elzárni a vizet)?
*   Lefoglalták-e az épületrészt a munkára, vagy bezavar egy táncóra?
*   Pontosan mennyit fog ezzel keresni (óradíj vs. fix díj)?

A WorkMan célja ezen **információs aszimmetria megszüntetése**, ezáltal a munkavállalás kockázatának minimalizálása.

---

## 2. A Work-Order (Feladat) Életciklus és Triage (D062)

Bármilyen munka a rendszerben egy **Work Order (WO)** entitás. 

### A. Bejelentés (Discovery)
A rendszer radikálisan nyitott: **Bárki** (takarító, gondnok, tanár, de akár egy diák is) jelezhet problémát (pl. "Letört a 3-as terem kilincse", "Beázott a mennyezet"). A bejelentés egy `Draft WO`-t hoz létre képpel és helyszín-címkével.

### B. Menedzseri Triage & Dokumentáció
A menedzser vagy a Head of Facility átveszi a Draft-ot, és "kikristályosítja" a munkát. Hozzáadja:
*   **Alapanyag és Eszköz lokáció:** (pl. "A pót kilincs a B raktár 3-as polcán van, csavarhúzó a műhelyben").
*   **Kivitelezési tervek / Blueprint:** Rendszerszintű épülettervek csatolása (pl. "Itt fut a vízvezeték").
*   **Függőségek (Dependencies):** Milyen egyéb feladatnak kell előbb elkészülnie (pl. "Áramtalanítás az M1 kapcsolón").
*   **Tér-idő zárolás:** Integráció a BeatPass naptárral (D059, D060) → a szerelés idejére a termet a rendszer lezárja.

### C. Bidding & Piactér (A Munkavállaló Döntése)
A kikristályosított feladat kikerül a **WorkMan Belső Piactérre**. Az önkéntes vagy dolgozó pontosan látja, hogy mit kell tenni, mi áll rendelkezésre, és **mennyit kereshet vele**. Mivel a rizikó nulla, szívesen elvállalja.

---

## 3. Pénzügyi Elszámolás és Valós Idejű Feed (D012, D046, D073)

A WorkMan a DANA White/Grey/Black Ledger (D002) filozófiáját alkalmazza a munkaerő elszámolására is.

### A. Kompozit Díjazási Mátrix (Settlement Matrix)
Egy feladat elszámolása rugalmasan kombinálható:
1.  **Órabéres (Hourly):** `time_logs` alapján (pl. takarítás 2000 Ft/h).
2.  **Teljesítési díjas (Fixed Fee):** Egy konkrét feladat elvégzése (pl. "Kilincs csere = 1500 Ft").
3.  **Projekt / Revenue Share:** Osztalék-szerű részesedés a projekt bevételéből (pl. rendezvény építés → jegybevétel 2%-a).
4.  **Kombinált:** Pl. Alap óradíj + Bónusz a határidő előtti befejezésért.

### B. Valós Idejű Fizetési Kimutatás (Real-Time Earnings Feed)
A dolgozó a dashboardján élőben látja: *"A mai tervezett munkáimmal 12.500 Ft-ot fogok keresni"*. Amint kipipál egy feladatot, a mérőeszköz azonnal jóváírja az összeget (Wallet pontban vagy Cash-ben).

### C. Előlegek, Kölcsönök és Cash Nyilvántartás (Cash Advances)
Különösen az építési munkáknál és önkénteseknél gyakori az előleg, a kölcsön és a nem hivatalos ("zsebből zsebbe") kifizetés. A WorkMan egy dedikált `cash_advances` ledger-t vezet. Ha a dolgozó felvett 50.000 Ft előleget anyagbeszerzésre vagy saját célra, a rendszer az elvégzett munkáival (Work Orders) automatikusan "dolgozza le" az előleg egyenlegét a pontos és korrekt elszámolás végett.

---

## 4. Felelősség, Szavatosság és Minőségbiztosítás (Decay & Liability)

Az old-school világban a "kész van" azt jelenti, hogy a munkás megkapja a pénzt és eltűnik. A WorkMan bevezeti a **Szavatossági Idő (Warranty / Decay)** fogalmát.

*   **Zárolt Bónusz:** A kifizetés egy része (pl. 20% bónusz) vagy a bizalmi reputációs pont (P32) "szavatossági letétbe" kerül.
*   **Decay Időszak:** Beállítható időablak (pl. 2 hét egy csapszerelésnél). Ha ezalatt a probléma újra előjön (ugyanott kezd el csöpögni), a felelősség (liability) a dolgozót terheli → Neki kell kijavítania garanciában, különben elveszti a letétet és csökken a megbízhatósági pontja.
*   **Sikeres Lejárat:** Ha a szavatossági idő hiba nélkül lejár, a rendszer automatikusan felszabadítja a maradék díjat/pontot.

---

## 5. Cross-Domain (Kereszt-Modul) Projektek Orchestrációja

A WorkMan nemcsak a falfestésre, hanem a meta-szervezet **minden** projekt-jellegű erőfeszítésére használható. A motor agnosztikus a tartalomra.

*   **Fizetős Táncos Fellépés (Artistic Project):** Projekt = "Szilveszteri Mambo Show". Függőségek = Kosztüm beszerzése, 4 db próba beiktatása a BeatPass naptárba, koreográfia betanulása a KineLex-ből. Elszámolás = A megrendelőtől kapott díj szétosztása a táncosok között (Revenue Share).
*   **Külső, Nem Hivatalos Projektek (External Gig):** Pl. a közösség építő emberei elmennek az egyik táncos fürdőszobáját kicsempézni. A WorkMan ledger kezeli az anyagköltség-előleget, az órabéreket és a "barter" elszámolást (D018) a hivatalos stúdiókönyveléstől teljesen elzárt, privát térben.
*   **Tánccsoportok Projekt Költségvetése:** Bevétel és kiadás követése egy adott csapatra (pl. "Versenycsapat utazása").

---

## 6. Adatbázis Séma (Supabase / PostgreSQL)

```sql
-- 1. Projektek (Agnosztikus gyűjtő: Építés, Tánc show, Külső munka)
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL,
    manager_id UUID REFERENCES auth.users(id),
    title VARCHAR(150) NOT NULL,
    description TEXT,
    project_type VARCHAR(50),             -- 'maintenance' | 'artistic_show' | 'external_gig'
    budget_allocated DECIMAL(12,2),
    status VARCHAR(20) DEFAULT 'planning', -- 'planning' | 'active' | 'completed'
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. Munka/Feladat (Work Order)
CREATE TABLE work_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id),
    reporter_id UUID REFERENCES auth.users(id),  -- Bárki jelentheti
    assignee_id UUID REFERENCES auth.users(id),  -- Munkavállaló
    title VARCHAR(150) NOT NULL,
    description TEXT,
    blueprint_url VARCHAR(255),                  -- Teremrajz, terv
    tools_location TEXT,                         -- Hol vannak a szerszámok/anyagok
    dependencies JSONB,                          -- Előfeltételek (pl. áramtalanítás)
    beatpass_room_id UUID,                       -- Foglalt terület
    status VARCHAR(20) DEFAULT 'draft',          -- 'draft' | 'triaged' | 'open_for_bidding' | 'in_progress' | 'warranty_period' | 'completed'
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Pénzügyi és Elszámolási Modell
CREATE TABLE settlement_rules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_order_id UUID NOT NULL REFERENCES work_orders(id),
    hourly_rate DECIMAL(10,2),
    fixed_fee DECIMAL(10,2),
    revenue_share_percent DECIMAL(5,2),
    warranty_retention_percent DECIMAL(5,2) DEFAULT 0.00, -- Szavatossági letét %
    warranty_period_days INT DEFAULT 0
);

-- 4. Készpénz Előleg és Kölcsön Nyilvántartás (Cash Advances)
CREATE TABLE cash_advances (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id),
    issuer_id UUID NOT NULL REFERENCES auth.users(id),  -- Aki adta a pénzt
    amount DECIMAL(12,2) NOT NULL,
    advance_type VARCHAR(20) NOT NULL,           -- 'material_purchase' | 'salary_advance' | 'loan'
    ledger_tier VARCHAR(10) DEFAULT 'grey',      -- 'white' | 'grey' | 'black'
    balance_remaining DECIMAL(12,2) NOT NULL,    -- Mennyit kell még ledolgozni / elszámolni
    issued_at TIMESTAMPTZ DEFAULT NOW(),
    cleared_at TIMESTAMPTZ
);

-- 5. Munkaidő és Teljesítés (Time Logs & Execution)
CREATE TABLE time_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_order_id UUID NOT NULL REFERENCES work_orders(id),
    user_id UUID NOT NULL REFERENCES auth.users(id),
    check_in TIMESTAMPTZ NOT NULL,
    check_out TIMESTAMPTZ,
    evidence_media_urls JSONB,                   -- Képek (Előtte/Utána)
    status VARCHAR(20) DEFAULT 'pending_approval'
);
```

---

## 7. DANA v1.36.0 Adopció — Koprodukció & Művészeti Projektek (Top-Down ▼, D097-D109)

A WorkMan a „Kulturális Mozgalom" kör (DANA D097-D109) **művészeti-projekt** vonatkozásait adoptálja — a content-agnosztikus projekt/settlement-motor (§5) természetes gazdája.

*   **D097 (CoChoreo Koprodukciós Royalty) — fő haszonélvező:** a `projects` (`project_type='artistic_show'`) + `settlement_rules` (revenue-share) a több tanár/koreográfus közös kűrjének elszámolója. A hozzájárulás-arány a **Shapley-alapú** elosztóra (P71) kerül a P53 waterfall fölött (a `revenue_share_percent` ad-hoc súlyok helyett); a több jogi láb közti zárás DrBill multilaterális nettinggel (P68).
*   **D100 (Művészeti Kockázati Alap):** a `cash_advances` ledger adja a kísérleti show előlegét (kosztüm, próbaterem), amit a show bevétele „dolgoz le"; a keret-allokáció peer komparatív bírálattal (D076) dől el.
*   **D104 (Tanári Rezidenciák):** a kereszt-entitás társtanítás mint projekt; a settlement a DrBill routing/klíring felé projektál.
*   **D109 (Non-Harm):** a szavatossági/megbízhatósági reputáció (P60) a wellbeing-keretbe is becsatornázódik (biztonságos, fenntartható munkavégzés).

> **Felszólítás (a User közvetíti):** a koprodukciós settlement Shapley-integrációja és a creative-fund előleg-logika a WorkMan Tech Lead hatásköre; a felosztás/klíring a DrBill-lel, a hozzájárulás-adat a KineLex/MeCat timeline-okkal egyeztetendő (`[CROSS-MODULE DELEGATION]`).

---

## 8. DANA v1.37.0 Adopció — Hozzájárulói Tükör & Show-Settlement (Top-Down ▼, D110-D120)

A WorkMan a kooperatív szakmai ív **valós idejű kereset-** és **projekt-elszámolási** vonatkozásait adoptálja.

*   **D112 (Owner's Lens):** a Real-Time Earnings Feed (§3B) a forrása a BeatPass/Core személyes mini-P&L („owner's lens") widgetnek — a ledolgozott/várható kereset + a cash_advance pozíció (§3C) egyetlen nettó nézetben (P75 Party–Account). Ez a tulajdonosi szemlélet (D061) adat-oldali bemenete.
*   **D118 (Verseny- és Színpad-Ív):** a versenyző→koreográfus pálya show-/projekt-elszámolása a `projects` (`project_type='artistic_show'`) + Shapley-elosztó (P71) motoron; a Creative Fund (D100) előlege a `cash_advances`-ből, amit a show bevétele „dolgoz le".

> **Felszólítás (a User közvetíti):** az earnings→owner's-lens feed és a show-settlement a WorkMan Tech Lead hatásköre; a felosztás/klíring a DrBill-lel egyeztetendő (`[CROSS-MODULE DELEGATION]`).