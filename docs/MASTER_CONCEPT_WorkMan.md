# MASTER_CONCEPT_WorkMan.md — WorkMan Technical Specification

Version: 1.0.0 (Work-Order, Resource & Value-Flow Engine)  
Date: 2026-06-14  
Status: RELEASED  
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