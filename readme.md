
- Note re projhect UUIDs

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