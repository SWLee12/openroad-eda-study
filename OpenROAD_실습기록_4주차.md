# OpenROAD 실습 기록 (4주차)

## 환경

| 항목 | 내용 |
|---|---|
| OS | Windows + WSL2 + Docker |
| 이미지 | `openroad/flow-ubuntu22.04-builder` |
| 디자인 | `gcd` (nangate45, 45nm) + `ibex` (nangate45, 45nm) |

---

## 실험 1: gcd 파라미터 추가 실험

### PLACE_DENSITY_LB_ADDON 변경

**설명:** 배치 밀도 여유값. 높을수록 셀 간 간격 넓게, 낮을수록 빽빽하게 배치

| 값 | 칩 면적 | Utilization |
|---|---|---|
| 0.20 (기본값) | 680 um² | 63% |
| 0.05 (변경값) | 680 um² | 63% |

**결과:** 변화 없음. gcd처럼 작은 회로는 이미 최적화되어 있어서 영향 미미

---

### TNS_END_PERCENT 변경

**설명:** 타이밍 위반 수정 목표값. 100이면 100% 다 수정, 50이면 50%만 수정하고 멈춤

| 값 | 칩 면적 | Utilization | HPWL |
|---|---|---|---|
| 100 (기본값) | 680 um² | 63% | 2637.3 u |
| 50 (변경값) | 680 um² | 63% | 2637.3 u |

**결과:** 면적 변화 없음. TNS_END_PERCENT는 타이밍 최적화 목표값이라 면적보다 타이밍에 영향을 줌

---

### gcd 파라미터 실험 종합 결론

gcd 회로(셀 544개)는 너무 작아서 파라미터 변화에 민감하게 반응하지 않음. 큰 회로에서 실험해야 차이가 뚜렷하게 나타남

---

## 실험 2: ibex 디자인으로 전환

### ibex vs gcd 기본 스펙 비교

| 항목 | gcd | ibex |
|---|---|---|
| 기능 | 최대공약수 계산 | RISC-V 32비트 CPU 코어 |
| 플로우 실행 시간 | 14초 | 1분 17초 |
| Resized instances | 91개 | 212개 |
| Peak memory | 138MB | 317MB |
| 기본 칩 면적 | 680 um² | 29,016 um² |
| 기본 Utilization | 63% | 51% |

ibex가 gcd보다 면적 기준 **약 43배** 큰 회로

---

## 실험 3: ibex CORE_UTILIZATION 변경

**방법:** `config.mk`에서 `CORE_UTILIZATION` 값 변경 후 Floorplan 단계 재실행

```bash
sed -i 's/CORE_UTILIZATION ?= 50/CORE_UTILIZATION ?= 70/' designs/nangate45/ibex/config.mk
make DESIGN_CONFIG=./designs/nangate45/ibex/config.mk DESIGN=ibex do-2_1_floorplan
```

### 결과

| CORE_UTILIZATION | 칩 면적 | Utilization |
|---|---|---|
| 50 (기본값) | 29,016 um² | 51% |
| 70 (변경값) | 28,826 um² | 71% |

면적 감소: **29,016 → 28,826 um² (약 0.7% 감소)**

---

## gcd vs ibex 파라미터 실험 비교

| 디자인 | Utilization 변경 | 면적 감소율 |
|---|---|---|
| gcd | 55 → 70 | 약 2% |
| ibex | 50 → 70 | 약 0.7% |

### 해석
- 큰 회로(ibex)에서 오히려 면적 감소율이 작게 나옴
- 이유: ibex처럼 복잡한 회로는 셀 간 배선 제약이 많아서 utilization을 높여도 면적을 크게 줄이기 어려움
- 단순히 utilization을 높인다고 면적이 비례해서 줄지 않는다는 것을 실험으로 확인

---

## 전체 실험 결과 종합

| 실험 | 디자인 | 파라미터 | 변경값 | 결과 |
|---|---|---|---|---|
| 1 | gcd | CORE_UTILIZATION | 55 → 70 | 면적 2% 감소 |
| 2 | gcd | PLACE_DENSITY_LB_ADDON | 0.20 → 0.05 | 변화 없음 |
| 3 | gcd | TNS_END_PERCENT | 100 → 50 | 변화 없음 |
| 4 | ibex | CORE_UTILIZATION | 50 → 70 | 면적 0.7% 감소 |

---

## 자소서 활용 문장 (업데이트)

"오픈소스 EDA 툴인 OpenROAD 플로우를 직접 실행해 gcd 및 ibex 두 가지 디자인에서 파라미터 변경 실험을 진행했다. CORE_UTILIZATION을 조정했을 때 소규모 회로(gcd, 544셀)에서는 약 2% 면적 감소가 나타난 반면, 대규모 회로(ibex, RISC-V CPU 코어)에서는 배선 제약으로 인해 약 0.7% 감소에 그쳤다. 이를 통해 회로 복잡도에 따라 파라미터 최적화 효과가 달라진다는 점을 실험적으로 확인했으며, 학부 연구(UROP)에서 구현한 A* 알고리즘과 LEF/DEF 처리가 실제 EDA 플로우의 어느 단계에 적용되는지 직접 검증했다."

---

## 다음 주 할 것 (5주차)

- [ ] OpenROAD 소스코드 분석
  - `src/grt/` — Global Routing 코드 (Euler trail 관련)
  - `src/dpl/` — Detailed Placement 코드 (A* 관련)
- [ ] UROP 알고리즘과 소스코드 직접 비교
- [ ] GitHub 레포 만들고 실습 기록 올리기 준비
