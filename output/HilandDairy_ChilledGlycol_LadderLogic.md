# Hiland Dairy — Chilled Glycol System Ladder Logic

**Project:** 23042 Hiland Dairy Plant Expansion
**Address:** 200 N Fuller Ave, Tyler, TX 75702
**Drawing Reference:** M6.1 — Chilled Glycol System Schematic
**PLC Platform:** Rockwell CompactLogix (Logix Designer)

---

## System Overview

The chilled glycol system consists of a primary loop (constant flow)
and secondary loop (variable flow). The system is active at all times.
A tertiary loop supplies 34°F glycol for HVAC usage.

### Equipment

| Tag | Description | HP | Details |
|-----|-------------|----|---------|
| CH-1 | Centrifugal Chiller | — | 500 ton |
| CGPP-1 | Primary Pump | 25 | 1,594 GPM / 45' HD, VFD (balance only) |
| CGSP-1 | Secondary Pump 1 | 30 | 650 GPM / 115' HD, VFD |
| CGSP-2 | Secondary Pump 2 | 30 | 650 GPM / 115' HD, VFD |
| CGTP-1 | Tertiary Pump | 7.5 | 240 GPM / 60' HD, constant speed |
| CWP-1 | Condenser Water Pump | 25 | 1,512 GPM / 50' HD |
| CT-1 | Cooling Tower | — | 7497 MBH, BAC #S3E, VFD fan |
| CV-1 | 2-Way Modulating Ball Valve | — | Belimo, Cv=39 |
| TT-1 | Temp Transmitter | — | Distribution pump discharge |
| TT-2 | Temp Transmitter | — | Tertiary loop supply |
| BT-1 | Buffer Tank | — | 1,040 gal |
| ET-4 | Expansion Tank | — | 132 gal, bladder |

---

## I/O Mapping Table

| Tag | Description | I/O Type | Address |
|-----|-------------|----------|---------|
| TT-1 | Distribution pump discharge temp | AI | ________ |
| TT-2 | Tertiary loop supply temp | AI | ________ |
| CT1_VIB_CO_SW | CT-1 vibration cutout switch | DI | ________ |
| CGPP1_FB | Primary pump feedback | DI | ________ |
| CGSP1_FB | Secondary pump 1 feedback | DI | ________ |
| CGSP2_FB | Secondary pump 2 feedback | DI | ________ |
| CGTP1_FB | Tertiary pump feedback | DI | ________ |
| CWP1_FB | Condenser water pump feedback | DI | ________ |
| CG_MCV1.Inp_OpenLS | CW isolation valve open LS | DI | ________ |
| CG_MCV1.Inp_ClosedLS | CW isolation valve closed LS | DI | ________ |
| CG_MCV2.Inp_OpenLS | CT inlet valve open LS | DI | ________ |
| CG_MCV2.Inp_ClosedLS | CT inlet valve closed LS | DI | ________ |
| CG_MCV3.Inp_OpenLS | CW bypass valve open LS | DI | ________ |
| CG_MCV3.Inp_ClosedLS | CW bypass valve closed LS | DI | ________ |
| CG_DPT1 | Distribution system DP | AI | ________ |
| CV1_POS_FDBK | CV-1 position feedback | AI | ________ |
| CGPP1_MS | Primary pump motor start | DO | ________ |
| CGSP1_MS | Secondary pump 1 motor start | DO | ________ |
| CGSP2_MS | Secondary pump 2 motor start | DO | ________ |
| CGTP1_MS | Tertiary pump motor start | DO | ________ |
| CWP1_MS | CW pump motor start | DO | ________ |
| CG_MCV1.Out_Open | CW isolation valve open cmd | DO | ________ |
| CG_MCV2.Out_Open | CT inlet valve open cmd | DO | ________ |
| CG_MCV3.Out_Open | CW bypass valve open cmd | DO | ________ |
| CV1_AO | CV-1 analog output (4-20mA) | AO | ________ |
| CGPP1_VFD | Primary pump VFD (Ethernet) | VFD | ________ |
| CGSP1_VFD | Sec pump 1 VFD (Ethernet) | VFD | ________ |
| CGSP2_VFD | Sec pump 2 VFD (Ethernet) | VFD | ________ |
| CT1_FAN_VFD | CT fan VFD speed ref/fdbk | AO/AI | ________ |

---

## Safety / Interlock Cross-Reference

| Interlock | Source | Affected Equipment | Action |
|-----------|--------|--------------------|--------|
| CT1_VIB_CO_SW | CT-1 vibration switch | CT1 Fan VFD | Disable fan, alarm |
| Chiller fault | CH-1 status | CGPP1, CWP1, MCV1, MCV2 | Stop associated equip |
| CW isolation not open | CG_MCV1 | CH-1 | Prevent chiller enable |
| CT inlet not open | CG_MCV2 | CH-1 | Prevent chiller enable |
| Both CGSP faulted | CGSP1 + CGSP2 | System | Alarm, no DP control |

---

## Program Structure

```
MainTask
├── Buffer_IO
│   ├── Main (JSR dispatcher)
│   ├── Slot_IF8
│   ├── Slot_OF4
│   ├── Slot_IQ16
│   ├── Slot_OB16
│   └── VFD
└── Process_Control
    ├── Main (JSR dispatcher)
    ├── Safety_Interlocks
    ├── Chiller_Control
    ├── CGPP1
    ├── CWP1
    ├── CGSP1
    ├── CGSP2
    ├── CGSP_LeadLag
    ├── CGTP1
    ├── CT1_Fan
    ├── CW_Bypass
    ├── Tertiary_CV1
    └── Alarms
```

---

## PlantPAx AOIs Used

| AOI | Applied To |
|-----|-----------|
| P_PF755 | CGPP1_VFD, CGSP1_VFD, CGSP2_VFD |
| P_VSD | CT1_FAN_VFD |
| P_Motor | CGTP1, CWP1 |
| P_ValveMO | CG_MCV1, CG_MCV2, CG_MCV3 |
| P_ValveC | CV1_BELMIO_AO |
| P_AIn | TT1_GLYCOL, CGTP1_TT2, CG_DPT1 |
| P_PIDE | CGSP_DP_PID, Tertiary_TT2_PID, CT1_CW_PID |
| P_Intlk | Chiller interlocks |
| P_Alarm | Per-device cmd/status alarms |

---

## JSR Dispatchers

### Buffer_IO / Main

```
Routine: Buffer_IO/Main
Total rungs: 6

Rung 0: JSR  Slot_IF8
Rung 1: JSR  Slot_OF4
Rung 2: JSR  Slot_IQ16
Rung 3: JSR  Slot_OB16
Rung 4: JSR  VFD
Rung 5: NOP
```

### Process_Control / Main

```
Routine: Process_Control/Main
Total rungs: 12

Rung 0:  JSR  Safety_Interlocks
Rung 1:  JSR  Chiller_Control
Rung 2:  JSR  CGPP1
Rung 3:  JSR  CWP1
Rung 4:  JSR  CGSP1
Rung 5:  JSR  CGSP2
Rung 6:  JSR  CGSP_LeadLag
Rung 7:  JSR  CGTP1
Rung 8:  JSR  CT1_Fan
Rung 9:  JSR  CW_Bypass
Rung 10: JSR  Tertiary_CV1
Rung 11: JSR  Alarms
```

---

## Ladder Logic — Safety / Interlocks

```
Routine: Safety_Interlocks — System Safety and Interlocks
Total rungs: 8

Rung 0: NOP

Rung 1: CT-1 VIBRATION CUTOUT
  XIC  CT1_VIB_CO_SW
  OTL  Safety.CT1_Vib_Trip
  ────
  XIC  System.Control.Reset
  OTU  Safety.CT1_Vib_Trip

Rung 2: CT-1 FAN INTERLOCK
  XIO  Safety.CT1_Vib_Trip
  OTE  Safety.CT1_Fan_OK

Rung 3: CHILLER FLOW PERMISSIVE
  XIC  CG_MCV1.Sts_Opened
  XIC  CG_MCV2.Sts_Opened
  OTE  Safety.CH1_Flow_OK

Rung 4: CHILLER ENABLE PERMISSIVE
  XIC  Safety.CH1_Flow_OK
  XIC  CGPP1.Status.Running
  XIC  CWP1.Status.Running
  OTE  Safety.CH1_Enable_OK

Rung 5: CHILLER FAULT TRIP
  XIC  CH1.Status.Faulted
  OTL  Safety.CH1_Fault_Trip
  ────
  XIC  System.Control.Reset
  OTU  Safety.CH1_Fault_Trip

Rung 6: CHILLER CASCADE INTERLOCK
  XIO  Safety.CH1_Fault_Trip
  XIC  Safety.CH1_Enable_OK
  OTE  Safety.CH1_Run_OK

Rung 7: SYSTEM GLOBAL RESET
  XIC  System.Control.HMI_Reset_PB
  OTE  System.Control.Reset
```

---

## Ladder Logic — Chiller Control

```
Routine: Chiller_Control — CH-1 Enable and Cascade Sequencing
Total rungs: 10

Rung 0: NOP

Rung 1: CAPACITY CONTROL — CHILLER ENABLE
  Source A: TT1_GLYCOL.Val
  Source B: System.SP.ChilledGlycol_Temp
  GRT
  OTE  CH1.Control.Enable_Req

Rung 2: CHILLER START SEQUENCE — STEP 1: OPEN CW ISOLATION
  XIC  CH1.Control.Enable_Req
  XIC  Safety.CH1_Run_OK
  OTE  CG_MCV1.PCmd_Open

Rung 3: CHILLER START SEQUENCE — STEP 2: OPEN CT INLET
  XIC  CH1.Control.Enable_Req
  XIC  Safety.CH1_Run_OK
  OTE  CG_MCV2.PCmd_Open

Rung 4: CHILLER START SEQUENCE — STEP 3: START PRIMARY PUMP
  XIC  CH1.Control.Enable_Req
  XIC  Safety.CH1_Run_OK
  XIC  CG_MCV1.Sts_Opened
  OTE  CGPP1.Control.Start_Req

Rung 5: CHILLER START SEQUENCE — STEP 4: START CW PUMP
  XIC  CH1.Control.Enable_Req
  XIC  Safety.CH1_Run_OK
  XIC  CG_MCV1.Sts_Opened
  OTE  CWP1.Control.Start_Req

Rung 6: CHILLER START SEQUENCE — STEP 5: ENABLE CHILLER
  XIC  CH1.Control.Enable_Req
  XIC  Safety.CH1_Run_OK
  XIC  CGPP1.Status.Running
  XIC  CWP1.Status.Running
  OTE  CH1.Control.Start_Cmd

Rung 7: CHILLER SHUTDOWN SEQUENCE
  XIO  CH1.Control.Enable_Req
  XIC  CH1.Status.Running
  OTE  CH1.Control.Stop_Cmd

Rung 8: CHILLER SHUTDOWN — STOP ASSOCIATED EQUIPMENT
  XIO  CH1.Control.Enable_Req
  XIO  CH1.Status.Running
  OTE  CGPP1.Control.Stop_Cmd
  OTE  CWP1.Control.Stop_Cmd
  OTE  CG_MCV1.PCmd_Close
  OTE  CG_MCV2.PCmd_Close

Rung 9: CHILLER RUNNING STATUS
  XIC  CH1_FB
  OTE  CH1.Status.Running
```

---

## Ladder Logic — CGPP1 (Primary Pump)

```
Routine: CGPP1 — Primary Pump
Total rungs: 12

Rung 0: NOP

Rung 1: AUTO / MANUAL SELECTION
  XIC  CGPP1.Control.Auto_PB
  OTE  CGPP1.Status.Auto
  ────
  XIO  CGPP1.Control.Manual_PB
  OTE  CGPP1.Status.Auto

Rung 2: AUTO / MANUAL INVERSE
  XIO  CGPP1.Status.Auto
  OTE  CGPP1.Status.Manual

Rung 3: FAULT RESET
  XIC  CGPP1.Control.HMI_Reset_PB
  OTE  CGPP1.Control.Reset
  ────
  XIC  System.Control.Reset
  OTE  CGPP1.Control.Reset

Rung 4: FAULT HANDLING
  [series]
    XIC  CGPP1.Control.Start_Req
    XIC  CGPP1.Status.Permissives_OK
  [parallel]
    XIC  CGPP1.Control.Manual_Start
  [series]
    XIO  CGPP1.Status.Running
    XIO  CGPP1.Status.Faulted
    XIO  CGPP1.Control.Reset
  TON  CGPP1.Fault_TMR  Preset:10000  Accum:0
  ────
  XIC  CGPP1.Fault_TMR.DN
  OTE  CGPP1.Status.Faulted

Rung 5: PERMISSIVES
  XIO  CGPP1.Status.Faulted
  OTE  CGPP1.Status.Permissives_OK

Rung 6: MANUAL START / STOP
  [series]
    XIC  CGPP1.Status.Manual
    XIC  CGPP1.Control.HMI_Start_PB
  OTE  CGPP1.Control.Manual_Start
  ────
  [series]
    XIO  CGPP1.Control.Stop_Cmd
  OTE  CGPP1.Control.Manual_Start  (seal-in)

Rung 7: AUTO START / STOP
  [series]
    XIC  CGPP1.Status.Auto
    XIC  CGPP1.Control.HMI_Start_PB
  OTE  CGPP1.Control.Start_Req
  ────
  [series]
    XIO  CGPP1.Control.Stop_Cmd
  OTE  CGPP1.Control.Start_Req  (seal-in)

Rung 8: MOTOR START OUTPUT
  [parallel]
    [series]
      XIC  CGPP1.Control.Start_Req
      XIC  CGPP1.Status.Permissives_OK
    XIC  CGPP1.Control.Manual_Start
  OTE  CGPP1_MS

Rung 9: STOP COMMAND
  [parallel]
    [series]
      XIC  CGPP1.Status.Auto
      XIC  System.Control.Stop_Cmd
    [series]
      XIC  CGPP1.Status.Manual
      XIC  System.Control.HMI_Stop_PB
    XIC  CGPP1.Control.HMI_Stop_PB
    XIC  CGPP1.Status.Faulted
  OTE  CGPP1.Control.Stop_Cmd
  ────
  XIO  CGPP1.Control.Reset
  OTE  CGPP1.Control.Stop_Cmd  (seal-in)

Rung 10: RUNNING STATUS
  XIC  CGPP1_FB
  OTE  CGPP1.Status.Running

Rung 11: VFD START FORWARD
  XIC  CGPP1_MS
  OTE  CGPP1_VFD.PCmd_StartFwd
```

---

## Ladder Logic — CWP1 (Condenser Water Pump)

```
Routine: CWP1 — Condenser Water Pump
Total rungs: 11

Rung 0: NOP

Rung 1: AUTO / MANUAL SELECTION
  XIC  CWP1.Control.Auto_PB
  OTE  CWP1.Status.Auto
  ────
  XIO  CWP1.Control.Manual_PB
  OTE  CWP1.Status.Auto

Rung 2: AUTO / MANUAL INVERSE
  XIO  CWP1.Status.Auto
  OTE  CWP1.Status.Manual

Rung 3: FAULT RESET
  XIC  CWP1.Control.HMI_Reset_PB
  OTE  CWP1.Control.Reset
  ────
  XIC  System.Control.Reset
  OTE  CWP1.Control.Reset

Rung 4: FAULT HANDLING
  [series]
    XIC  CWP1.Control.Start_Req
    XIC  CWP1.Status.Permissives_OK
  [parallel]
    XIC  CWP1.Control.Manual_Start
  [series]
    XIO  CWP1.Status.Running
    XIO  CWP1.Status.Faulted
    XIO  CWP1.Control.Reset
  TON  CWP1.Fault_TMR  Preset:10000  Accum:0
  ────
  XIC  CWP1.Fault_TMR.DN
  OTE  CWP1.Status.Faulted

Rung 5: PERMISSIVES
  XIO  CWP1.Status.Faulted
  OTE  CWP1.Status.Permissives_OK

Rung 6: MANUAL START / STOP
  [series]
    XIC  CWP1.Status.Manual
    XIC  CWP1.Control.HMI_Start_PB
  OTE  CWP1.Control.Manual_Start
  ────
  [series]
    XIO  CWP1.Control.Stop_Cmd
  OTE  CWP1.Control.Manual_Start  (seal-in)

Rung 7: AUTO START / STOP
  [series]
    XIC  CWP1.Status.Auto
    XIC  CWP1.Control.HMI_Start_PB
  OTE  CWP1.Control.Start_Req
  ────
  [series]
    XIO  CWP1.Control.Stop_Cmd
  OTE  CWP1.Control.Start_Req  (seal-in)

Rung 8: MOTOR START OUTPUT
  [parallel]
    [series]
      XIC  CWP1.Control.Start_Req
      XIC  CWP1.Status.Permissives_OK
    XIC  CWP1.Control.Manual_Start
  OTE  CWP1_MS

Rung 9: STOP COMMAND
  [parallel]
    [series]
      XIC  CWP1.Status.Auto
      XIC  System.Control.Stop_Cmd
    [series]
      XIC  CWP1.Status.Manual
      XIC  System.Control.HMI_Stop_PB
    XIC  CWP1.Control.HMI_Stop_PB
    XIC  CWP1.Status.Faulted
  OTE  CWP1.Control.Stop_Cmd
  ────
  XIO  CWP1.Control.Reset
  OTE  CWP1.Control.Stop_Cmd  (seal-in)

Rung 10: RUNNING STATUS
  XIC  CWP1_FB
  OTE  CWP1.Status.Running
```

---

## Ladder Logic — CGSP1 (Secondary Pump 1)

```
Routine: CGSP1 — Secondary Pump 1
Total rungs: 12

Rung 0: NOP

Rung 1: AUTO / MANUAL SELECTION
  XIC  CGSP1.Control.Auto_PB
  OTE  CGSP1.Status.Auto
  ────
  XIO  CGSP1.Control.Manual_PB
  OTE  CGSP1.Status.Auto

Rung 2: AUTO / MANUAL INVERSE
  XIO  CGSP1.Status.Auto
  OTE  CGSP1.Status.Manual

Rung 3: FAULT RESET
  XIC  CGSP1.Control.HMI_Reset_PB
  OTE  CGSP1.Control.Reset
  ────
  XIC  System.Control.Reset
  OTE  CGSP1.Control.Reset

Rung 4: FAULT HANDLING
  [series]
    XIC  CGSP1.Control.Start_Req
    XIC  CGSP1.Status.Permissives_OK
  [parallel]
    XIC  CGSP1.Control.Manual_Start
  [series]
    XIO  CGSP1.Status.Running
    XIO  CGSP1.Status.Faulted
    XIO  CGSP1.Control.Reset
  TON  CGSP1.Fault_TMR  Preset:10000  Accum:0
  ────
  XIC  CGSP1.Fault_TMR.DN
  OTE  CGSP1.Status.Faulted

Rung 5: PERMISSIVES
  XIO  CGSP1.Status.Faulted
  OTE  CGSP1.Status.Permissives_OK

Rung 6: MANUAL START / STOP
  [series]
    XIC  CGSP1.Status.Manual
    XIC  CGSP1.Control.HMI_Start_PB
  OTE  CGSP1.Control.Manual_Start
  ────
  [series]
    XIO  CGSP1.Control.Stop_Cmd
  OTE  CGSP1.Control.Manual_Start  (seal-in)

Rung 7: AUTO START / STOP
  [series]
    XIC  CGSP1.Status.Auto
    XIC  CGSP1.Control.HMI_Start_PB
  OTE  CGSP1.Control.Start_Req
  ────
  [series]
    XIO  CGSP1.Control.Stop_Cmd
  OTE  CGSP1.Control.Start_Req  (seal-in)

Rung 8: MOTOR START OUTPUT
  [parallel]
    [series]
      XIC  CGSP1.Control.Start_Req
      XIC  CGSP1.Status.Permissives_OK
    XIC  CGSP1.Control.Manual_Start
  OTE  CGSP1_MS

Rung 9: STOP COMMAND
  [parallel]
    [series]
      XIC  CGSP1.Status.Auto
      XIC  System.Control.Stop_Cmd
    [series]
      XIC  CGSP1.Status.Manual
      XIC  System.Control.HMI_Stop_PB
    XIC  CGSP1.Control.HMI_Stop_PB
    XIC  CGSP1.Status.Faulted
  OTE  CGSP1.Control.Stop_Cmd
  ────
  XIO  CGSP1.Control.Reset
  OTE  CGSP1.Control.Stop_Cmd  (seal-in)

Rung 10: RUNNING STATUS
  XIC  CGSP1_FB
  OTE  CGSP1.Status.Running

Rung 11: VFD START FORWARD
  [parallel]
    XIC  CGSP1.Control.Start_Req
    XIC  CGSP1.Control.Manual_Start
  OTE  CGSP1_VFD.PCmd_StartFwd
```

---

## Ladder Logic — CGSP2 (Secondary Pump 2)

```
Routine: CGSP2 — Secondary Pump 2
Total rungs: 12

Rung 0: NOP

Rung 1: AUTO / MANUAL SELECTION
  XIC  CGSP2.Control.Auto_PB
  OTE  CGSP2.Status.Auto
  ────
  XIO  CGSP2.Control.Manual_PB
  OTE  CGSP2.Status.Auto

Rung 2: AUTO / MANUAL INVERSE
  XIO  CGSP2.Status.Auto
  OTE  CGSP2.Status.Manual

Rung 3: FAULT RESET
  XIC  CGSP2.Control.HMI_Reset_PB
  OTE  CGSP2.Control.Reset
  ────
  XIC  System.Control.Reset
  OTE  CGSP2.Control.Reset

Rung 4: FAULT HANDLING
  [series]
    XIC  CGSP2.Control.Start_Req
    XIC  CGSP2.Status.Permissives_OK
  [parallel]
    XIC  CGSP2.Control.Manual_Start
  [series]
    XIO  CGSP2.Status.Running
    XIO  CGSP2.Status.Faulted
    XIO  CGSP2.Control.Reset
  TON  CGSP2.Fault_TMR  Preset:10000  Accum:0
  ────
  XIC  CGSP2.Fault_TMR.DN
  OTE  CGSP2.Status.Faulted

Rung 5: PERMISSIVES
  XIO  CGSP2.Status.Faulted
  OTE  CGSP2.Status.Permissives_OK

Rung 6: MANUAL START / STOP
  [series]
    XIC  CGSP2.Status.Manual
    XIC  CGSP2.Control.HMI_Start_PB
  OTE  CGSP2.Control.Manual_Start
  ────
  [series]
    XIO  CGSP2.Control.Stop_Cmd
  OTE  CGSP2.Control.Manual_Start  (seal-in)

Rung 7: AUTO START / STOP
  [series]
    XIC  CGSP2.Status.Auto
    XIC  CGSP2.Control.HMI_Start_PB
  OTE  CGSP2.Control.Start_Req
  ────
  [series]
    XIO  CGSP2.Control.Stop_Cmd
  OTE  CGSP2.Control.Start_Req  (seal-in)

Rung 8: MOTOR START OUTPUT
  [parallel]
    [series]
      XIC  CGSP2.Control.Start_Req
      XIC  CGSP2.Status.Permissives_OK
    XIC  CGSP2.Control.Manual_Start
  OTE  CGSP2_MS

Rung 9: STOP COMMAND
  [parallel]
    [series]
      XIC  CGSP2.Status.Auto
      XIC  System.Control.Stop_Cmd
    [series]
      XIC  CGSP2.Status.Manual
      XIC  System.Control.HMI_Stop_PB
    XIC  CGSP2.Control.HMI_Stop_PB
    XIC  CGSP2.Status.Faulted
  OTE  CGSP2.Control.Stop_Cmd
  ────
  XIO  CGSP2.Control.Reset
  OTE  CGSP2.Control.Stop_Cmd  (seal-in)

Rung 10: RUNNING STATUS
  XIC  CGSP2_FB
  OTE  CGSP2.Status.Running

Rung 11: VFD START FORWARD
  [parallel]
    XIC  CGSP2.Control.Start_Req
    XIC  CGSP2.Control.Manual_Start
  OTE  CGSP2_VFD.PCmd_StartFwd
```

---

## Ladder Logic — CGSP Lead/Lag and DP PID

```
Routine: CGSP_LeadLag — Lead/Lag Selection and DP PID
Total rungs: 12

Rung 0: NOP

Rung 1: WEEKLY LEAD ROTATION TIMER
  TON  CGSP_LL.Rotation_TMR  Preset:604800000  Accum:0
  ────
  XIC  CGSP_LL.Rotation_TMR.DN
  [toggle]
    XIO  CGSP_LL.Lead_Is_SP1
    OTE  CGSP_LL.Lead_Is_SP1
    ────
    XIC  CGSP_LL.Lead_Is_SP1
    OTU  CGSP_LL.Lead_Is_SP1
  RES  CGSP_LL.Rotation_TMR

Rung 2: LEAD PUMP ASSIGNMENT
  XIC  CGSP_LL.Lead_Is_SP1
  OTE  CGSP_LL.CGSP1_Is_Lead
  ────
  XIO  CGSP_LL.Lead_Is_SP1
  OTE  CGSP_LL.CGSP2_Is_Lead

Rung 3: LEAD PUMP START
  XIC  CH1.Status.Running
  [parallel]
    [series]
      XIC  CGSP_LL.CGSP1_Is_Lead
      XIC  CGSP1.Status.Permissives_OK
      OTE  CGSP1.Control.Start_Req
    [series]
      XIC  CGSP_LL.CGSP2_Is_Lead
      XIC  CGSP2.Status.Permissives_OK
      OTE  CGSP2.Control.Start_Req

Rung 4: LAG PUMP ENABLE — HIGH DP DEMAND
  Source A: CGSP_DP_PID.Out_CV
  Source B: CGSP_LL.Lag_Enable_SP
  GRT
  OTE  CGSP_LL.Lag_Enable

Rung 5: LAG PUMP START
  XIC  CGSP_LL.Lag_Enable
  [parallel]
    [series]
      XIC  CGSP_LL.CGSP2_Is_Lead
      XIC  CGSP1.Status.Permissives_OK
      OTE  CGSP1.Control.Start_Req
    [series]
      XIC  CGSP_LL.CGSP1_Is_Lead
      XIC  CGSP2.Status.Permissives_OK
      OTE  CGSP2.Control.Start_Req

Rung 6: DP PID LOOP
  P_PIDE  CGSP_DP_PID
    Inp_PV:   CG_DPT1.Val
    Val_SP:   System.SP.CG_DP
    Out_CV:   CGSP_DP_PID.Out_CV

Rung 7: DP PID TO SPEED REFERENCE
  Source: CGSP_DP_PID.Out_CV
  Dest:   CGSP_LL.Speed_Ref
  MOV

Rung 8: SPEED MATCHING — ALL PUMPS SAME SPEED
  XIC  CGSP1.Status.Running
  Source: CGSP_LL.Speed_Ref
  Dest:   CGSP1_VFD.PSet_SpeedRef
  MOV
  ────
  XIC  CGSP2.Status.Running
  Source: CGSP_LL.Speed_Ref
  Dest:   CGSP2_VFD.PSet_SpeedRef
  MOV

Rung 9: LAG PUMP DISABLE — LOW DP DEMAND
  Source A: CGSP_DP_PID.Out_CV
  Source B: CGSP_LL.Lag_Disable_SP
  LES
  OTU  CGSP_LL.Lag_Enable

Rung 10: LAG PUMP STOP
  XIO  CGSP_LL.Lag_Enable
  [parallel]
    [series]
      XIC  CGSP_LL.CGSP2_Is_Lead
      OTE  CGSP1.Control.Stop_Cmd
    [series]
      XIC  CGSP_LL.CGSP1_Is_Lead
      OTE  CGSP2.Control.Stop_Cmd

Rung 11: BOTH PUMPS FAULTED ALARM
  XIC  CGSP1.Status.Faulted
  XIC  CGSP2.Status.Faulted
  OTL  CGSP_LL.Status.Alarm
```

---

## Ladder Logic — CGTP1 (Tertiary Pump)

```
Routine: CGTP1 — Tertiary Pump (Continuous Run)
Total rungs: 11

Rung 0: NOP

Rung 1: AUTO / MANUAL SELECTION
  XIC  CGTP1.Control.Auto_PB
  OTE  CGTP1.Status.Auto
  ────
  XIO  CGTP1.Control.Manual_PB
  OTE  CGTP1.Status.Auto

Rung 2: AUTO / MANUAL INVERSE
  XIO  CGTP1.Status.Auto
  OTE  CGTP1.Status.Manual

Rung 3: FAULT RESET
  XIC  CGTP1.Control.HMI_Reset_PB
  OTE  CGTP1.Control.Reset
  ────
  XIC  System.Control.Reset
  OTE  CGTP1.Control.Reset

Rung 4: FAULT HANDLING
  [series]
    XIC  CGTP1.Control.Start_Req
    XIC  CGTP1.Status.Permissives_OK
  [parallel]
    XIC  CGTP1.Control.Manual_Start
  [series]
    XIO  CGTP1.Status.Running
    XIO  CGTP1.Status.Faulted
    XIO  CGTP1.Control.Reset
  TON  CGTP1.Fault_TMR  Preset:10000  Accum:0
  ────
  XIC  CGTP1.Fault_TMR.DN
  OTE  CGTP1.Status.Faulted

Rung 5: PERMISSIVES
  XIO  CGTP1.Status.Faulted
  OTE  CGTP1.Status.Permissives_OK

Rung 6: MANUAL START / STOP
  [series]
    XIC  CGTP1.Status.Manual
    XIC  CGTP1.Control.HMI_Start_PB
  OTE  CGTP1.Control.Manual_Start
  ────
  [series]
    XIO  CGTP1.Control.Stop_Cmd
  OTE  CGTP1.Control.Manual_Start  (seal-in)

Rung 7: AUTO START — CONTINUOUS RUN
  XIC  CGTP1.Status.Auto
  XIC  CGTP1.Status.Permissives_OK
  OTE  CGTP1.Control.Start_Req

Rung 8: MOTOR START OUTPUT
  [parallel]
    [series]
      XIC  CGTP1.Control.Start_Req
      XIC  CGTP1.Status.Permissives_OK
    XIC  CGTP1.Control.Manual_Start
  OTE  CGTP1_MS

Rung 9: STOP COMMAND
  [parallel]
    [series]
      XIC  CGTP1.Status.Manual
      XIC  System.Control.HMI_Stop_PB
    XIC  CGTP1.Control.HMI_Stop_PB
    XIC  CGTP1.Status.Faulted
  OTE  CGTP1.Control.Stop_Cmd
  ────
  XIO  CGTP1.Control.Reset
  OTE  CGTP1.Control.Stop_Cmd  (seal-in)

Rung 10: RUNNING STATUS
  XIC  CGTP1_FB
  OTE  CGTP1.Status.Running
```

---

## Ladder Logic — CT1 Fan (Cooling Tower Fan)

```
Routine: CT1_Fan — Cooling Tower Fan Speed Control
Total rungs: 7

Rung 0: NOP

Rung 1: CT FAN ENABLE
  XIC  CH1.Status.Running
  XIC  Safety.CT1_Fan_OK
  OTE  CT1_Fan.Control.Enable

Rung 2: CW TEMP PID
  XIC  CT1_Fan.Control.Enable
  P_PIDE  CT1_CW_PID
    Inp_PV:   CW_TT.Val
    Val_SP:   System.SP.CW_Temp
    Out_CV:   CT1_CW_PID.Out_CV

Rung 3: PID TO FAN SPEED REF
  XIC  CT1_Fan.Control.Enable
  Source: CT1_CW_PID.Out_CV
  Dest:   CT1_FAN_VFD.PSet_SpeedRef
  MOV

Rung 4: FAN VFD START
  XIC  CT1_Fan.Control.Enable
  OTE  CT1_FAN_VFD.PCmd_StartFwd

Rung 5: FAN VFD STOP
  XIO  CT1_Fan.Control.Enable
  OTE  CT1_FAN_VFD.PCmd_Stop

Rung 6: FAN RUNNING STATUS
  XIC  CT1_FAN_VFD.Sts_RunningFwd
  OTE  CT1_Fan.Status.Running
```

---

## Ladder Logic — CW Bypass

```
Routine: CW_Bypass — Condenser Water Bypass Valve Logic
Total rungs: 5

Rung 0: NOP

Rung 1: BYPASS CONDITION — FAN OFF AND CW TOO COLD
  XIO  CT1_Fan.Status.Running
  Source A: CW_TT.Val
  Source B: System.SP.CW_Temp_Min
  LES
  OTE  CW_Bypass.Bypass_Active

Rung 2: CLOSE TOWER INLET VALVE
  XIC  CW_Bypass.Bypass_Active
  OTE  CG_MCV2.PCmd_Close

Rung 3: OPEN BYPASS VALVE
  XIC  CW_Bypass.Bypass_Active
  OTE  CG_MCV3.PCmd_Open

Rung 4: NORMAL OPERATION — CLOSE BYPASS
  XIO  CW_Bypass.Bypass_Active
  OTE  CG_MCV3.PCmd_Close
```

---

## Ladder Logic — Tertiary CV-1 PID

```
Routine: Tertiary_CV1 — Tertiary Loop Temperature Control
Total rungs: 4

Rung 0: NOP

Rung 1: TERTIARY TEMP PID
  XIC  CGTP1.Status.Running
  P_PIDE  Tertiary_TT2_PID
    Inp_PV:   CGTP1_TT2.Val
    Val_SP:   34.0
    Out_CV:   Tertiary_TT2_PID.Out_CV

Rung 2: PID TO CV-1 OUTPUT
  XIC  CGTP1.Status.Running
  Source: Tertiary_TT2_PID.Out_CV
  Dest:   CV1_BELMIO_AO.PSet_CV
  MOV

Rung 3: CV-1 CLOSE WHEN PUMP OFF
  XIO  CGTP1.Status.Running
  Source: 0
  Dest:   CV1_BELMIO_AO.PSet_CV
  MOV
```

---

## Ladder Logic — Alarms

```
Routine: Alarms — System Alarm Aggregation
Total rungs: 8

Rung 0: NOP

Rung 1: CGPP1 ALARM
  XIC  CGPP1_MS
  XIO  CGPP1_FB
  XIO  CGPP1.Control.Reset
  TON  CGPP1.Alarm_TMR  Preset:5000  Accum:0
  ────
  XIC  CGPP1.Alarm_TMR.DN
  OTL  CGPP1.Status.Alarm

Rung 2: CGSP1 ALARM
  XIC  CGSP1_MS
  XIO  CGSP1_FB
  XIO  CGSP1.Control.Reset
  TON  CGSP1.Alarm_TMR  Preset:5000  Accum:0
  ────
  XIC  CGSP1.Alarm_TMR.DN
  OTL  CGSP1.Status.Alarm

Rung 3: CGSP2 ALARM
  XIC  CGSP2_MS
  XIO  CGSP2_FB
  XIO  CGSP2.Control.Reset
  TON  CGSP2.Alarm_TMR  Preset:5000  Accum:0
  ────
  XIC  CGSP2.Alarm_TMR.DN
  OTL  CGSP2.Status.Alarm

Rung 4: CGTP1 ALARM
  XIC  CGTP1_MS
  XIO  CGTP1_FB
  XIO  CGTP1.Control.Reset
  TON  CGTP1.Alarm_TMR  Preset:5000  Accum:0
  ────
  XIC  CGTP1.Alarm_TMR.DN
  OTL  CGTP1.Status.Alarm

Rung 5: CWP1 ALARM
  XIC  CWP1_MS
  XIO  CWP1_FB
  XIO  CWP1.Control.Reset
  TON  CWP1.Alarm_TMR  Preset:5000  Accum:0
  ────
  XIC  CWP1.Alarm_TMR.DN
  OTL  CWP1.Status.Alarm

Rung 6: CT1 FAN ALARM
  XIC  CT1_FAN_VFD.PCmd_StartFwd
  XIO  CT1_FAN_VFD.Sts_RunningFwd
  XIO  System.Control.Reset
  TON  CT1_Fan.Alarm_TMR  Preset:10000  Accum:0
  ────
  XIC  CT1_Fan.Alarm_TMR.DN
  OTL  CT1_Fan.Status.Alarm

Rung 7: ALARM RESET — ALL DEVICES
  XIC  System.Control.Reset
  OTU  CGPP1.Status.Alarm
  OTU  CGSP1.Status.Alarm
  OTU  CGSP2.Status.Alarm
  OTU  CGTP1.Status.Alarm
  OTU  CWP1.Status.Alarm
  OTU  CT1_Fan.Status.Alarm
  OTU  CGSP_LL.Status.Alarm
```
