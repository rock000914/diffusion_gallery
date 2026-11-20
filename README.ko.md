# Diffusion Gallery

🌐 [English](README.en.md) · [日本語](README.md) · [简体中文](README.zh-CN.md) · **한국어**

AI 생성 이미지(Stable Diffusion 등)를 정리하고 열람하는 뷰어 소프트웨어입니다 (이전 이름: PNG Metadata Viewer).

PNG를 중심으로 메타데이터 표시, LoRA 관리, 썸네일 목록, 태그 집계, 중복 감지, 로고 오버레이까지 하나의 데스크톱 앱에 담았습니다.

AI 이미지의 제작·정리·확인에 잘 맞으며, 생성 시의 프롬프트와 LoRA 정보를 확인하면서 운용하기 쉽습니다.

## 주요 기능

- `PySide6` 기반 3분할 GUI
- 빠른 스캔, 열람, 썸네일 표시
- PNG 메타데이터 분석 및 표시
- LoRA / 모델 정보의 목록 관리와 검색
- 상세 검색: 태그·파일명·제외어·정확한 구문·정규식·날짜/해상도/크기/Steps/CFG 범위·검색 기록 ("🔍 상세" 패널)
- 중복 감지, 평가, 즐겨찾기, 태그 집계
- 캐시를 활용한 가벼운 폴더 운용
- 영어·중국어(간체)·한국어 지원 (환경설정의 "言語 / Language" 탭에서 선택, 재시작 후 반영)

## 기술 포인트

- `QThread` / `QRunnable` / `QThreadPool`로 UI가 멈추지 않는 설계
- `QSplitter` 기반의 유연한 레이아웃
- Pillow와 Qt를 조합한 이미지 처리
- 로컬 캐시와 영속 설정으로 재시작 후에도 편리
- 단일 스크립트로 배포하기 쉬운 구현

## 요구 사항

- Python 3.11+
- Windows 10 / 11

```bash
pip install -r requirements.txt
python png_viewer.py
```

## 선택적 환경 변수

- `PNG_VIEWER_DEFAULT_FOLDER`
  - 시작 시 처음에 열 폴더를 지정합니다 (환경 변수 이름은 이전 이름을 유지)

```powershell
$env:PNG_VIEWER_DEFAULT_FOLDER = "D:\Images"
python .\png_viewer.py
```
