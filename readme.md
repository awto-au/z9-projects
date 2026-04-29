
- Note re projhect UUIDs

- IMPORTANT: avoid duplicate schematic UUIDs across shared files

If a shared sheet is copied/renamed (for example in `z9-shared/`), make sure the copied `.kicad_sch` does not keep the same top-level `(uuid "...")` as another active sheet file.

Duplicate UUIDs can cause confusing behavior: sheets may fail to load correctly, references can point to the wrong sheet content, and project/sheet-instance mapping can become unstable.

Safe practice when duplicating a sheet:

- duplicate the file
- open it in KiCad and save-as / resave so a new UUID is generated, or manually replace the top-level UUID with a new one
- re-open the parent project and verify the `Sheetfile` path and `sheet_instances` resolve to the intended file

## Shared Sheet Architecture (as of 2026-04-29)

This repository uses a mixed architecture where core modules are shared across multiple projects via `z9-shared/`, while project-specific functionality remains local.

### Truly Shared Sheets (common UUID across projects)

These sheets have the same top-level UUID and represent unified modules used by multiple projects:

| Sheet Name | UUID | Projects Using | Purpose |
|---|---|---|---|
| shared_usbc.kicad_sch | `1f11bca8-5815-40a7-b8ee-263189e52167` | PDM, console, core | USB Type-C connectivity & power |
| shared_isolated_can.kicad_sch | `01fd732b-d28d-4591-958a-cdee10605b4f` | PDM, console, core | Isolated CAN bus interface |
| shared_ism330dhcx.kicad_sch | `c1bd34f9-1a2e-4e3d-a058-d6dd5157e270` | PDM, console, core | 6-axis IMU / accelerometer |
| shared_buzzer.kicad_sch | `58de86fb-6cef-462e-b9e3-6fa5d0bb0e90` | PDM, console, core | Audio/feedback buzzer |

### Project-Specific or Diverged Sheets

These sheets exist with different UUIDs/content across projects:

| Sheet Name | PDM UUID | Console UUID | Status |
|---|---|---|---|
| stm32.kicad_sch | `6cdc1f96-2079-4699-9b8b-64cd08d3349d` (33K lines) | `4182fd16-8ac6-4c4f-8a6b-17b87a3a1318` (26K lines) | ✅ DE-DUPLICATED — assigned unique UUIDs after significant content divergence |
| connectors.kicad_sch | `62522fec-fb26-4067-a233-5bb78faa120b` | `ff807900-38a7-468c-8c88-36815649564b` | Different modules, project-specific |
| untitled.kicad_sch (power) | `4ea6699b-0436-432c-816e-7adf1044fb41` | `f4d1610f-a9d7-400f-8201-07e24024a234` | Different modules, project-specific |

#### stm32 Divergence Details (resolved 2026-04-29)

PDM and console both previously referenced stm32.kicad_sch with the same UUID (`b85e7fcc-fcb8-4f3f-b9d9-a567574ce4fb`), but the files had diverged significantly:

**PDM stm32 (33K lines, older 2022-04-10 metadata):**
- Comprehensive embedded symbol library (JLCPCB capacitors, resistors)
- ST connector types (STDC14, TagConnect)
- Crystal oscillators (8MHz)
- Specialized connectors (DF40HC3.0-100DS-0.4V51)
- More detailed layout/definitions
- Metadata: "title l8", "company DNT"

**Console stm32 (26K lines, newer 2026 metadata):**
- Simplified schematic focused on MCU core
- Test points instead of dedicated connectors
- Metadata: "company Zero9 - 0-9.au", "Some schematic is shared"

**Resolution:**
- PDM stm32 now UUID: `6cdc1f96-2079-4699-9b8b-64cd08d3349d`
- Console stm32 now UUID: `4182fd16-8ac6-4c4f-8a6b-17b87a3a1318`

These are now independent modules that should not be assumed to be interchangeable. Future maintainers should treat them as separate designs and explicitly sync if unification is desired.

### Console-Only Sheets

- shared_usbc_esp32.kicad_sch (`7f987ba3-810e-4341-b9d4-ad527c98f4c1`) — ESP32 variant of USB-C module

### Core Design Structure (rebuilt 2026-04-29)

Core is now a hierarchical module that references only the truly shared sheets in `z9-shared/`:

```
z9-core.kicad_sch (root)
  ├── z9-shared/shared_usbc.kicad_sch
  ├── z9-shared/shared_isolated_can.kicad_sch
  ├── z9-shared/shared_ism330dhcx.kicad_sch
  └── z9-shared/shared_buzzer.kicad_sch
```

### UUID Deduplication (completed 2026-04-29)

Several shared files had duplicate UUIDs that have since been fixed:

| File | Old UUID | New UUID | Reason |
|---|---|---|---|
| usbc.kicad_sch | `1f11bca8-5815-40a7-b8ee-263189e52167` | `fa8ae3e9-6b30-442f-8590-d93320e7db7b` | Copy of shared_usbc; renamed to prevent collision |
| shared.kicad_sch | `1f11bca8-5815-40a7-b8ee-263189e52167` | `6e9f9a04-b3f3-4667-a147-eb1a8952cf4f` | Duplicate of shared_usbc; fixed |
| shared_5V_LMR54410.kicad_sch | `a7202c14-0f41-48d8-b26c-392947ef0c55` | `c6e889f8-d252-47bb-acac-ee7397c7ab1d` | Duplicate of shared_5V_LMR14206; fixed |

stored in [project name].kicad_pro

snup ####################
  "sheets": [
    [
      "8d063f79-9282-4820-bcf4-1ff3c006cf08",
      "Root"
    ],

then 'root' schmeatic 
snip ############
(kicad_sch
	(version 20250114)
	(generator "eeschema")
	(generator_version "9.0")
	(uuid "8d063f79-9282-4820-bcf4-1ff3c006cf08")
	(paper "A3")
	(title_block
		(date "2024-05-30")
	)


-- project uuids
old pdm 8d063f79-9282-4820-bcf4-1ff3c006cf08
new pdm b643db5c-578e-4b48-aafc-cc4190db582a


shared 

new 0d832c56-2071-4d02-b574-dc412c8c5a75

8d063f79-9282-4820-bcf4-1ff3c006cf08


- KiCad library setup notes

KiCad path management is not the only setting required for these projects.
For the PDM project, the symbol and footprint library tables also need to be configured so ERC can resolve the custom libraries used by the design.

Required custom libraries:

- `dnt_global` = my global symbols
- `dnt` = my global footprints

What was found:

- Project-local footprint table: [z9-pdm/fp-lib-table](/home/dan/git/z9-projects/z9-pdm/fp-lib-table)
- Project-local symbol table: [z9-pdm/sym-lib-table](/home/dan/git/z9-projects/z9-pdm/sym-lib-table)
- Global KiCad footprint table: `/home/dan/.config/kicad/10.0/fp-lib-table`
- Global KiCad symbol table: `/home/dan/.config/kicad/10.0/sym-lib-table`
- The PDM project tables were effectively empty, so KiCad was falling back to global configuration.
- `dnt_global` was configured globally on this machine at `/home/dan/git/dnt.kicad.libs/symbols/dnt_global.kicad_sym`
- `dnt` was configured globally on this machine at `/home/dan/git/dnt.kicad.libs/footprints/dnt.pretty`
- KiCad 10 third-party alternate library content was found under `/home/dan/.local/share/kicad/10.0/3rdparty/`
- That third-party content includes AKL resistor/capacitor/diode libraries, but not exact `PCM_JLCPCB*` library names in the global tables.

Current project-local fix added for PDM:

- `z9-pdm/sym-lib-table` now maps `dnt_global`
- `z9-pdm/sym-lib-table` now maps `PCM_Espressif` to `dnt_global.kicad_sym` because the required ESP32-S3 symbol was already embedded there
- `z9-pdm/fp-lib-table` now maps `dnt`
- `z9-pdm/fp-lib-table` now maps `PCM_Espressif` to `dnt.kicad.libs/footprints/PCM_Espressif.pretty`
- `z9-pdm/sym-lib-table` also maps `PCM_JLCPCB-Resistors`, `PCM_JLCPCB-Capacitors`, and `PCM_JLCPCB-Diodes` to get ERC closer to usable
- `z9-pdm/fp-lib-table` also maps `PCM_JLCPCB`
- The required Espressif footprint files `ESP32-S3-WROOM-1.kicad_mod` and `ESP32-S3-WROOM-1U.kicad_mod` were copied from `https://github.com/espressif/kicad-libraries` into `dnt.kicad.libs/footprints/PCM_Espressif.pretty`
- The matching Espressif STEP models were copied into `dnt.kicad.libs/models/`

Result:

- ERC on PDM dropped from `1 error, 675 warnings` to `1 error, 229 warnings`
- Most of the missing-library noise was removed
- `PCM_Espressif` now resolves from repo-local library content instead of requiring a separate KiCad add-on install on this machine
- Remaining warnings show that some `PCM_JLCPCB*` symbols and footprints still do not exist under the currently available local libraries, so installing or restoring the original PCM/JLC libraries is still needed for a clean ERC result