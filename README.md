# OpenROAD EDA Flow Study

## Overview
Personal study log for OpenROAD RTL-to-GDS flow.  
Connecting undergraduate UROP research experience (Euler trail, A*, LEF/DEF) to actual EDA tool implementation.

## Environment
- OS: Windows 11 + WSL2 + Docker
- Image: `openroad/flow-ubuntu22.04-builder`
- Designs: `gcd` (nangate45), `ibex` (nangate45)

## UROP Research → OpenROAD Flow Mapping

| UROP Experience | OpenROAD Stage | Source File |
|---|---|---|
| Euler trail | 5. Routing | `grt/src/fastroute/src/graph2d.cpp` |
| A* Algorithm | 3. Placement | `dpl/src/optimization/detailed_global.cxx` |
| LEF/DEF Processing | 2. Floorplan | `grt/src/Grid.cpp` |

## Experiment Results

### gcd Design (544 cells)
| Parameter | Value | Die Area | Utilization |
|---|---|---|---|
| CORE_UTILIZATION | 55 (default) | 641 um² | 59% |
| CORE_UTILIZATION | 70 | 629 um² | 72% |
| After Placement | - | 680 um² | 63% |

- Placement HPWL optimization: 2971.6 → 2640.4 u (**-11.1%**)

### ibex Design (RISC-V CPU Core)
| Parameter | Value | Die Area | Utilization |
|---|---|---|---|
| CORE_UTILIZATION | 50 (default) | 29,016 um² | 51% |
| CORE_UTILIZATION | 70 | 28,826 um² | 71% |

### Key Finding
Increasing CORE_UTILIZATION reduced die area by **2%** in gcd and **0.7%** in ibex.  
Complex circuits (ibex) show smaller area reduction due to routing constraints.

## Flow Stages
| Stage | Name | Description |
|---|---|---|
| 1 | Synthesis | Verilog → Logic gates |
| 2 | Floorplan | Define chip area and block placement |
| 3 | Placement | Place cells at exact coordinates |
| 4 | CTS | Clock tree synthesis |
| 5 | Routing | Connect cells with metal wires |
| 6 | Finishing | DRC check and GDS generation |

## Note
CTS stage (Stage 4) failed due to Docker + WSL2 environment compatibility issue (`illegal instruction`).  
Stages 1-3 completed successfully with result verification.
