```md
## 6️⃣ Cursor AI Build Checklist (Step-by-Step)

This is a literal execution checklist for implementing the mod with minimal backtracking.
Follow in order.

---

### A) Repo & File Setup

1. Create folder structure:
   - `GameData/EpsteinDrive/Parts/Engines/`
   - `GameData/EpsteinDrive/Localization/`
   - `GameData/EpsteinDrive/Patches/`

2. Create empty files:
   - `GameData/EpsteinDrive/Parts/Engines/EpsteinDrive_Frigate.cfg`
   - `GameData/EpsteinDrive/Parts/Engines/EpsteinDrive_Cruiser.cfg`
   - `GameData/EpsteinDrive/Parts/Engines/EpsteinDrive_Battleship.cfg`
   - `GameData/EpsteinDrive/Localization/en-us.cfg`
   - `GameData/EpsteinDrive/Patches/Dependencies.cfg`

3. Add a minimal `README.md` describing:
   - Required dependency: CommunityResourcePack (CRP)
   - Engines included: Frigate / Cruiser / Battleship
   - Two-mode design (MultiModeEngine)

---

### B) Localization (Do this before parts)

4. In `en-us.cfg`, add localization keys for each part:
   - Title, description, manufacturer
   - Mode display names: Cruise, Overburn, Atmo Ejection

5. Use a consistent naming scheme for localization keys:
   - `#LOC_EpsteinDrive_part_Frigate_title`
   - `#LOC_EpsteinDrive_part_Frigate_desc`
   - `#LOC_EpsteinDrive_mode_Cruise`
   - `#LOC_EpsteinDrive_mode_Overburn`
   - `#LOC_EpsteinDrive_mode_AtmoEjection`

---

### C) Dependencies Patch (CRP presence)

6. In `Dependencies.cfg`, add a ModuleManager patch that:
   - Runs only if CRP is present: `:NEEDS[CommunityResourcePack]`
   - (Optional) If CRP missing, add a clear on-screen warning via `@description` or disable parts
     using `category = none` in a `:NEEDS[!CommunityResourcePack]` patch.

Goal: prevent confusing “engine won’t run” behavior when CRP isn’t installed.

---

### D) Shared Engine Template (Author once, then copy)

7. Create a shared internal template pattern used across all three engine configs:

Required modules per engine part:
- `MultiModeEngine`
- `ModuleEnginesFX` x2 (one per mode)
- `ModuleGenerator` (passive 100 EC/s)
- `ModuleAlternator` (up to 1000 EC/s at full throttle)

Required propellants (both modes, all engines):
- Water ratio 1000
- FusionPellets ratio 1

8. Ensure both `ModuleEnginesFX` blocks:
- Have unique `engineID` values matching the MultiModeEngine IDs
- Include the same propellant list and ratios
- Differ only by:
  - `maxThrust`
  - `heatProduction`
  - `atmosphereCurve`
  - effects (optional)

---

### E) Implement EpsteinDrive_Frigate.cfg

9. Create `PART {}` with:
- `name = EpsteinDrive_Frigate`
- category = Engine
- reasonable mass/cost placeholders (tune later)

10. Add **power modules**:
- `ModuleGenerator` → 100 EC/s always-on
- `ModuleAlternator` → 1000 EC/s under thrust

11. Add `MultiModeEngine`:
- primary = Cruise
- secondary = AtmoEjection
- Use localized display names

12. Add `ModuleEnginesFX` for **Cruise**:
- `engineID = Cruise`
- high vacuum Isp (for Frigate tier)
- steep atmo drop (poor atmo)
- set maxThrust (Frigate low–medium band)

13. Add `ModuleEnginesFX` for **AtmoEjection**:
- `engineID = AtmoEjection`
- low overall Isp
- flatter atmosphereCurve (works well in dense air)
- vacuum Isp only slightly better than sea-level
- lower heatProduction
- maxThrust medium (sufficient for descent/ascent)

14. Add optional:
- `ModuleGimbal` (small gimbal range, if desired)
- `ModuleSurfaceFX` (if desired)

---

### F) Implement EpsteinDrive_Cruiser.cfg

15. Copy Frigate file as a base.

16. Update:
- `name = EpsteinDrive_Cruiser`
- Modes = Cruise / Overburn

17. Cruise mode:
- Increase maxThrust to Cruiser medium–high band
- Increase vacuum Isp slightly above Frigate (5–15%)
- Keep steep atmo drop

18. Overburn mode:
- Same propellant ratios
- maxThrust = 1.3×–1.6× Cruise
- Isp lower than Cruise
- heatProduction higher than Cruise

---

### G) Implement EpsteinDrive_Battleship.cfg

19. Copy Cruiser file as a base.

20. Update:
- `name = EpsteinDrive_Battleship`
- Modes = Cruise / Overburn

21. Cruise mode:
- Increase maxThrust to Battleship very high band
- Increase vacuum Isp slightly above Cruiser (5–15%)

22. Overburn mode:
- maxThrust = 1.3×–1.6× Cruise
- Isp lower than Cruise (but may still be strong)
- heatProduction highest of all modes

---

### H) Consistency Pass (Do not skip)

23. Verify on all three parts:
- Exactly two modes
- `MultiModeEngine` IDs match `ModuleEnginesFX engineID`
- Both modes use:
  - Water ratio 1000
  - FusionPellets ratio 1
- Both power modules present and correct:
  - 100 EC/s passive
  - 1000 EC/s alternator

24. Verify localization:
- All titles/descriptions reference localization keys (no hard-coded English strings)

25. Verify no forbidden modules/resources appear:
- No WasteHeat
- No Megajoules/ThermalPower
- No B9PartSwitch

---

### I) In-Game Validation Checklist

26. Install dependencies:
- KSP + ModuleManager
- CommunityResourcePack (CRP)

27. Confirm in editor:
- All 3 engines appear under Engine category
- Mode switching appears and works in PAW (right-click menu)

28. Confirm in flight:
- Passive EC generates when engine is off (~100 EC/s)
- Under thrust, EC increases significantly (alternator output)
- Engine consumes Water and FusionPellets at the fixed ratio
- Mode switching swaps thrust/Isp behavior as intended

29. Confirm expected behavior by class:
- Frigate: AtmoEjection feels usable in thick air
- Cruiser/Battleship: Overburn clearly higher thrust and worse Isp than Cruise
- Battleship: slightly better vacuum efficiency than Cruiser

---

### J) Tuning Pass (After it works)

30. Tune in this order (one variable at a time):
1) Cruise vacuum Isp by size tier (Frigate < Cruiser < Battleship)
2) Overburn thrust multiplier (1.3×–1.6×)
3) AtmoEjection atmosphereCurve flatness
4) heatProduction values per mode
5) mass/cost for internal progression

Stop tuning as soon as the engines “feel right” for the mod’s intended fantasy.

---
```
