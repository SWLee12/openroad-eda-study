# OpenROAD 실습 기록 (5주차)

## 목표
OpenROAD 소스코드에서 UROP 경험(Euler trail, A*, LEF/DEF)이 실제로 어느 파일에 구현되어 있는지 직접 확인

---

## 1. grt 모듈 분석 (Global Routing = Euler trail 연결)

### 경로
`OpenROAD/src/grt/src/`

### 핵심 파일 목록

| 파일 | 역할 |
|---|---|
| `GlobalRouter.cpp` | 글로벌 라우팅 메인 로직 |
| `fastroute/src/graph2d.cpp` | 2D 라우팅 그래프 엣지 관리 ← **Euler trail 직접 연결** |
| `fastroute/src/maze.cpp` | 미로 탐색 기반 라우팅 (A*와 유사) |
| `fastroute/src/FastRoute.cpp` | 글로벌 라우팅 전체 관리 |
| `fastroute/src/RipUp.cpp` | 배선 충돌 시 재라우팅 |
| `Grid.cpp` | 라우팅 그리드 구조 정의 ← **LEF/DEF 연결** |
| `Net.cpp` | 배선(Net) 정보 관리 |

---

## 2. graph2d.cpp 소스코드 분석

### Euler trail 연결 포인트 3가지

**① 그리드 엣지 구조**
```cpp
h_edges_.resize(boost::extents[x_grid - 1][y_grid]);
v_edges_.resize(boost::extents[x_grid][y_grid - 1]);
```
- 가로/세로 엣지를 2D 격자로 구성
- UROP에서 Euler trail로 순회했던 그래프의 엣지가 바로 이 구조

**② 엣지 사용량 추적**
```cpp
h_edges_[x][y].usage += used;
v_edges_[x][y].usage += used;
```
- 배선이 지나갈 때마다 usage 증가
- Euler trail에서 "이 엣지를 이미 지나갔는지" 체크하는 것과 동일한 개념

**③ 혼잡도 관리**
```cpp
const int overflow = edge.usage - edge.cap;
if (overflow > 0) {
    edges[x][y].congCNT++;
    edges[x][y].last_usage += overflow;
}
```
- 용량(cap)보다 사용량(usage)이 많으면 혼잡(overflow) 상태
- 경로 탐색 시 꽉 찬 엣지를 피하는 로직 → Euler trail의 엣지 회피와 동일한 개념

---

## 3. dpl 모듈 분석 (Detailed Placement = A* 연결)

### 경로
`OpenROAD/src/dpl/src/optimization/`

### 핵심 파일 목록 + 실습 로그 매핑

| 파일 | 실습 로그 결과 | UROP 연결 |
|---|---|---|
| `detailed_global.cxx` | global swaps: **3.66% 개선** | A* 전체 공간 탐색 |
| `detailed_mis.cxx` | independent set matching: **0.30% 개선** | 그래프 매칭 알고리즘 |
| `detailed_random.cxx` | random improver: **3.43% 개선** | 무작위 탐색 최적화 |
| `detailed_reorder.cxx` | reordering: **0.93% 개선** | 셀 순서 재배치 |
| `detailed_orient.cxx` | cell flipping: **2.55% 개선** | 셀 방향 최적화 |

### A* 연결 포인트
`detailed_global.cxx`가 A*랑 가장 직접적으로 연결됨. A*가 전체 공간에서 최적 경로를 찾는 것처럼, global swaps는 전체 배치 공간에서 더 나은 위치를 탐색함. 실습에서 이 단계가 HPWL을 3.66% 개선하는 것을 직접 확인함.

---

## 4. UROP 경험 → OpenROAD 소스코드 최종 매핑

| UROP 경험 | OpenROAD 소스 파일 | 실습에서 확인한 수치 |
|---|---|---|
| Euler trail | `grt/src/fastroute/src/graph2d.cpp` | 라우팅 그리드 엣지 구조 확인 |
| A* 알고리즘 | `dpl/src/optimization/detailed_global.cxx` | HPWL 3.66% 개선 확인 |
| LEF/DEF 처리 | `grt/src/Grid.cpp` | 플로어플랜 면적 629 um² 확인 |

---

## 5. 면접 답변 초안

**"본인 연구 경험이 EDA 플로우 어디에 해당하나요?"**

"UROP에서 구현한 Euler trail 알고리즘은 OpenROAD의 `grt/src/fastroute/src/graph2d.cpp`에 해당합니다. 이 파일에서 라우팅 그리드의 모든 엣지를 순회하며 배선 경로를 탐색하는 방식이 Euler trail과 동일한 구조임을 소스코드에서 직접 확인했습니다. 또한 A* 알고리즘은 `dpl/src/optimization/detailed_global.cxx`의 global swaps 단계에 해당하며, 실습을 통해 이 단계가 배선 길이(HPWL)를 3.66% 개선하는 것을 확인했습니다."

---

## 다음 할 것 (6~7주차)

- [ ] GitHub 레포 만들기
- [ ] 실습 기록 README 작성
- [ ] 자소서 최종 문장 완성
- [ ] 면접 예상 질문 답변 준비
  - "OpenROAD가 뭔가요?"
  - "본인 연구 경험이 EDA 플로우 어디에 해당하나요?"
  - "실제로 돌려본 결과가 어땠나요?"
  - "파라미터 실험에서 어떤 걸 배웠나요?"
