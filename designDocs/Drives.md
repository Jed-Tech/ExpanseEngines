## 1️⃣ Overview

This mod introduces a family of **Epstein Drive** engines inspired by *The Expanse*,
designed as **integrated reactor–engine systems** that provide continuous high thrust,
long-duration acceleration, and abundant onboard power.

The focus is on **role-based engine variants**, simple operation, and internal balance
within the mod—without regard for stock KSP balance or realism constraints.


## 2️⃣ Part Role Definition 

This section defines the **intended role, operating modes, and design philosophy**
for each Epstein Drive variant. These roles are foundational constraints and must
be preserved during implementation and tuning.

---

### EpsteinDrive_Frigate

**Modes**
- Cruise
- Atmo Ejection

**Role**
- Frigate-class and smaller vessels
- Capable of operating in dense atmospheres
- Suitable for planetary landing and ascent
- Lower thrust and lower absolute efficiency than larger drives

**Design Intent**
- Flexible and forgiving
- Emphasizes atmospheric handling over raw acceleration
- Intended for escorts, scouts, landers, and general-purpose ships

---

### EpsteinDrive_Cruiser

**Modes**
- Cruise
- Overburn

**Role**
- General-purpose torchship drive
- Optimized for interplanetary and interstellar travel
- Not intended for routine atmospheric operations
- Balanced thrust versus efficiency

**Design Intent**
- The default Epstein Drive experience
- Long-duration cruise burns with optional high-thrust Overburn
- Core workhorse engine for capital ships

---

### EpsteinDrive_Battleship

**Modes**
- Cruise
- Overburn

**Role**
- Capital-ship-class Epstein Drive
- Highest thrust output of the series
- Slightly more vacuum-efficient than Cruiser-class drives
- Not designed for atmospheric operations

**Design Intent**
- Strategic mobility for very large vessels
- Prioritizes sustained acceleration and Overburn capability
- Intended for battleships, carriers, and station-scale ships

---

### Cross-Cutting Role Rules

- All engines use:
  - **Water** as reaction mass
  - **FusionPellets** as reactor fuel
- All engines maintain a fixed **1000:1 Water : FusionPellets** consumption ratio
- Larger engines are **slightly more vacuum-efficient** than smaller engines
- No engine variant supports all three modes
- Atmospheric capability is exclusive to Frigate-class engines

---

## 3️⃣ Engine Systems (Canonical + Final)

This section defines **shared mechanical systems** used by all Epstein Drive parts.
These rules apply universally unless explicitly overridden.

---

### Integrated Reactor Model

- Epstein Drives are **single-part systems**
- Each part includes:
  - A fusion reactor
  - A magnetic drive / exhaust assembly
- There is no separate reactor part

---

### Power Generation

Each Epstein Drive provides ship power via two reactor-driven mechanisms.

**Passive Reactor Output**
- Always active (engine on or off)
- Produces **100 ElectricCharge per second**
- Represents reactor idle power and baseline ship systems

**Alternator Output (Under Thrust)**
- Active only while the engine is producing thrust
- Produces up to **1000 ElectricCharge per second at full throttle**
- Scales linearly with throttle

**Power Constraints**
- Engines do **not** consume ElectricCharge
- Thrust is never gated by ElectricCharge
- Power generation is entirely reactor-side

---

### Engine Switching Model

- All engines use **exactly two modes**
- Mode switching is handled via `MultiModeEngine`
- Each mode has its own `ModuleEnginesFX`
- B9PartSwitch and similar frameworks are **not used**

---

### Propulsion Resources

All engines use the same propellant pairing:

- **Water** — reaction mass
- **FusionPellets** — fusion reactor fuel

**Consumption Rule**
- Fixed across all engines and modes:
  - **Water : FusionPellets = 1000 : 1**
- Higher-thrust modes consume *more* of both resources but never change ratios

---

### Mode Behavior Definitions

**Cruise**
- Baseline operating mode
- Best efficiency for that engine size
- Primary mode for long-duration burns
- Poor atmospheric performance

**Overburn**
- High-thrust mode
- Same propellant ratio as Cruise
- Lower Isp than Cruise
- Intended for rapid acceleration and combat maneuvers

**Atmo Ejection**
- Atmospheric-capable mode
- Lowest overall efficiency
- Flatter atmosphereCurve
- Vacuum Isp improves slightly over sea level
- Still significantly less efficient than Cruise or Overburn
- Reduced heat production

---

### Efficiency Scaling Rule

- Larger Epstein Drives are **incrementally more vacuum-efficient**
- Improvements are modest (not exponential)
- Represents improved magnetic containment and nozzle geometry at scale

---

## 4️⃣ Propulsion Numbers (Thrust & Isp Bands)

This section defines **relative thrust and efficiency relationships**, not final values.
Absolute numbers should be derived later to fit gameplay scale.

---

### Thrust Philosophy

- Epstein Drives are **orders of magnitude more powerful** than stock KSP engines
- Continuous thrust is expected
- Overburn increases thrust, not efficiency
- Thrust scales primarily by **engine class**, not by mode

---

### Relative Thrust Bands

| Engine Class | Cruise Thrust | Overburn Thrust | Atmo Ejection Thrust |
|-------------|---------------|-----------------|----------------------|
| Frigate     | Low–Medium    | N/A             | Medium               |
| Cruiser     | Medium–High   | High            | N/A                  |
| Battleship  | Very High     | Extreme         | N/A                  |

**Notes**
- Atmo Ejection thrust must support controlled descent and ascent
- Battleship Overburn should feel dramatic and unmistakable

---

### Mode Thrust Relationships

- **Cruise → Overburn**
  - Overburn thrust ≈ **1.3× – 1.6× Cruise thrust**
  - Propellant ratios remain unchanged

- **Cruise → Atmo Ejection**
  - Thrust may be similar or slightly lower
  - Efficiency loss is the primary tradeoff

---

### Isp Philosophy

- Epstein Drives are highly efficient torch drives
- Isp values greatly exceed stock engines
- Atmospheric efficiency is intentionally poor except where supported

---

### Relative Isp Bands (Vacuum)

| Engine Class | Cruise Isp | Overburn Isp | Atmo Ejection Isp |
|-------------|------------|--------------|-------------------|
| Frigate     | High       | N/A          | Low               |
| Cruiser     | Higher     | Medium       | N/A               |
| Battleship  | Highest    | Medium–High  | N/A               |

**Rules**
- Each size tier gains roughly **+5–15% vacuum Isp**
- Overburn Isp is always lower than Cruise
- Atmo Ejection Isp is always the lowest

---

### Atmosphere Curve Shape Rules

**Cruise**
- Steep efficiency drop in atmosphere
- Optimized for vacuum

**Overburn**
- Similar curve to Cruise
- Overall lower efficiency

**Atmo Ejection**
- Flatter curve
- Sea-level Isp closer to vacuum Isp
- Vacuum improvement remains modest

---

### Heat Production Guidelines

- Cruise: moderate heatProduction
- Overburn: high heatProduction
- Atmo Ejection: reduced heatProduction

Heat exists only for stock KSP compatibility and effects feedback.
No active thermal management is required.

---

### Design Guardrails

- No engine should obsolete another within the mod
- Frigate engines must never outperform Cruiser engines in vacuum
- Atmospheric capability remains exclusive to Frigate-class engines
- Overburn must feel powerful but inefficient

---

## 5️⃣ Implementation Constraints & Module Usage

This section defines **hard implementation rules**. These are non-negotiable.

---

### Allowed KSP Modules

Only the following stock modules may be used:

- `ModuleEnginesFX`
- `MultiModeEngine`
- `ModuleGenerator`
- `ModuleAlternator`
- `ModuleSurfaceFX`
- `ModuleGimbal` (optional)
- `ModulePartVariants` (visual-only)

No plugins or custom PartModules are permitted.

---

### Engine Configuration Rules

- Each engine part:
  - Uses **exactly two modes**
  - Uses **one `MultiModeEngine`**
  - Uses **two `ModuleEnginesFX` blocks**
- Modes may differ only by:
  - Thrust
  - Isp curves
  - heatProduction
  - Visual/audio effects

---

### Power System Rules

- Power generation uses:
  - `ModuleGenerator` (100 EC/s, always on)
  - `ModuleAlternator` (1000 EC/s at full throttle)
- Engines do **not** consume ElectricCharge
- ElectricCharge never gates thrust

---

### Thermal Rules

- Heat is implemented only via `heatProduction`
- No use of:
  - WasteHeat
  - Radiators
  - Thermal power resources

---

### Resource Dependency Rules

- Community Resource Pack (CRP) is required for:
  - `Water`
  - `FusionPellets`
- Optional warning patches may be included if CRP is missing

---

### Compatibility & Scope

- No modification of stock resources
- No patches affecting other mods
- No assumptions about life support systems
- No interaction with KSPI-E, Near Future, or RealFuels

---

### Naming & Organization Rules

- Part internal names:
  - `EpsteinDrive_Frigate`
  - `EpsteinDrive_Cruiser`
  - `EpsteinDrive_Battleship`
- All configs live under:
  - `GameData/EpsteinDrive/`
- No legacy naming conventions

---

### Explicit Non-Goals

- No balance relative to stock engines
- No realism enforcement beyond Expanse-inspired flavor
- No player-facing configuration complexity
- No hidden mechanics or failure modes
