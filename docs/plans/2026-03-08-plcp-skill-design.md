# PLCP Skill Design

## Goal

Generate high-accuracy ladder logic (LD) programs for industrial
automation, targeting controls engineers who program PLCs daily.

## Architecture

Single skill (`/plcp:start`) with built-in review phase.
No separate reviewer command.

### Workflow Phases

```
GATHER → ARCHITECTURE → GENERATE → REVIEW
```

### Phase 1: GATHER

- Accept user's goal + sequence of operations or P&ID (PDF, text,
  image)
- If neither provided, prompt for: system purpose, equipment list,
  control requirements
- Ask user to specify PLC platform (Rockwell, Siemens, Automation
  Direct, etc.)
- Extract all equipment tags verbatim from user's documentation

### Phase 2: ARCHITECTURE

Present for user approval before generating logic:

- System overview (all control routines identified)
- I/O mapping table (user's tags → platform-specific address
  placeholders for user to fill)
- Interlock cross-reference (which safety items affect which
  equipment)
- Dedicated safety routine outline, separate from process logic

### Phase 3: GENERATE

After architecture approval, generate ladder logic in groups:

- Dedicated safety/interlock routine first
- Then process routines in logical groups
- Each rung gets a short description comment
- Alarms: simple command vs status mismatch, configurable delay
  timer, latched until acknowledged
- Output in `.md` (default) or `.L5X` on request

#### Output Format

Ladder logic uses standard instruction mnemonics in structured
text-table format matching Logix Designer export conventions:

```
Routine: CGSP1 — Secondary Pump 1
Total rungs: 11

Rung 0: NOP
  [separator]

Rung 1: AUTO / MANUAL SELECTION
  XIC  CGSP1.Control.Auto_PB
  OTE  CGSP1.Status.Auto
  ────
  XIO  CGSP1.Control.Manual_PB
  OTE  CGSP1.Status.Auto
```

For non-Rockwell platforms, translate instruction mnemonics to
the target platform's equivalent.

### Phase 4: REVIEW

Built-in self-review checklist before presenting final output:

- All SOO requirements mapped to rungs
- Safety interlocks properly cross-referenced
- I/O table complete
- Tag names match user's documentation exactly
- Alarm points covered for all commanded equipment

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| IEC languages | LD only | User's domain, broadest audience |
| Output formats | `.md` + `.L5X` | Text-based, zero dependencies |
| Tag convention | User's tags verbatim + I/O table | Respects engineer's naming |
| PLC platform | Always ask | No assumptions on hardware |
| System scope | Architecture-first, grouped routines | Manageable review chunks |
| Safety logic | Dedicated separate routine | Clean separation |
| Alarms | Cmd/status + delay + latch | Covers 80% of use cases |
| Rung comments | Short descriptions | Controls engineers audience |
| Target user | Experienced controls engineers | Fast output, no hand-holding |

## Scope Exclusions

- No FBD, ST, SFC, or IL generation
- No software installation or dependencies
- No HMI/SCADA screen generation
- No ISA-18.2 full alarm management
- No electrical schematics or wiring diagrams

## File Structure

```
plcp-skill/
  .claude-plugin/
    plugin.json
  skills/
    plcp-start/
      SKILL.md
  commands/
    start.md
```
