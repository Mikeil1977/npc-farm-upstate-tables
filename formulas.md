# Realm Resource Formulas (v1)

This file describes how to turn asset stats into realm-level Operational Points (OP) and Resource Points (RP).

---

## 1. Per-Asset Stats

Each asset in `assets.json` has the following numeric fields:

- `food`
- `fish`
- `ore`
- `trade`
- `prosperity`
- `population`
- `administration`
- `stability`
- `rp_bonus`
- `op_bonus`
- `threat`
- `upkeep`

Any field not present on an asset is treated as **0**.

To get the realm totals, you sum each field across **all assets in the realm**.

---

## 2. Prosperity Tier

Prosperity can be any non-negative integer.

We convert **Prosperity** into a **Prosperity Tier**:

> `ProsperityTier = floor(Prosperity / 3)`

- Prosperity 0–2 → Tier 0  
- Prosperity 3–5 → Tier 1  
- Prosperity 6–8 → Tier 2  
- etc.

Prosperity Tier is used in both OP and RP calculations.

---

## 3. Operational Points (OP)

OP represents how many **strategic actions** a realm can take per turn.

Base formula:

> `OP = Administration + Population + ProsperityTier + Stability + sum(op_bonus)`

Where:

- `Administration` = sum of all asset `administration`
- `Population`    = sum of all asset `population`
- `Stability`     = sum of all asset `stability`
- `ProsperityTier` as defined above
- `op_bonus`      = sum of all asset `op_bonus` values (rare special cases)

Round normally; values can’t go below 0.

---

## 4. Resource Points (RP)

RP represents how many **resources** (money, goods, favours) a realm gains per turn.

Base production:

> `BaseProduction = Food + Fish + Ore + Trade`

Where each term is the sum across all assets:

- `Food` = sum(food)
- `Fish` = sum(fish)
- `Ore`  = sum(ore)
- `Trade` = sum(trade)

Add Prosperity Tier:

> `BaseWithProsperity = BaseProduction + ProsperityTier`

Subtract upkeep:

> `AfterUpkeep = BaseWithProsperity - Upkeep`

Where:

- `Upkeep` = sum(upkeep) across all assets

Add any flat RP bonuses:

> `RP = AfterUpkeep + sum(rp_bonus)`

Again, RP can be negative (a realm that is bleeding resources), but most starting realms should be at least 0 or slightly positive.

---

## 5. Threat

Threat is not currently folded into OP/RP, but is tracked as:

> `Threat = sum(threat)`

Typical uses:

- `Threat` feeds into random event tables.  
- High Threat → higher chance of monster attacks, rebellions, disasters, etc.  
- Ruins, wrecks, cursed places, and dangerous wild sites usually add Threat.

Exact usage is left to the campaign or event tables.

---

## 6. Storehouses and Stockpiles

A **Storehouse** or similar structure doesn’t need special rules here:

- A realm has a current **RP stockpile** (e.g. starts with 10 RP).  
- Each turn, you:
  1. Calculate `RP` for the turn using the formula above.  
  2. Add it to the stockpile (which may go up or down).  

Storehouses might give extra `rp_bonus`, reduce `upkeep`, or protect stockpiles from loss, but those are modelled per-asset in `assets.json`.

---

## 7. Extensibility

If you add new resource types later (e.g. `magic`, `influence`), you can:

1. Add new keys to `assets.json`.
2. Decide whether they:
   - feed into OP,  
   - feed into RP, or  
   - stand alone as separate tracked values.

Update this file to document any new formulas.
