# PRD — TreeMap 360 (세이브트리 수목 관리 데모)

## 1. Product Overview

| 항목 | 내용 |
|------|------|
| **서비스명** | TreeMap 360 — 세이브트리 수목 관리 데모 |
| **한 줄 설명** | 아파트 단지 수목을 3D 포인트클라우드 + 트리맵으로 관리하는 데모. SaveTree × Greehill 협업. |
| **대상 사용자** | 조경 관리자, 아파트 관리사무소 |

## 2. Tech Stack

| 구분 | 기술 |
|------|------|
| 프레임워크 | 단일 HTML (Vanilla JS) |
| 그래픽 | SVG 지도 + WebGL (포인트클라우드) |
| 스타일링 | 인라인 CSS / `<style>` 태그 |
| 빌드 도구 | 없음 (순수 HTML/JS/CSS) |
| 배포 | GitHub Pages (main 브랜치) |

## 3. Architecture

```
단일 HTML 파일
├── 5개 화면 (탭/네비게이션 전환)
│   ├── 대시보드 — 단지 전체 수목 현황
│   ├── 트리맵 — 3컬럼 레이아웃 (핵심 화면)
│   ├── 수목 상세 — 개별 수목 정보
│   ├── 포인트클라우드 — WebGL 3D 뷰
│   └── 수목 관리 — 관리 이력/작업
├── SVG 지도 렌더러
├── WebGL 포인트클라우드 렌더러
└── JavaScript (DOM 조작 기반)
```

## 4. Pages & Routes

단일 HTML 내 화면 전환:

| 화면 | 기능 |
|------|------|
| 대시보드 | 단지 전체 수목 현황 요약 (등급별 건수, 센서 상태 등) |
| **트리맵** | 핵심 화면 — 3컬럼 레이아웃으로 수목 탐색/상세/지도 통합 뷰 |
| 수목 상세 | 개별 수목의 5탭 상세 정보 |
| 포인트클라우드 | WebGL 기반 3D 포인트클라우드 뷰어 |
| 수목 관리 | 관리 이력, 작업 일지 |

### 트리맵 (핵심 화면) — 3컬럼 레이아웃

| 컬럼 | 폭 | 내용 |
|------|-----|------|
| 좌측 | 280px | 수목 카드 1열 리스트 (수종/등급/수고/DBH) |
| 중앙 | 420px | 선택 수목 상세 (사진 4종 + 5탭 슬라이드) |
| 우측 | flex | SVG 지도 + 마커 + 상단 필터바 |

## 5. Data Models

```typescript
// 수목 데이터 (14개 수목)
interface TreeData {
  id: string;              // 수목 ID
  sp: string;              // 수종명 (예: '왕벚나무')
  grade: 'A' | 'B' | 'C' | 'D';  // 건강등급
  height: number;          // 수고 (m)
  dbh: number;             // DBH - 흉고직경 (cm)
  crown: number;           // 수관폭 (m)
  age: number;             // 수령 (년)
  location: string;        // 위치 설명
  sensor: SensorData;      // 센서 데이터
  sti: number;             // STI (Structural Tree Index)
  x: number;               // 지도 X 좌표
  y: number;               // 지도 Y 좌표
}

// 센서 데이터 (4종)
interface SensorData {
  soilMoisture: number;    // 토양수분 (%)
  ph: number;              // pH
  ei: number;              // EI (전기전도도 지수)
  ec: number;              // EC (전기전도도)
}

// 5탭 카테고리 데이터
interface TreeDetailTabs {
  // 탭 1: 기초
  basic: {
    treeNo: string;        // 수목번호
    species: string;       // 수종 (학명 포함)
    location: string;      // 위치
    age: number;           // 수령
    sti: number;           // STI
  };
  // 탭 2: 규격
  specification: {
    height: number;        // 수고
    dbh: number;           // DBH
    crownWidth: number;    // 수관폭
    age: number;           // 수령
    crownShape: string;    // 수관형태
    rootCollarDia: number; // 근원직경
    clearTrunk: number;    // 지하고
  };
  // 탭 3: 위험성
  risk: {
    overallRisk: 'A' | 'B' | 'C' | 'D';  // 종합위험등급
    structuralDefect: string;    // 구조적결함
    branchFailure: string;       // 가지파손
    lean: number;                // 경사 (도)
    vitality: string;            // 활력도
    sensors: SensorData;         // 센서 4종
  };
  // 탭 4: 생태계서비스
  ecosystem: {
    carbonStorage: number;       // 탄소저장량 (kg)
    carbonAbsorption: number;    // 탄소흡수량 (kg/yr)
    stormwaterReduction: number; // 우수유출억제 (m³/yr)
    airPollutionReduction: number; // 대기오염저감 (kg/yr)
    economicValue: number;       // 경제가치 - i-Tree (원)
    biodiversity: string;        // 생물다양성 등급
  };
  // 탭 5: 간섭정보
  interference: {
    underground: string;         // 지하매설물
    pavementGap: number;         // 포장재이격 (m)
    buildingGap: number;         // 건물이격 (m)
    wireInterference: boolean;   // 전선간섭
    sidewalkIntrusion: boolean;  // 보행로침범
    maintenanceHistory: MaintenanceRecord[];
  };
}

interface MaintenanceRecord {
  date: string;
  type: string;                  // 작업 유형
  description: string;
}
```

## 6. Key Features

### 6.1 트리맵 (3컬럼 레이아웃)
- **좌측 280px**: 수목 카드 리스트 — 수종, 건강등급 배지, 수고, DBH 표시. 클릭 시 중앙/우측 연동
- **중앙 420px**: 선택 수목 상세 — 사진 4종 슬라이더 + 5탭 슬라이드 (기초/규격/위험성/생태계서비스/간섭정보)
- **우측 flex**: SVG 단지 지도 — 수목 마커(등급별 색상), 상단 필터바

### 6.2 5탭 상세 정보
1. **기초**: 수목번호, 수종(학명), 위치, 수령, STI
2. **규격**: 수고, DBH, 수관폭, 수령, 수관형태, 근원직경, 지하고
3. **위험성**: 종합위험등급, 구조적결함, 가지파손, 경사, 활력도 + 센서 4종(토양수분/pH/EI/EC)
4. **생태계서비스**: 탄소저장량, 흡수량, 우수유출억제, 대기오염저감, 경제가치(i-Tree), 생물다양성
5. **간섭정보**: 지하매설물, 포장재이격, 건물이격, 전선간섭, 보행로침범 + 관리이력

### 6.3 필터바
- 건강등급 필터: A / B / C / D
- 수종 필터: 왕벚나무 / 느티나무 / 은행나무 / 소나무
- 센서 상태 필터

### 6.4 SVG 단지 지도
- 단지 레이아웃 SVG 렌더링
- 수목 위치 마커 (등급별 색상)
- 마커 클릭 → 좌측 리스트/중앙 상세 연동

### 6.5 포인트클라우드 (WebGL)
- 3D 포인트클라우드 렌더링
- 마우스/터치 회전/줌 지원

### 6.6 수목 데이터
- 14개 수목 하드코딩 (id/sp/grade/height/dbh/crown/age/location/sensor/sti/x/y)

## 7. Design System

| 속성 | 값 |
|------|-----|
| 컬러 팔레트 | **딥그린** — green-900 ~ green-50 |
| 배경색 | green-50 (밝은 녹색) |
| 강조색 | green-700 ~ green-900 |
| 등급 배지 | A: 초록, B: 노랑, C: 주황, D: 빨강 |
| 폰트 | Pretendard |
| 레이아웃 | 3컬럼 (280px + 420px + flex) |
| 카드 | 라운드 코너, 그림자, 호버 효과 |
| 지도 마커 | 등급별 색상 원형 마커 |

## 8. External Integrations

| 서비스 | 연동 방식 |
|--------|-----------|
| i-Tree | 경제가치 산출 기준 참조 (하드코딩) |
| Greehill | 포인트클라우드 데이터 참조 (목업) |
| 센서 시스템 | 토양수분/pH/EI/EC 데이터 (하드코딩) |

## 9. Deployment

| 항목 | 값 |
|------|-----|
| 호스팅 | GitHub Pages |
| 브랜치 | `main` |
| URL | `https://{owner}.github.io/savetree-demo/` |

## 10. Known Limitations

- **14개 수목 데이터 하드코딩** — 실제 DB/API 없음
- **포인트클라우드는 데모용** — 실제 LiDAR 스캔 데이터 아님
- **센서 데이터는 정적** — 실시간 IoT 연동 없음
- **i-Tree 경제가치는 예시값** — 실제 i-Tree 엔진 미연동
- **SVG 지도는 하드코딩** — 동적 지도 타일 아님
- **생태계서비스 값은 목업** — 실제 측정/계산 아님
- **관리이력은 샘플 데이터** — 실제 작업 기록 연동 없음
