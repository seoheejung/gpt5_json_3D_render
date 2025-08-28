# 🖼️ JSON → 3D 이미지 생성기
- Three.js 기반 JSON 스펙을 입력하면 즉시 3D 씬을 렌더링해주는 도구입니다.
- 브라우저에서 index.html 파일을 더블클릭(file:// 방식)만으로 실행 가능합니다.

## 🚀 실행 방법
1. 저장소 클론 또는 ZIP 다운로드
2. index.html 파일을 브라우저에서 더블클릭 (별도 서버 실행 필요 없음)
3. 좌측 에디터에 JSON을 작성 → 렌더 버튼 클릭  
또는 Ctrl/Cmd + Enter로 즉시 반영

## 🧩 주요 기능
- JSON 입력 → 3D 씬 렌더링
- 자동 교정(AutoFix)
  - 잘못된 키(shape, cube, colour, fovDegrees 등) 자동 변환
  - 단위(cm, mm, rad, deg) 자동 변환
- 툴팁
  - JSON 스펙 설명에 속성별 i 아이콘 → 마우스 오버 시 설명 표시
- 샘플 삽입 버튼
  - 버튼 클릭으로 기본 예시 JSON 자동 입력
- 파일 불러오기 (.json)
- PNG/GLB 저장
- 배열 스키마 변환기
  - [ { shape, size, position... } ] 형식도 변환 가능
- 반응형 UI (모바일/태블릿 대응)

---

## 📜 JSON 스펙 요약
### 루트 필드
| 키            | 설명                                                  |
| ------------ | --------------------------------------------------- |
| `background` | 배경색 (#hex, rgb, hsl 등)                              |
| `axes`       | XYZ 축 표시 여부 (기본 `true`)                             |
| `grid`       | 바닥 격자 표시 여부 (기본 `true`)                             |
| `camera`     | `{ pos:[x,y,z], target:[x,y,z], fov }`              |
| `lights`     | 조명 배열. `ambient`, `directional`, `point`, `spot` 지원 |
| `objects`    | 장면 오브젝트 배열                                          |

### 오브젝트 공통
 - **type**: "box" | "sphere" | "cylinder" | "cone" | "torus" | "plane"
   - AutoFix에서 cube → box, shape → type 자동 교정 포함
 - **position, rotationDeg, scale** → 전부 setPos, setRotDeg, setScale에서 처리
 - **material**: { color, metalness, roughness, transparent, opacity, wireframe } → makeMaterial()에서 지원
 - **children → buildObject()** 안에서 재귀적으로 생성
 - **animation.spin → animate()** 루프에서 axis/speed 적용

### 형상별 파라미터
 - **box/cube**: ``BoxGeometry(w,h,d)``
 - **sphere**: ``SphereGeometry(radius, widthSegments, heightSegments)``
 - **cylinder**: ``CylinderGeometry(radiusTop, radiusBottom, height, radialSegments)``
 - **cone**: ``ConeGeometry(radius, height, radialSegments)``
 - **torus**: ``TorusGeometry(radius, tube, tubularSegments, radialSegments)``
 - **plane**: ``PlaneGeometry(width, height) + autoRotateToFloor일 때 rotation.x = -90°``
### 🎮 최소 예시
```json
{
  "background": "#0b0f16",
  "camera": { "pos": [6,4,8], "target": [0,0,0], "fov": 55 },
  "lights": [
    { "type": "ambient", "intensity": 0.6, "color": "#fff" },
    { "type": "directional", "intensity": 0.9, "color": "#fff", "pos": [5,10,7] }
  ],
  "objects": [
    { "type": "box", "size": [1,1,1], "position": [-1.5,0,0], "material": { "color": "#f00" } },
    { "type": "sphere", "radius": 0.75, "position": [1.5,0,0], "material": { "color": "#00f" } }
  ]
}
```
---

## 🔧 변경 사항 요약 (최근 개선점)
1. 샘플(랜덤) / 샘플(태양계)
    - 샘플(랜덤): 예시 풀에서 랜덤 JSON 주입
    - 샘플(태양계): 수·금·지·화·목·토·천·해·명 전부 포함 + 공전 애니메이션 + 토성 고리 + 궤도 링
2. 라이트 누적 밝아짐 버그 해결
    - 샘플 버튼을 반복 클릭해도 조명이 누적되지 않도록, 스펙에서 추가한 라이트를 추적/제거 후 재생성.
3. 배열 스키마 변환 UX 강화
    - 모달에 기본 예시 자동 주입
    - BOM 제거, 최상위 [] 검사, 친절한 오류 가이드로 Unexpected end of JSON input 방지
4. AutoFix(관대한 입력 교정) 강화
    - shape → type, cube → box, colour → color, fovDegrees → fov 등 자동 변환
    - mm/cm/m, deg/rad 단위 자동 처리
    - 스칼라 scale → [x,y,z] 확장, {x,y,z} / [x,y,z] 포지션 모두 지원
5. 키보드 지원 & 내보내기 유지
    - Ctrl/Cmd + Enter 즉시 렌더, PNG/GLB 저장 지원

## 💡 샘플 버튼 동작
1. 샘플(랜덤): 다양한 데모 장면 중 하나를 랜덤으로 에디터에 넣고 즉시 렌더
2. 샘플(태양계): 9개(왜소행성 포함) 행성+달(간단) 장면 로드
    - 각 행성은 orbit_* 그룹의 animation.spin으로 공전
    - 바닥에 눕힌 토러스 링으로 궤도 시각화
    - 토성 고리는 행성 로컬 좌표의 토러스 child로 구성

## 🔄 배열 스키마 변환 사용법
1. 좌측 툴바 [배열→스펙 변환] 클릭
2. 모달에 배열 JSON 붙여넣기 → 변환 실행
3. 변환된 결과가 에디터에 입력되고 즉시 렌더
### 입력 예시
```json
[
  { "shape": "cube", "size": { "width": 1, "height": 1, "depth": 1 }, "color": "#f00",
    "position": { "x": -1.5, "y": 0, "z": 0 } },
  { "shape": "sphere", "size": { "radius": 0.75 }, "color": "#00f",
    "position": { "x": 1.5, "y": 0, "z": 0 } }
]
```

## 🩺 트러블슈팅
1. 변환/파싱 실패 메시지
    - 모달/에디터에서 JSON 오류 시 원인을 자세히 안내(닫는 괄호/콤마/따옴표/BOM 등)
    - 최상위가 반드시 [](배열)인지 확인 (배열 변환기용)
2. 장면이 점점 밝아져요
    - 해결됨: 스펙 라이트를 렌더 전에 모두 제거 후 다시 추가합니다.
3. 회전이 이상해요
    - rotationDeg는 도(deg) 기준. rad로 넣어도 자동 변환됩니다.
4. 스케일 하나만 줬는데 왜 늘어나죠?
    - 스칼라 scale: 2 → [2,2,2]로 자동 확장합니다.

## ⌨️ 단축키
1. Ctrl/Cmd + Enter: 즉시 렌더
2. ESC: 변환 모달 닫기

## 🧠 팁
- 바닥 평면은 "type": "plane", "autoRotateToFloor": true로 쉽게 만들 수 있어요.
- 공전은 그룹에 animation.spin을 주고, 행성을 그 그룹의 children으로 두면 됩니다.
- 라이트 프리셋은 lights 배열만 바꿔도 분위기가 크게 달라집니다(ambient/directional/point/spot).

---
### 🗂️ 레포 구조
```
json_3D_render/
├── README.md  # 설명 문서
├── index.html # 리다리렉트용
└── src/  # (옵션) 실제 코드 넣는 공간
    └── index.html   # 단일 HTML 파일 (Three.js + OrbitControls + GLTFExporter)
```
### 📄 서버 없이도 바로 URL로 실행 (GitHub Pages 이용)
- GitHub에 새 리포지토리를 하나 만들고 left_turn_maze.html 파일을 업로드
- 리포지토리의 Settings → Pages 메뉴에서 Branch: main + / (root) 선택 후 저장
- 몇 분 뒤에 https://seoheejung.github.io/gpt5_json_3D_render/src/index.html 같은 URL로 접속 가능

<br>

### 🕹️ [사이트 실행](https://seoheejung.github.io/gpt5_json_3D_render)