# ManimGL 전수조사 분석 및 활용·수익화 가이드 (한국어)

> 이 문서는 `bmshin94/manim` 저장소를 전수조사한 결과와, 이를 바탕으로 한
> 설치·활용·AI 에이전트 연계·수익화 방안을 정리한 기록입니다.
>
> - **이 저장소**: https://github.com/bmshin94/manim
> - **원본 저장소 (upstream)**: https://github.com/3b1b/manim
> - **커뮤니티 버전 (다른 프로젝트)**: https://github.com/ManimCommunity/manim
> - **3b1b 실제 영상 소스코드**: https://github.com/3b1b/videos
> - **공식 문서**: https://3b1b.github.io/manim/
> - **PyPI 패키지**: https://pypi.org/project/manimgl/
> - **중국어 문서(참고용)**: https://docs.manim.org.cn/
> - 작성일: 2026-10-03

---

## 목차

1. [저장소 정체와 전수조사 결과](#1-저장소-정체와-전수조사-결과)
2. [핵심 개념 — 쉽게 다시 설명](#2-핵심-개념--쉽게-다시-설명)
3. [질문별 답변 (설치·정체·토큰·AI·웹·유튜브)](#3-질문별-답변)
4. [수익화 아이디어 상세](#4-수익화-아이디어-상세)
5. [종합 추천 전략](#5-종합-추천-전략)

---

## 1. 저장소 정체와 전수조사 결과

### 1.1 기본 정보

| 항목 | 내용 |
|---|---|
| 저장소 | `bmshin94/manim` (https://github.com/bmshin94/manim) — `3b1b/manim` 포크 |
| 패키지명 | **`manimgl`** (v1.7.2) — `manim`이 아님 |
| 제작자 | Grant Sanderson (유튜브 3Blue1Brown 운영자) |
| 라이선스 | **MIT** (상업적 이용·수정·재배포·유료 판매 전부 자유) |
| 규모 | Python 116개 파일, 약 **28,400줄** |
| 종류 | **독립 Python 라이브러리 + CLI 프로그램** (플러그인·스킬·MCP 아님) |
| Python | 3.10 이상 |
| 의존성 | 30개 (numpy, scipy, sympy, wgpu, rendercanvas, manimpango, av, trimesh 등) |

한 줄 정의: **코드로 수학·과학 애니메이션 영상을 만드는 렌더링 엔진.**
영상 편집기가 아니라, Python 코드를 작성하면 `.mp4`로 뽑아주는 프로그램입니다.

### 1.2 폴더 구조 전수조사

```
manim/
├── manimlib/              # 엔진 본체
│   ├── __main__.py        # CLI 진입점 (manimgl 명령어)
│   ├── config.py          # 34개 CLI 옵션 파싱
│   ├── default_config.yml # 기본 설정 (해상도/색상/단축키/FPS)
│   ├── constants.py       # UP, DOWN, LEFT, PI, BLUE 등 상수
│   ├── window.py          # 실시간 미리보기 창
│   │
│   ├── mobject/           # 【화면에 나오는 물체】 166개 클래스
│   │   ├── mobject.py            (2,313줄 — 모든 물체의 조상)
│   │   ├── geometry.py           Circle, Square, Line, Arrow, Polygon...
│   │   ├── coordinate_systems.py Axes, NumberPlane, ComplexPlane
│   │   ├── three_dimensions.py   Sphere, Cube, Torus
│   │   ├── vector_field.py       벡터장 (유체·전자기장)
│   │   ├── matrix.py / probability.py / fractals.py
│   │   ├── svg/   # tex_mobject.py(LaTeX), text_mobject.py(글자),
│   │   │          # brace.py(중괄호), drawings.py(시계·피아노·전구)
│   │   └── types/ # vectorized_mobject.py, surface.py(3D 곡면),
│   │              # video_mobject.py(영상 삽입), dot_cloud.py
│   │
│   ├── animation/         # 【움직이는 방식】 78개 클래스
│   │   ├── creation.py    ShowCreation, Write, DrawBorderThenFill
│   │   ├── transform.py   Transform, ReplacementTransform, ApplyMatrix
│   │   ├── fading.py      FadeIn, FadeOut, FadeTransform
│   │   ├── indication.py  Flash, Indicate, Wiggle
│   │   ├── movement.py / rotation.py / numbers.py
│   │   ├── composition.py AnimationGroup, LaggedStart
│   │   └── transform_matching_parts.py  TransformMatchingTex ★
│   │
│   ├── scene/             # 【감독】
│   │   ├── scene.py              (947줄) play(), wait(), add(), embed()
│   │   ├── interactive_scene.py  (651줄) 마우스로 선택·복사·이동
│   │   ├── scene_embed.py        (248줄) IPython 실시간 코딩
│   │   └── scene_file_writer.py  (399줄) FFmpeg로 mp4 출력
│   │
│   ├── camera/            # 카메라 (줌·3D 회전·오일러각)
│   ├── renderer/          # 【GPU 렌더링】 10개 파일, 약 1,900줄
│   ├── shaders/           # GPU 셰이더 11개 (.wgsl)
│   ├── event_handler/     # 마우스·키보드 이벤트
│   └── utils/             # 21개 유틸 (bezier, color, space_ops, tex_file_writing...)
│
├── example_scenes.py      # 예제 12개 (826줄) ★ 학습 출발점
├── docs/                  # Sphinx 문서 18페이지
├── tests/                 # 테스트 8개 (대부분 수동 시각 검증 스크립트)
├── requirements.txt       # 의존성 30개
└── .github/workflows/     # docs.yml(문서 배포), publish.yml(PyPI 배포)
```

### 1.3 조사 중 발견한 중요 포인트

**① 렌더러가 WebGPU로 교체된 최신판입니다.**
`requirements.txt`에 `wgpu`, `rendercanvas`가 있고 셰이더가 `.glsl`이 아니라 **`.wgsl`** 입니다.
과거 ManimGL은 OpenGL(moderngl)을 썼으나, 이 버전은 **WebGPU 기반으로 재작성**된 상태이며
`manimlib/renderer/` 전체가 신규 코드입니다.

**② 문서가 코드와 불일치합니다.**
`docs/source/getting_started/structure.rst`는 여전히 옛 구조(GLSL, `shader_wrapper.py`,
`once_useful_constructs/`, `three_d_scene.py`)를 설명합니다. 문서만 믿고 따라가면 혼란스럽습니다.
**실제 구조는 위 1.2절(코드 직접 조사 결과)을 기준으로 보세요.**

**③ 최근 커밋 이력**
- `49ff795` CLAUDE.md 추가 (PR #1 머지)
- `fafa083` VideoMobject / Sprite 추가 (#2521) — 애니메이션 안에 **동영상 삽입** 기능
- `9d57bcf` 병합된 mobject의 stale separator 레코드 버그 수정

**④ 외부 네트워크·인증 코드가 전혀 없습니다.**
`grep -rn "api_key|API_KEY|token|requests.get|urlopen" manimlib/` → **결과 0건**.
완전 오프라인, 완전 무료로 동작합니다.

### 1.4 ManimGL vs ManimCommunity — 반드시 구분

manim은 **두 개의 완전히 다른 프로젝트**로 갈라져 있습니다. README에도 경고문이 있습니다.

| | **ManimGL** (이 저장소) | **ManimCommunity** |
|---|---|---|
| pip 설치 | `pip install manimgl` | `pip install manim` |
| 명령어 | `manimgl` | `manim` |
| import | `from manimlib import *` | `from manim import *` |
| 생성 애니메이션 | `ShowCreation()` | `Create()` |
| 운영 | 3b1b 개인 | 커뮤니티 |
| 성격 | 3b1b 영상 제작용, 실험적, API 자주 변경 | 안정적, 테스트·문서 충실, 입문 친화 |
| 강점 | **실시간 미리보기 + 인터랙티브 편집** | 생태계·튜토리얼 풍부 |

**둘을 섞어 설치하면 반드시 깨집니다. 코드도 호환되지 않습니다.**

### 1.5 어떨 때 쓰는가

**적합한 경우**
- 개념의 "변환"을 보여줄 때 — 행렬이 평면을 찌그러뜨리는 모습, `z → z²` 복소함수 사상
- 수식이 수식으로 변형되는 과정 (`TransformMatchingTex`)
- 그래프·좌표계 기반 설명 — 리만 적분, 접선 기울기 변화
- 3D 곡면·벡터장 — 유체 흐름, 전자기장
- 알고리즘·물리 시뮬레이션 시각화

**부적합한 경우**
- 실사 편집, 로고 모션그래픽, 캐릭터 애니메이션, 일반 PPT 대체
  → 프리미어/애프터이펙트/Figma의 영역

### 1.6 나에게 주는 가치

1. **교육 콘텐츠 제작 무기** — MIT 라이선스로 상업적 사용·판매 전부 자유
2. **코드 기반이라 자동화·양산 가능** — "구구단 애니메이션 81개"를 for문으로 생성
3. **차별화된 비주얼** — 3b1b 특유의 배경(`#333333`)·파스텔 팔레트가 기본값으로 내장
4. **AI 에이전트의 "손"으로 쓸 수 있음** — 입력이 텍스트(Python), 출력이 파일(mp4)
   → **"텍스트 → 영상" 파이프라인** 구성 가능

---

## 2. 핵심 개념 — 쉽게 다시 설명

### 2.1 비유: "코딩으로 하는 영상 제작"

- **일반 영상 편집**: 마우스로 도형을 끌고, 타임라인에 키프레임 찍고, 눈으로 미세조정
- **manim**: `self.play(ShowCreation(Circle()))` 를 적고 엔터 → 영상 파일이 나옴

### 2.2 manim의 3대 요소

#### ① Mobject = "배우" (화면에 보이는 모든 것) — 166종

Mobject = **M**athematical **Object**

```python
Circle()                    # 원
Square()                    # 정사각형
Text("안녕하세요")            # 글자
Tex(r"\int_0^1 x^2 dx")     # LaTeX 수식
Axes()                      # 좌표축
Sphere()                    # 3D 구
NumberPlane()               # 모눈종이 평면
Arrow(LEFT, RIGHT)          # 화살표
```

공통 조작 메서드 (`mobject.py` 2,313줄이 이 공통 기능):

```python
circle.shift(UP)               # 위로 이동
circle.scale(2)                # 2배 확대
circle.rotate(PI/4)            # 45도 회전
circle.set_color(BLUE)         # 색 지정
circle.next_to(square, RIGHT)  # 사각형 오른쪽에 배치
circle.to_edge(UP)             # 화면 위쪽 끝으로
```

#### ② Animation = "연기 지시" — 78종

```python
ShowCreation(circle)    # 펜으로 그리듯 나타남
Write(text)             # 손으로 쓰이듯 나타남
FadeIn(circle)          # 서서히 나타남
FadeOut(circle)         # 서서히 사라짐
Transform(A, B)         # A가 B로 녹아들며 변신 ★manim의 꽃
Rotate(cube, PI)        # 회전
Indicate(text)          # 잠깐 커졌다 작아지며 강조
Flash(dot)              # 반짝
```

#### ③ Scene = "감독" (순서 지정)

```python
from manimlib import *

class MyVideo(Scene):          # Scene 상속 = 영상 한 편
    def construct(self):       # construct 안에 각본
        원 = Circle()
        네모 = Square()

        self.play(ShowCreation(네모))   # 1. 사각형 그리기
        self.wait()                     # 2. 1초 쉬기
        self.play(Transform(네모, 원))   # 3. 사각형 → 원 변신
        self.wait(2)                    # 4. 2초 쉬기
```

```sh
manimgl my_file.py MyVideo -w     # → videos/MyVideo.mp4
```

**요약: 배우(Mobject)를 만들고 → 연기 지시(Animation)를 내리고 → 감독(Scene)이 `self.play()`로 찍는다.**

### 2.3 왜 특별한가 — 다른 도구로 못 하는 것

#### (1) 수식이 수식으로 변신

```python
식1 = Tex("(a+b)^2")
식2 = Tex("a^2 + 2ab + b^2")
self.play(TransformMatchingTex(식1, 식2))
```

`a`는 `a²` 자리로, `b`는 `b²` 자리로 **글자 단위로 날아가 재배치**됩니다.
`transform_matching_parts.py`가 LaTeX 글자를 추적해 짝을 맞춥니다.
애프터이펙트로 하려면 글자마다 수동 키프레임이 필요합니다.

#### (2) 행렬이 평면을 찌그러뜨리는 모습 (`example_scenes.py` 첫 예제)

```python
grid = NumberPlane((-10, 10), (-5, 5))
matrix = [[1, 1], [0, 1]]
self.play(grid.animate.apply_matrix(matrix), run_time=3)
```

선형대수 수업의 "그 영상"이 **4줄**입니다.

#### (3) 실시간 인터랙티브 개발 — ManimGL만의 킬러 기능

```python
self.embed()   # 이 한 줄
```

영상이 그 지점에서 멈추고 터미널이 열립니다. 미리보기 창이 살아있는 상태에서
명령을 입력하면 즉시 반영됩니다.

```
>>> play(circle.animate.shift(2*RIGHT))    # 창에서 바로 움직임
>>> circle.set_color(RED)                  # 즉시 빨개짐
>>> undo()                                 # 되돌리기
>>> touch()                                # 창과 직접 상호작용
```

**렌더링 30분 기다렸다가 "색이 이상하네" 하고 또 30분 기다리는 지옥이 없습니다.**
`scene_embed.py`가 IPython을 끼워 구현했고, ManimCommunity 대신 ManimGL을 쓰는 가장 큰 이유입니다.

#### (4) 마우스 직접 편집 (`InteractiveScene`, 651줄)

| 키 | 기능 | 키 | 기능 |
|---|---|---|---|
| `s` | 선택 모드 | `c` | 색상 팔레트 |
| `g` | 끌어 이동 | `d` + 마우스 | 3D 카메라 회전 |
| `t` | 크기 조절 | `f` | 화면 이동(pan) |
| `h` / `v` / `z` | x/y/z축 고정 이동 | `r` | 카메라 리셋 |
| `i` | 좌표 정보 표시 | `k` | 커서 표시 |
| `Ctrl+C/V` | 복사/붙여넣기 | `Ctrl+Z` | 되돌리기 |

#### (5) 코드라서 양산이 됨 ★가장 중요

```python
for 단 in range(2, 10):
    for 수 in range(1, 10):
        식 = Tex(f"{단} \\times {수} = {단*수}")
        self.play(Write(식))
```

구구단 영상 81개 자동 생성. **수작업 편집과의 결정적 차이이자 수익화의 토대.**

### 2.4 설치가 까다로운 이유 — 외부 프로그램 의존

| 외부 프로그램 | 역할 | 필수 여부 |
|---|---|---|
| **FFmpeg** | 프레임들을 mp4로 묶기 | 필수 |
| **OpenGL / GPU 드라이버 (WebGPU)** | 실제 그림 그리기 | 필수 |
| **LaTeX** | 수식을 벡터 도형으로 변환 | 수식 쓸 때만 |
| **Pango** | 글꼴 렌더링 | Linux만 필수 |

설치 실패의 약 90%가 **LaTeX와 FFmpeg** 때문입니다. LaTeX 전체 설치는 6GB가 넘으므로
README의 경량 설치(`texlive-science texlive-fonts-extra texlive-latex-extra`)를 권장합니다.

### 2.5 3b1b의 "그 느낌"이 설정파일에 내장

`manimlib/default_config.yml` 발췌:

```yaml
camera:
  background_color: "#333333"     # 3b1b의 어두운 회색 배경
  resolution: (1920, 1080)
  fps: 30
colors:
  blue_c:   "#58C4DD"    # 그 하늘색
  yellow_c: "#FFFF00"
  teal_c:   "#5CD0B3"
  red_c:    "#FC6255"
  purple_c: "#9A72AC"
sizes:
  frame_height: 8.0      # 화면 세로 = manim 좌표 8단위
```

작업 폴더에 `custom_config.yml`을 두면 덮어쓸 수 있습니다.

### 2.6 좌표계 (초보자가 가장 헷갈리는 부분)

픽셀이 아니라 **추상 단위**를 사용합니다.

- 화면 세로 = **8단위**, 가로 ≈ 14.2단위
- 화면 중앙 = `(0,0,0)` = `ORIGIN`
- 방향 상수: `UP`, `DOWN`, `LEFT`, `RIGHT`, `IN`, `OUT`

```python
circle.shift(2 * UP + 3 * RIGHT)
circle.to_edge(UP)
```

→ **해상도를 바꿔도 레이아웃이 깨지지 않습니다.** 480p로 테스트하고 4K로 최종 렌더링해도 그대로입니다.

---

## 3. 질문별 답변

### Q1. 설치 및 사용법

#### Windows

```powershell
python --version              # 3.10 이상 확인
winget install ffmpeg         # FFmpeg
# LaTeX: MiKTeX 설치 (https://miktex.org/download)
pip install manimgl
manimgl --version
```

#### 이 저장소를 직접 쓰는 경우 (권장)

```sh
git clone https://github.com/bmshin94/manim.git
cd manim

python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # Mac/Linux

pip install -e .               # editable 설치 → 소스 수정 즉시 반영
```

#### Linux (Ubuntu/Debian)

```sh
sudo apt update
sudo apt install ffmpeg python3-pip libpango1.0-dev
sudo apt install texlive-science texlive-fonts-extra texlive-latex-extra

git clone https://github.com/bmshin94/manim.git
cd manim
python3 -m pip install -e .
```

#### macOS

```sh
brew install ffmpeg mactex
arch -arm64 brew install pkg-config cairo   # Apple Silicon만
git clone https://github.com/bmshin94/manim.git
cd manim
pip install -e .
```

#### 첫 실행

```sh
manimgl example_scenes.py OpeningManimExample
```

창이 뜨고 애니메이션이 재생되면 설치 성공입니다.

#### 자주 쓰는 CLI 옵션 (총 34개 중 핵심)

| 옵션 | 의미 | 쓰는 순간 |
|---|---|---|
| (없음) | 창에서 미리보기만 | 작업 중 |
| `-w` | mp4 저장 | 완성 시 |
| `-o` | 저장 + 자동 열기 | 완성 시 |
| `-s` | 마지막 프레임만 | 레이아웃 확인 ★시간절약 |
| `-so` | 마지막 프레임을 이미지로 저장 | 썸네일 |
| `-l` | 480p (빠름) | 테스트 ★필수 |
| `-m` | 720p | |
| `--hd` | 1080p | 최종 |
| `--uhd` | 4K | 최종 |
| `-n 3` / `-n 3,6` | 3번째부터 / 3~6번째만 | 뒷부분 수정 ★시간절약 |
| `-t` | 투명 배경(알파 채널) | 합성용 |
| `-i` | GIF 저장 | SNS |
| `-p` | 발표자 모드 (`wait()`에서 정지) | 실시간 강의 ★ |
| `-a` | 파일의 모든 Scene 렌더링 | 일괄 처리 ★자동화 |
| `-f` | 전체화면 | |
| `-e 42` | 42번째 줄에서 멈추고 터미널 | 디버깅 ★ |
| `--fps 60` | 프레임레이트 | |
| `-c "#000000"` | 배경색 | |
| `--autoreload` | 파일 수정 시 자동 리로드 | 작업 중 ★ |
| `--clear-cache` | LaTeX 캐시 삭제 | 수식이 안 바뀔 때 |
| `--subdivide` | 애니메이션별 파일 분리 | 편집용 |
| `--config_file` | 커스텀 설정 파일 지정 | |

#### 실전 작업 루틴 ★중요

```sh
manimgl my_scene.py MyScene -sl        # 1. 저화질 최종프레임 — 레이아웃 확인
manimgl my_scene.py MyScene -l         # 2. 저화질 전체 — 움직임 확인
manimgl my_scene.py MyScene -l -n 5    # 3. 뒷부분만 수정했으면 그 부분만
manimgl my_scene.py MyScene -o --hd    # 4. 최종 출력
```

**처음부터 `--hd -w`로 돌리면 렌더링 대기로 하루를 다 씁니다.**

#### `-p` 발표자 모드 활용
`-p`를 주면 `self.wait()`마다 멈추고 스페이스바로 넘어갑니다. **manim을 PPT 대신** 쓸 수 있습니다.

---

### Q2. 플러그인? 스킬? MCP?

**셋 다 아닙니다.** `pip`으로 설치하는 **독립 Python 패키지(라이브러리 + CLI)** 입니다.

| | 정의 | manim은? |
|---|---|---|
| 플러그인 | 호스트 프로그램에 끼우는 확장 | ❌ 호스트 없이 독립 실행 |
| 스킬 (Claude Skill) | Claude에게 작업 방식을 가르치는 지침 묶음 | ❌ 단, 감쌀 수 있음 |
| MCP 서버 | AI가 외부 도구를 호출하는 표준 프로토콜 서버 | ❌ 단, 만들 수 있음 |
| 라이브러리/CLI | `import` 또는 터미널 실행 | ✅ **정답** |

**근거 (전수조사)**
- `setup.cfg`의 `console_scripts`에 `manimgl`, `manim-render` 등록 → CLI 프로그램
- `manimlib/__init__.py`가 모든 클래스를 export → 라이브러리
- `.mcp.json`, `.claude/skills/`, `plugin.json` 등 **전무**

#### 다만 — 스킬/MCP로 감싸는 것은 가능하고 유용

manim은 **"텍스트(Python) → 파일(mp4)"** 라는 깔끔한 인터페이스라 AI 도구화에 최적입니다.

**A. Claude Skill로 감싸기 (쉬움, 반나절)**

```
.claude/skills/manim/
├── SKILL.md              # "수학 애니메이션 요청 시 이 규칙으로 코드를 써라"
├── references/
│   ├── mobjects.md       # 166개 Mobject 치트시트
│   ├── animations.md     # 78개 Animation 치트시트
│   └── pitfalls.md       # 자주 나는 에러와 해결법
└── scripts/
    └── render.sh         # manimgl -w 래퍼
```

**LLM이 ManimGL/ManimCommunity 문법을 혼동하는 것이 manim AI 활용의 최대 장애물**인데,
스킬이 이를 해결합니다.

**B. MCP 서버로 감싸기 (중급, 2~3일)**

```python
@tool
def render_animation(scene_code: str, quality: str = "low") -> str:
    """Python 코드를 받아 렌더링하고 영상 경로를 반환"""

@tool
def preview_frame(scene_code: str) -> Image:
    """-so로 마지막 프레임만 뽑아 이미지 반환 → LLM이 눈으로 검증"""
```

`preview_frame`이 핵심입니다. **LLM이 결과를 보고 스스로 고치는 루프**가 완성됩니다.

> ⚠️ MCP로 만들면 임의 Python 코드를 실행하는 구조이므로
> **반드시 샌드박스(Docker, 네트워크 차단, 타임아웃)에서** 돌려야 합니다.

---

### Q3. API 토큰이 필요한가?

**전혀 필요 없습니다. 완전 무료, 완전 오프라인입니다.**

전수조사 결과:

```sh
$ grep -rn "api_key|API_KEY|token|requests.get|urlopen" manimlib/
(결과 없음)
```

- 외부 서버 통신 코드 **0건**
- 인증·과금·계정 개념 **없음**
- 유일한 네트워크 관련 코드는 `utils/file_ops.py:33`의 `validators.url()` —
  사용자가 넣은 경로가 URL인지 **판별**하는 용도
- MIT 라이선스 → 상업적 사용·수정·재배포·유료 판매 전부 무료
- 모든 연산이 로컬 GPU에서 수행 → 인터넷을 끊어도 작동

#### 토큰이 필요해지는 경우

| 조합 | 토큰 필요? |
|---|---|
| manim만 사용 | ❌ |
| **AI가 manim 코드를 생성** | ✅ Claude/OpenAI API |
| AI 음성 내레이션 | ✅ TTS API (ElevenLabs 등) |
| 클라우드 GPU 렌더링 | ✅ AWS/GCP 요금 |
| 웹 서비스화 | ✅ 서버 비용 |

→ 실제 원가는 **"manim 0원 + LLM API 비용"**. 사업성 측면에서 유리한 구조입니다.

---

### Q4. AI 에이전트 구축에 도움이 되는가?

**매우 도움이 됩니다.** manim은 "AI가 쓰기 좋은 도구"의 조건을 거의 다 갖췄습니다.

| 조건 | manim |
|---|---|
| 입력이 텍스트인가 | ✅ Python 코드 |
| 출력이 명확한 파일인가 | ✅ mp4 / png |
| 결정론적인가 | ✅ |
| 에러가 구조적인가 | ✅ Python 트레이스백 |
| **LLM이 결과를 검증할 수 있는가** | ✅ `-so`로 프레임 → 이미지 확인 ★결정적 |
| 토큰·인증 장벽 | ✅ 없음 |
| 라이선스 | ✅ MIT |

대부분의 생성 도구는 AI가 결과를 못 봅니다. manim은 `-so` 한 번으로 이미지를 뽑아
**멀티모달 LLM이 "수식이 화면 밖으로 나갔다"를 판단하고 코드를 고칠 수 있습니다.**

#### 에이전트 아키텍처

```
사용자: "베이즈 정리를 중학생도 알게 설명해줘"
   ↓
① 기획 에이전트 (Planner)
   - 설명 순서를 장면으로 분할
   - 출력: 씬 리스트 + 내레이션 대본
   ↓
② 코드 생성 에이전트 (Coder)
   - ManimGL 문법으로 Python 작성
   - ★스킬/RAG로 문법 혼동 방지
   ↓
③ 렌더 & 검증 에이전트 (Verifier)
   - manimgl -sl 로 빠르게 렌더
   - 에러 → ②로 피드백 (최대 N회)
   - 성공 → -so 이미지를 VLM에 제시
     "화면 이탈 / 겹침 / 가독성" 체크
   ↓
④ 합성 에이전트 (Composer)
   - TTS 내레이션 생성
   - FFmpeg로 영상+음성 결합, 자막 삽입
   ↓
최종 mp4
```

#### 구축 시 반드시 해결할 4가지

**① ManimGL vs ManimCommunity 문법 혼동 — 최대 난관**

LLM 학습 데이터에 두 버전이 섞여 있어, 그냥 시키면 `from manim import *`(Community)와
`Create()`(Community 전용, ManimGL은 `ShowCreation()`)를 뱉습니다.

대응:
- 시스템 프롬프트에 **"ManimGL 전용. `from manimlib import *`. `Create` 금지, `ShowCreation` 사용"** 명시
- `example_scenes.py` 12개 예제를 few-shot으로 주입 (826줄 ≈ 1만 토큰, 프롬프트 캐싱으로 저렴)
- API 화이트리스트 자동 생성 후 정적 검증:

```sh
grep -rh "^class" manimlib/mobject/ manimlib/animation/ > allowed_api.txt
```

목록에 없는 클래스를 쓰면 렌더링 전에 차단. **정적 검증이 LLM 재시도보다 싸고 빠릅니다.**

**② 보안 — LLM이 짠 코드를 실행하는 구조**
- Docker 격리, 네트워크 차단(`--network=none`), 타임아웃
- `os`, `subprocess`, `eval` 등 AST 레벨 차단

**③ 렌더링 시간 — 루프의 병목**
- 검증 단계는 **반드시 `-sl` 또는 `-sol`**, 최종만 `--hd`
- 씬 분할 후 병렬 렌더링 → FFmpeg concat

**④ LaTeX 에러 — 실패 1순위**
- 렌더링 전 LaTeX만 따로 컴파일해 검증
- 자주 틀리는 패턴(`\mathds` 패키지 누락, 중괄호 불일치)을 프롬프트에 명시

#### 학습 교보재로서의 가치
- 코드 생성 + 실행 + 검증 루프(ReAct 패턴)의 교과서적 사례
- 멀티모달 자기검증 실습
- 샌드박싱 실습
- 결과가 영상이라 성공/실패가 눈에 보임 → 디버깅이 즐겁습니다

---

### Q5. 수익화 아이디어가 있는가?

있습니다. → **4장에서 상세히 다룹니다.**

요약: ① 유튜브 교육 채널 ② 영상 제작 외주 ③ 온라인 강의 판매 ④ 에셋·템플릿 판매
⑤ AI 영상생성 SaaS ⑥ 기업 B2B 설명영상 ⑦ 교재·출판 부가영상 ⑧ 스톡 영상 판매

---

### Q6. React나 PHP로 만들 수 있는가?

**엔진 이식은 비현실적, 웹 서비스로 감싸는 것은 완전히 가능합니다.**

#### ❌ 엔진을 React/PHP로 재구현 — 권장하지 않음

| 이유 | 설명 |
|---|---|
| 규모 | 28,400줄. 베지에 수학, GPU 셰이더 11개, 3D 변환 포함 |
| 수치 연산 | `numpy`, `scipy`, `sympy` — JS/PHP에 동급 대체재 없음 |
| LaTeX | 수식을 벡터 도형으로 쪼개는 로직. 브라우저에서 LaTeX 컴파일 불가 |
| PHP 특히 부적합 | 요청-응답 모델 ↔ 실시간 GPU 렌더링 루프와 상극 |

웹 네이티브 대안이 이미 있습니다: **`motion-canvas`**(TypeScript), **`remotion`**(React).
"웹에서 코드로 영상 만들기"가 목적이라면 그쪽이 맞습니다.

#### ✅ 웹 서비스로 감싸기 — 정답이고 매우 현실적

```
React 프론트엔드
 ┌───────────────┬──────────────────┐
 │ Monaco Editor │  영상 미리보기      │
 │ (코드 입력)     │  <video> 태그     │
 └───────────────┴──────────────────┘
 [템플릿 갤러리] [AI로 생성] [렌더링 ▶]
          │ POST /api/render { code, quality }
          ▼
FastAPI 백엔드 (Python)
 - 코드 정적 검증 (AST 화이트리스트)
 - 작업 큐 등록 (Celery + Redis)
 - 202 Accepted + job_id 반환
          ▼
렌더 워커 (Docker, GPU 인스턴스)
 - 격리 컨테이너에서 manimgl -w 실행
 - 타임아웃 120초, 네트워크 차단
 - 결과 mp4 → S3 업로드
          ▼
WebSocket으로 진행률 push → React가 영상 표시
```

**React 구현 포인트**
- Monaco Editor 자동완성에 `manimlib/`에서 추출한 166 Mobject + 78 Animation 목록 주입
- 렌더링은 비동기 — 폴링보다 WebSocket
- 저화질 미리보기 먼저, 고화질은 "다운로드" 버튼에서

**PHP를 쓰려면** API 게이트웨이 / 결제 / 사용자 관리만 PHP, 렌더링은 Python 마이크로서비스로 분리.
기존 PHP 서비스(워드프레스 등)에 붙이는 경우 합리적입니다.

```php
// Laravel 예시
$response = Http::post('http://manim-service:8000/render', [
    'code' => $request->input('code'),
    'quality' => 'low',
]);
return response()->json(['job_id' => $response->json('job_id')]);
```

**현실적 난관**

| 문제 | 영향 | 대응 |
|---|---|---|
| GPU 서버 비용 | 월 수십만원~ | 저화질 무료 / 고화질 유료, 스팟 인스턴스 |
| 렌더링 시간 | 수초~수분 | 큐 + WebSocket 진행률, 기대치 관리 |
| 보안 | 치명적 | Docker 격리 + AST 검증 + 네트워크 차단 |
| LaTeX 설치 | 이미지 6GB | 경량 TeX만 담은 커스텀 Docker 이미지 |
| 동시 사용자 | GPU 경합 | 워커 풀 + 유저별 레이트 리밋 |

**개발 난이도**
- MVP (코드 입력 → 영상 출력): **1~2주**
- AI 코드 생성 추가: **+1주**
- 결제·사용자 관리·템플릿 갤러리: **+3~4주**
→ **1인 개발로 2달 내 베타 가능**

---

### Q7. 유튜브 강의 영상으로 제작 가능한가?

**가능하고 매우 유망합니다. 한국어 manim 콘텐츠가 거의 없습니다.**

#### 왜 유망한가

| 근거 | 설명 |
|---|---|
| 수요의 증거 | 3Blue1Brown 구독자 700만+. "저런 영상 어떻게 만들어요?" 댓글이 단골 |
| 한국어 자료 공백 | 한글 manim 강의 거의 없음. 영어 자료도 ManimCommunity 중심 → **ManimGL 한국어는 거의 무주공산** |
| 썸네일 경쟁력 | 결과물이 예뻐 클릭률 높음 |
| 적당한 진입장벽 | 쉽지도 불가능하지도 않음 → 강의 수요의 최적 지점 |
| 메타 구조 | manim 강의 영상 자체를 manim으로 만들면 그게 포트폴리오 |

#### 커리큘럼 20강

**시즌 1: 입문 (5강) — 조회수 담당**

| 강 | 제목 | 핵심 |
|---|---|---|
| 1 | 3Blue1Brown 영상, 당신도 만들 수 있습니다 | 결과물 먼저 (훅) |
| 2 | 설치 완전정복 — 에러 10가지 잡기 | ★**최다 조회 예상**. GL/Community 구분, FFmpeg, LaTeX |
| 3 | 첫 애니메이션 10줄 | Circle → Square 변신 |
| 4 | Mobject·Animation·Scene 3대 개념 | |
| 5 | CLI 옵션으로 작업 10배 빠르게 | `-sl`, `-n`, `-p` |

**시즌 2: 핵심 기술 (7강)**

| 강 | 제목 |
|---|---|
| 6 | 좌표계와 배치 — `shift`, `next_to`, `to_edge` |
| 7 | LaTeX 수식 + `TransformMatchingTex` ★ |
| 8 | 그래프와 좌표축 — `Axes`, 함수, 접선 |
| 9 | Transform 완전정복 |
| 10 | Updater — 실시간으로 변하는 객체 |
| 11 | `ValueTracker`로 슬라이더 만들기 |
| 12 | 3D — 구, 곡면, 카메라 회전 |

**시즌 3: ManimGL 전용 무기 (4강) — 차별화 담당**

| 강 | 제목 |
|---|---|
| 13 | `self.embed()` 실시간 인터랙티브 개발 ★Community엔 없는 기능 |
| 14 | 마우스로 편집하는 `InteractiveScene` |
| 15 | `custom_config.yml`로 내 스타일 만들기 |
| 16 | `-p` 발표자 모드로 PPT 대체 |

**시즌 4: 실전·수익화 (4강) — 전환율 담당**

| 강 | 제목 |
|---|---|
| 17 | 피타고라스 정리 증명 영상 처음부터 끝까지 |
| 18 | 음성 내레이션 + 편집 + 유튜브 업로드 |
| 19 | **AI(Claude)로 manim 코드 자동 생성** ★트렌드 |
| 20 | manim으로 수익 내는 6가지 방법 |

#### 제작 실무 팁

- 화면 녹화: OBS Studio (무료)
- 레이아웃: 왼쪽 VSCode / 오른쪽 manim 미리보기 창
  (`default_config.yml`의 `position_string: UR`이 정확히 이 용도)
- 녹화 중에는 `-sl`로 렌더링 — 대기 시간이 짧아야 시청 유지율이 유지됩니다
- **에러가 나면 편집으로 지우지 말고 그대로 보여주고 고치세요.** 입문자가 가장 원하는 장면입니다

**조회수 전략**
- **2강(설치)에 가장 공들이기.** "manim 설치 에러" 검색 수요가 꾸준해 채널 유입구가 됩니다
- 쇼츠 분할: "10줄로 만드는 수식 변형" 30초 클립 → 긴 영상 유입
- 강의별 완성 코드를 GitHub 공개 → 설명란 링크 → 신뢰도·재방문

**수익 구조**

```
유튜브 광고 (무료 20강)
   ↓ 신뢰 확보
유료 심화 강의 (인프런/클래스101, 10~20만원)
   ↓
1:1 컨설팅 / 기업 교육
   ↓
영상 제작 외주 수주
```

유튜브 광고만으로는 적습니다. **무료 강의는 마케팅, 수익은 유료 강의와 외주**에서 나옵니다.

**예상 성과 (현실적)**
- 3개월 / 20강 완주: 구독자 1,000~3,000명
- 설치 강의가 터지면: 5,000~10,000명
- 유료 전환: 구독자 3,000명 → 수강생 50~100명 → **500만~2,000만원**

---

## 4. 수익화 아이디어 상세

> 전제: MIT 라이선스로 **상업적 이용·수정·유료 판매에 제약이 없습니다.**
> 저작권 표기만 유지하면 되고, **소프트웨어 원가는 0원**입니다.

### 4.0 한눈에 비교

| # | 모델 | 초기비용 | 난이도 | 수익화 속도 | 월 기대수익 | 확장성 |
|---|---|---|---|---|---|---|
| 1 | 유튜브 교육 채널 | 거의 0 | ★★☆ | 느림 (6개월+) | 50만~500만 | ★★★★★ |
| 2 | **영상 제작 외주** | 거의 0 | ★★☆ | **빠름 (2주)** | 200만~1,000만 | ★★☆ |
| 3 | 온라인 강의 판매 | 거의 0 | ★★☆ | 중간 (3개월) | 100만~2,000만 | ★★★★ |
| 4 | 에셋·템플릿 판매 | 거의 0 | ★☆☆ | 중간 | 30만~300만 | ★★★★ |
| 5 | **AI 영상생성 SaaS** | 높음 | ★★★★★ | 느림 (6개월+) | 0~수천만 | ★★★★★ |
| 6 | 기업 B2B 설명영상 | 거의 0 | ★★★ | 중간 | 500만~3,000만 | ★★☆ |
| 7 | 교재·출판 부가영상 | 거의 0 | ★★☆ | 중간 | 협상 | ★★★ |
| 8 | 스톡 영상 판매 | 거의 0 | ★☆☆ | 느림 | 10만~100만 | ★★★★ |

---

### 4.1 유튜브 교육 채널

**두 방향**
- **A. "manim 사용법" 채널** — 경쟁 거의 없음(한국어), 유료 강의 전환 쉬움 / 시장은 니치
- **B. "수학·과학 설명" 채널 (3b1b 스타일)** — 시장 거대 / 제작시간 많고 경쟁 치열

**추천: A로 시작해 B로 확장.** A로 빨리 신뢰를 쌓고, B로 규모를 키웁니다.

**수익 구조 (광고가 전부가 아님)**

| 경로 | 비중 | 설명 |
|---|---|---|
| 광고 수익 | 10~20% | 교육 콘텐츠 RPM 2~5천원/천뷰 |
| **유료 강의 유입** | **40~50%** | ★실제 주력 |
| 외주 문의 유입 | 20~30% | 포트폴리오 역할 |
| 멤버십/후원 | 5~10% | |
| 제휴·협찬 | 가변 | 교육 플랫폼, 노트 앱 등 |

**실행 계획**

```
[1개월] 입문 5강 + GitHub 예제 공개 + 쇼츠 10개
[2~3개월] 핵심 7강 + "AI로 manim 코드 생성" 영상 → 구독자 1,000명(수익화 조건)
[4~6개월] 유료 심화 강의 출시(인프런) + 외주 포트폴리오 페이지
```

---

### 4.2 영상 제작 외주 ★가장 빠른 현금화

초기 비용 0원, **2주 안에 첫 수주 가능.**

**타겟 고객 & 단가**

| 고객 | 수요 | 단가 (추정) |
|---|---|---|
| 온라인 강의 제작자 | 수학/통계 개념 클립 | 10~30만원/분 |
| 학원·교육기관 | 홍보용, 수업용 자료 | 50~200만원/건 |
| 대학 교수 | 강의자료, 연구 발표 | 30~100만원/건 |
| 출판사 | 교재 QR 영상 | 협상 |
| 유튜버 | 채널용 설명 클립 | 10~50만원/건 |
| **스타트업/기업** | 기술·알고리즘 설명 | **200~1,000만원/건** ★최고단가 |
| 논문 저자 | graphical abstract, 학회 발표 | 50~150만원/건 |

**단가 책정 가이드**

```
기본: 완성 영상 1분당 15~30만원
  + LaTeX 수식 많음     → +20%
  + 3D 애니메이션       → +50%
  + 내레이션 녹음 포함   → +30%
  + 급행(1주 이내)      → +50%
  + 소스코드 제공       → +30%   ← 추천
```

**차별화 포인트 (vs 애프터이펙트 모션그래퍼)**

1. **수정이 압도적으로 싸다** — "3을 5로" → 코드 한 글자. AE는 전체 재작업
2. **양산이 된다** — "같은 형식으로 10개 더" → for문
3. **수학적 정확성** — 함수 그래프가 실제 계산 결과. 손으로 그린 근사가 아님
4. **소스코드 납품 가능** — 고객이 직접 수정 → 신뢰. 유지보수 구독 계약으로 연결

제안서에 **"수정 무료 3회 + 소스코드 제공"** 을 명시하면 강력합니다.

**수주 경로**

1. **포트폴리오 먼저** — 샘플 5개(미적분/선형대수/확률/물리/알고리즘)를 유튜브·노션 공개
2. **크몽·숨고 등록** — "수학 애니메이션 제작" 카테고리 선점
3. **콜드 메일** — 수학 유튜버·강의 제작자에게 "귀 채널 X번 영상의 Y부분을 이렇게
   만들어봤습니다" + 실제 샘플 첨부 → 전환율 매우 높음
4. **학회·대학 접근** — 교수에게 "논문 figure를 애니메이션으로" 제안
5. **유튜브 채널이 영업사원** — 4.1과 결합

---

### 4.3 온라인 강의 판매

**플랫폼 비교**

| 플랫폼 | 수수료 | 특징 |
|---|---|---|
| 인프런 | 20~30% | 개발자 타겟. manim과 궁합 최고 |
| 클래스101 | 높음 | 취미·크리에이터. 노출 많음 |
| 유데미 | 50%+ | 글로벌. **영어로 만들면 시장 10배** ★ |
| 자체 판매 (Gumroad 등) | 결제수수료만 | 마진 최고, 마케팅 직접 |

**상품 라인업**

```
[무료] 유튜브 20강           → 유입
[입문] 49,000원             → manim 기초 — 첫 영상 만들기 (5시간)
[심화] 149,000원            → ManimGL 완전정복 (15시간, 실전 프로젝트 3개)
[프로] 399,000원            → AI + manim 영상 자동화 파이프라인 ★차별화
[1:1]  시간당 10~20만원     → 컨설팅
[기업] 건당 300~1,000만원   → 출장 교육
```

**핵심 차별화: "AI + manim" 강의 (경쟁자 없음)**

1. Claude/GPT로 manim 코드 생성
2. ManimGL 문법을 LLM에 정확히 가르치는 프롬프트 설계
3. 치트시트 자동 추출 (`manimlib/`에서 API 목록 뽑기)
4. 렌더링 → 에러 → 자동 수정 루프
5. `-so` 프레임을 멀티모달 LLM에 보여 자동 검증
6. TTS 내레이션 + FFmpeg 자동 합성
7. 전체 파이프라인을 Claude Skill / MCP 서버로 패키징

→ **"하루에 교육 영상 10개 자동 생산"** 결과물이 있으면 39만원도 비싸지 않습니다.

**영어 시장 ★최대 기회**
ManimGL은 영어권에도 체계적 강의가 없습니다(Community 자료만 많음).
한국어로 먼저 만들고 → AI 번역 + 영어 TTS 더빙 → 거의 무비용 재판매.
3Blue1Brown 구독자 700만이 잠재 고객입니다.

---

### 4.4 에셋·템플릿 판매

**가장 쉽고, 한번 만들면 계속 팔립니다.**

| 상품 | 가격 | 설명 |
|---|---|---|
| 중학 수학 애니메이션 팩 | 49,000원 | 교과 단위 30종 + 소스코드 |
| 고등 미적분 팩 | 79,000원 | 극한·미분·적분 50종 |
| 통계·확률 팩 | 69,000원 | 정규분포, 중심극한정리, 베이즈 |
| 선형대수 팩 | 79,000원 | 행렬변환, 고유값, 기저 |
| 물리 시뮬레이션 팩 | 89,000원 | 진자, 파동, 전자기장 |
| 알고리즘 시각화 팩 | 69,000원 | 정렬, 탐색, 그래프 |
| **Mobject 확장 라이브러리** | 99,000원 | 기본 166개에 없는 커스텀 도형 |
| **커스텀 테마 팩** | 39,000원 | `custom_config.yml` 색상 테마 모음 |

**판매 채널**: Gumroad / 레몬스퀴지, 크몽 디지털 콘텐츠, 자체 사이트(4.6의 React와 연결),
GitHub Sponsors(무료 공개 + 후원)

**왜 쉬운가**
- 한 번 만들면 재고 없이 무한 판매
- 외주 작업물을 일반화해 재판매 가능 (계약서 저작권 조항 확인 필요)
- 유튜브 강의 예제를 묶어 팩으로 → 콘텐츠 재활용

**프리미엄 전략**: 무료 팩 10종을 GitHub 공개 → 신뢰 확보 → 유료 50종 팩 판매.
무료 팩이 영업사원 역할을 합니다.

---

### 4.5 AI 영상 생성 SaaS ★최대 잠재력, 최고 난이도

> Q4(AI 에이전트) + Q6(웹 서비스) 결합. **"텍스트 한 줄 → 수학 애니메이션"**

**컨셉**: 사용자가 "피타고라스 정리를 시각적으로 증명해줘" 입력 → 30초 후 mp4 다운로드

**요금제**

| 플랜 | 월 요금 | 내용 |
|---|---|---|
| Free | 0원 | 월 3개, 480p, 워터마크 |
| Starter | 9,900원 | 월 30개, 1080p, 워터마크 없음 |
| Pro | 29,900원 | 월 200개, 4K, 소스코드 다운로드, API |
| Team | 99,000원 | 무제한, 협업, 브랜드 커스텀 |
| Enterprise | 협의 | 온프레미스, SLA |

**타겟**: 교사·강사, 온라인 강의 제작자, 유튜버, 학생, 기업 교육팀

**단계적 구축 ★처음부터 SaaS를 만들지 마세요**

```
[0단계] 나 혼자 쓰는 CLI 도구               ← 1주
[1단계] Claude Skill로 패키징 + 무료 공개     ← 2주  ★수요 검증
[2단계] 웹 데모 (코드 입력 → 렌더링)          ← 2주  React + FastAPI, 무료
[3단계] AI 생성 + 유료 전환 (결제, 사용량 제한) ← 1개월
[4단계] SaaS 정식 런칭 (팀 기능, API, 마켓)   ← 2개월
```

0~1단계를 먼저 하는 이유: **서버 비용 0원으로 수요를 검증**할 수 있습니다.
반응이 없으면 거기서 멈추면 됩니다.

**원가 구조 (월 사용자 1,000명 가정)**

| 항목 | 비용 | 비고 |
|---|---|---|
| manim 라이선스 | **0원** | MIT ★ |
| LLM API (Claude) | 30~100만원 | 캐싱·저가 모델 혼용으로 절감 |
| GPU 렌더 서버 | 50~200만원 | 스팟 인스턴스 |
| 스토리지 (S3) | 5~20만원 | 영상 30일 후 삭제 |
| 기타 (DB, CDN, 도메인) | 10~30만원 | |
| **합계** | **100~350만원** | |
| 매출 (전환 5% = 50명 × 2만원) | 100만원 | ⚠️ 적자 |
| 매출 (전환 10% = 100명 × 3만원) | 300만원 | 손익분기 근처 |

→ **전환율이 생명.** 무료 티어를 너무 후하게 주면 망합니다.

**비용 절감 핵심 아이디어**

1. **템플릿 우선 전략** — 자주 나오는 요청 100종(피타고라스, 미분 정의, 정규분포 등)을
   **미리 렌더링해 캐싱**. LLM도 GPU도 안 씀. 요청의 약 70% 커버 가능
2. **파라미터화된 템플릿** — "y=x²의 접선"과 "y=x³의 접선"은 같은 템플릿에 숫자만 다름
3. **저화질 먼저** — 미리보기 480p, 고화질은 다운로드 시에만
4. **프롬프트 캐싱** — 예제 코드 1만 토큰을 매번 전송하지 말고 캐싱 → LLM 비용 90% 절감
5. **정적 검증으로 재시도 줄이기** — AST로 미리 걸러 렌더링 실패 감소

**리스크**

| 리스크 | 대응 |
|---|---|
| 품질 불안정 | 템플릿 비중 높이기, 미리보기 후 재생성 |
| GPU 비용 폭증 | 엄격한 레이트 리밋, 큐 대기 허용 |
| 보안 사고 | Docker + 네트워크 차단 + AST 검증 |
| 대기업 진입 | 니치(수학 특화) 방어, 커뮤니티 구축 |
| 수요 부족 | 0~1단계에서 미리 검증 ★ |

---

### 4.6 기업 B2B 설명 영상 ★최고 단가

**수요가 있는 곳**

| 업종 | 필요한 영상 | 단가 |
|---|---|---|
| AI/ML 스타트업 | "우리 알고리즘의 작동 원리" | 300~1,000만원 |
| 핀테크 | 금융 상품 구조, 수익률 시뮬레이션 | 200~800만원 |
| 블록체인 | 합의 알고리즘, 암호 원리 | 300~1,000만원 |
| 바이오·제약 | 분자 구조, 약물 작용 기전 | 300~800만원 |
| 반도체·제조 | 공정 원리 | 500~2,000만원 |
| 컨설팅·리서치 | 데이터 인사이트 시각화 | 200~500만원 |

**왜 manim이 적합한가**
- IR·투자유치 자료: "수학적으로 정확해 보이는" 비주얼이 신뢰를 줌
- 기술 블로그·논문 보조 영상, 채용 브랜딩, 학회 발표

**접근 전략**
1. 산업 하나를 정해 **그 산업 샘플 3개** 제작
   (예: AI 스타트업용 — Transformer attention, 경사하강법, 임베딩 공간)
2. 업계 마케팅/IR 담당자에게 콜드 메일 + 샘플
3. 첫 고객에게 원가 수준으로 제공해 **레퍼런스 확보** → 이후 정가
4. 레퍼런스 3개 쌓이면 단가 2배

---

### 4.7 교재·출판 부가 영상

- 출판사가 교재에 **QR코드 → 해설 애니메이션**을 연결하는 추세
- 수학·물리 교재 한 권에 영상 50~200개 필요 → **대량 수주**
- manim의 양산 능력이 결정적 우위 (같은 형식 100개를 for문으로)
- 계약 형태: 건당 / 권당 패키지 / 로열티
- EBS, 교육청, 디지털교과서 사업도 타겟

---

### 4.8 스톡 영상 판매

- 교육용 수학 애니메이션 클립을 스톡 사이트에 등록
- 플랫폼: Envato Elements, Pond5, Motion Array, Adobe Stock
- **투명 배경(`-t` 옵션)** 으로 만들면 합성 소재로서 가치 상승
- 수동적 수익. 개당 적지만 500개 올려두면 월 수십만원
- 외주·강의 작업물을 그대로 재활용 → 추가 노력 거의 0

---

## 5. 종합 추천 전략

### 5.1 단계별 로드맵

```
[0~1개월] 역량 + 포트폴리오
  - 설치하고 예제 12개 전부 돌려보기
  - 샘플 5개 제작 (미적분/선형대수/확률/물리/알고리즘)
  - GitHub + 유튜브 공개
  수익: 0원 (투자 기간)

[1~2개월] 첫 현금 흐름 ← 외주
  - 크몽·숨고 등록
  - 수학 유튜버 20명에게 콜드 메일 + 샘플
  - 유튜브 입문 5강 (설치 강의 집중)
  수익: 50만~300만원

[2~4개월] 자산화 ← 강의 + 에셋
  - 유튜브 핵심 7강
  - 에셋 팩 2종 출시
  - 인프런 입문 강의 출시
  수익: 200만~800만원/월

[4~6개월] 차별화 ← AI 결합
  - AI + manim 파이프라인 구축 (Q4 내용)
  - Claude Skill로 패키징해 무료 공개 → 화제성
  - "AI + manim" 심화 강의 (39만원)
  - 외주 생산성 5배 → 단가 유지하며 물량 증가
  수익: 500만~2,000만원/월

[6개월~] 확장 ← B2B + SaaS
  - 기업 B2B 영업 (최고 단가)
  - 영어 강의로 글로벌 진출
  - 수요 검증되면 SaaS 베타
  수익: 1,000만원+/월
```

### 5.2 핵심 원칙 4가지

**① 외주 → 강의 → 에셋 → SaaS 순서를 지키세요.**
외주로 현금흐름을 만들며 실력·포트폴리오를 쌓고, 그 결과물을 강의와 에셋으로 재활용하고,
수요가 검증된 뒤 SaaS에 투자합니다. **처음부터 SaaS를 만들면 돈과 시간을 잃을 확률이 높습니다.**

**② 한 작업물을 4번 팔아먹으세요.**

```
외주 제작물 하나
  → ① 외주비 수령
  → ② 포트폴리오 (영업 자산)
  → ③ 유튜브 강의 예제 (콘텐츠)
  → ④ 일반화해서 에셋 팩에 포함 (재판매)
```

이것이 manim 비즈니스의 핵심 레버리지입니다.

**③ AI 결합이 진짜 차별화입니다.**
수학 애니메이션 제작자는 소수지만 존재합니다. 그러나 **"AI로 자동 생산하는 사람"은 거의 없습니다.**
Q4의 에이전트를 구축하면 생산성이 5~10배가 되고, 그 자체가 판매 가능한 상품
(강의·스킬·SaaS)이 됩니다.

**④ 영어 시장을 잊지 마세요.**
3Blue1Brown 구독자 700만이 잠재 고객입니다. 한국어로 만든 뒤 AI 번역 + TTS 더빙으로
거의 무료로 재판매할 수 있습니다. 시장이 10배 이상 커집니다.

### 5.3 현실적 경고

- **manim 자체로는 돈이 안 됩니다.** 돈은 "manim으로 무엇을 만들어 누구에게 파는가"에서
  나옵니다. 도구에 매몰되지 마세요.
- **제작 시간을 과소평가하지 마세요.** 1분짜리 완성 영상에 초보는 5~10시간 걸립니다.
  외주 단가를 낮게 잡으면 시간당 수익이 최저임금 이하가 됩니다.
- **수학 실력도 필요합니다.** manim은 도구일 뿐, "무엇을 어떻게 설명할지"가 진짜 실력입니다.
- **SaaS는 GPU 비용 때문에 적자 위험이 큽니다.** 반드시 0~1단계로 수요를 검증하고
  템플릿 캐싱 전략을 먼저 세우세요.

---

## 부록 A. 링크 모음

| 구분 | URL |
|---|---|
| **이 저장소** | https://github.com/bmshin94/manim |
| 원본 (upstream) | https://github.com/3b1b/manim |
| 커뮤니티 버전 (다른 프로젝트) | https://github.com/ManimCommunity/manim |
| 3b1b 영상 소스코드 | https://github.com/3b1b/videos |
| 공식 문서 | https://3b1b.github.io/manim/ |
| 커뮤니티 버전 문서 | https://docs.manim.community/ |
| 중국어 문서 | https://docs.manim.org.cn/ |
| PyPI (manimgl) | https://pypi.org/project/manimgl/ |
| manim Reddit | https://www.reddit.com/r/manim/ |
| manim Discord | https://discord.com/invite/bYCyhM9Kz2 |
| manim_sandbox (추가 예제) | https://github.com/manim-kindergarten/manim_sandbox |
| 3Blue1Brown 채널 | https://www.3blue1brown.com/ |
| 인터랙티브 워크플로 데모 영상 | https://www.youtube.com/watch?v=rbu7Zu5X1zI |
| 웹 네이티브 대안 — motion-canvas | https://github.com/motion-canvas/motion-canvas |
| 웹 네이티브 대안 — remotion | https://github.com/remotion-dev/remotion |

## 부록 B. 빠른 참조 치트시트

```python
from manimlib import *

class Demo(Scene):
    def construct(self):
        # --- 객체 생성 ---
        c = Circle(radius=1, color=BLUE)
        s = Square(side_length=2)
        t = Text("안녕")
        f = Tex(r"e^{i\pi} + 1 = 0")
        ax = Axes(x_range=(-3, 3), y_range=(-2, 2))

        # --- 배치 ---
        c.shift(2 * LEFT)          # 이동
        s.next_to(c, RIGHT)        # 옆에 붙이기
        t.to_edge(UP)              # 화면 끝으로
        f.move_to(ORIGIN)          # 중앙으로

        # --- 스타일 ---
        c.set_fill(BLUE, opacity=0.5)
        c.set_stroke(WHITE, width=4)
        t.set_backstroke(width=5)  # 글자 뒤 외곽선(가독성)

        # --- 애니메이션 ---
        self.play(ShowCreation(c))
        self.play(Write(t))
        self.play(Transform(c, s))
        self.play(c.animate.scale(2).set_color(RED))   # .animate 체이닝
        self.play(FadeOut(c), FadeIn(f))               # 동시 재생
        self.play(LaggedStart(*[FadeIn(m) for m in [c, s, t]]))  # 시차 재생
        self.wait(2)

        # --- 개발 중 중단점 ---
        # self.embed()   # 여기서 터미널이 열림
```

```sh
# 자주 쓰는 명령
manimgl file.py Scene          # 미리보기
manimgl file.py Scene -sl      # 저화질 최종 프레임 (가장 빠른 확인)
manimgl file.py Scene -l       # 저화질 전체 재생
manimgl file.py Scene -l -n 5  # 5번째 애니메이션부터
manimgl file.py Scene -o --hd  # 1080p 저장 후 열기
manimgl file.py Scene -t -w    # 투명 배경 저장 (합성용)
manimgl file.py -a -w          # 파일 내 모든 Scene 일괄 렌더링
manimgl file.py Scene -p       # 발표자 모드 (PPT 대체)
manimgl --clear-cache          # LaTeX 캐시 삭제
```

---

*이 문서는 저장소 전수조사(코드 직접 분석) 결과를 바탕으로 작성되었습니다.
수익 금액은 시장 상황에 따른 추정치이며 보장된 값이 아닙니다.*
