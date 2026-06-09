# OpenROAD 실습 기록 (2주차~3주차)

## 환경

| 항목 | 내용 |
|---|---|
| OS | Windows + WSL2 + Docker |
| 이미지 | `openroad/flow-ubuntu22.04-builder` |
| 디자인 | `gcd` (nangate45 공정, 45nm) |

---

## 실험 1: Floorplan 파라미터 변경

### 방법
`config.mk`에서 `CORE_UTILIZATION` 값을 변경 후 Floorplan 단계 재실행

```bash
sed -i 's/CORE_UTILIZATION ?= 55/CORE_UTILIZATION ?= 70/' /OpenROAD-flow-scripts/flow/designs/nangate45/gcd/config.mk
make DESIGN_CONFIG=./designs/nangate45/gcd/config.mk do-2_1_floorplan
```

### 결과

| CORE_UTILIZATION | 칩 면적 | Utilization |
|---|---|---|
| 55 (기본값) | 641 um² | 59% |
| 70 (변경값) | 629 um² | 72% |

### 해석
- Utilization을 높이면 같은 회로를 더 작은 면적에 구현 가능
- 단, 빽빽할수록 배선 복잡도 증가 및 타이밍 맞추기 어려워지는 트레이드오프 존재
- **LEF/DEF 처리 경험(UROP)이 이 단계에 해당**

---

## 실험 2: Placement 결과 분석

### 결과

| 항목 | 수치 |
|---|---|
| 총 배치 셀 수 | 544개 |
| 배치 성공률 | 100% |
| 배치 후 HPWL | 2971.6 u |
| 최적화 후 HPWL | 2640.4 u |
| HPWL 개선율 | 약 11% 감소 |
| 최종 칩 면적 | 680 um² |
| 최종 Utilization | 63% |

### 최적화 단계별 개선율

| 알고리즘 | HPWL 개선율 |
|---|---|
| Global swaps | 3.66% |
| Vertical swaps | 0.91% |
| Random improver | 3.43% |
| Cell flipping | 2.55% |

### 해석
- 배치 후 여러 최적화 알고리즘을 반복 적용해 배선 길이를 약 11% 단축
- **A* 알고리즘 경험(UROP)이 이 단계에 해당**

---

## CTS 에러 기록

### 에러 내용
```
Error: cts.tcl, 86 child killed: illegal instruction
```

### 원인
Windows + WSL2 + Docker 환경 호환성 문제로 CTS 단계 바이너리 실행 불가

### 조치
환경 제약으로 4단계(CTS) 이후 미완료. 1~3단계 결과로 실습 대체

---

## UROP 경험 연결 요약

| UROP 경험 | OpenROAD 단계 | 확인한 결과 |
|---|---|---|
| LEF/DEF 처리 | 2단계 Floorplan | 641→629 um² 면적 변화 확인 |
| A* 알고리즘 | 3단계 Placement | HPWL 11% 개선 확인 |
| Euler trail | 5단계 Routing | 환경 제약으로 미완료 |

---

## OpenROAD 플로우 6단계 요약

| 단계 | 이름 | 설명 | UROP 연결 |
|---|---|---|---|
| 1 | Synthesis | Verilog → 게이트 변환 | - |
| 2 | Floorplan | 칩 면적 및 블록 위치 설정 | LEF/DEF 처리 |
| 3 | Placement | 게이트 실제 좌표 배치 | A* 알고리즘 |
| 4 | CTS | 클럭 트리 합성 | - |
| 5 | Routing | 게이트 간 배선 연결 | Euler trail |
| 6 | Finishing | DRC 검증 및 GDS 생성 | - |

---

## 자소서 활용 문장 (초안)

오픈소스 EDA 툴인 OpenROAD 플로우를 직접 실행해 Floorplan 및 Placement 단계 결과를 분석했다. CORE_UTILIZATION 파라미터를 55에서 70으로 조정해 칩 면적이 641 um²에서 629 um²로 약 2% 감소하는 것을 확인했으며, Placement 단계에서 최적화 알고리즘 적용 후 배선 길이(HPWL)가 약 11% 단축되는 것을 검증했다. 학부 연구(UROP)에서 구현한 A* 알고리즘과 LEF/DEF 처리가 실제 EDA 플로우의 어느 단계에 적용되는지 직접 확인했다.

---

## 다음 주 할 것 (4~5주차)

- [ ] 추가 파라미터 실험 (`PLACE_DENSITY_LB_ADDON`, `TNS_END_PERCENT`)
- [ ] OpenROAD 소스코드 분석 (`src/grt/`, `src/dpl/`)
- [ ] UROP 알고리즘과 소스코드 비교
- [ ] GitHub에 실습 기록 올리기
