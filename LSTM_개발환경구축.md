# Anaconda 기반 LSTM 개발 환경 구축 가이드 (Windows, Jupyter 미사용)
2026-08-12 기준 작성 | GPU: Intel(R) Arc(TM) 130T GPU (16GB)

---

## 1단계: Anaconda 설치

1. [anaconda.com/download](https://www.anaconda.com/download)에서 **Windows용 설치 파일(64-bit)** 다운로드
2. `.exe` 파일 실행
3. 설치 옵션
   - **"Just Me"** 선택 (권장, 관리자 권한 불필요)
   - 설치 경로는 기본값 유지 권장
   - "Add Anaconda to my PATH" 체크박스는 비활성 상태 유지 권장 (Anaconda Prompt 사용 예정이므로)
4. 설치 완료까지 10~20분 소요

---

## 2단계: 설치 확인

시작 메뉴에서 **"Anaconda Prompt"** 실행 후:

```bash
conda --version
python --version
```

---

## 3단계: conda 업데이트 (선택, 권장)

```bash
conda update conda
conda update anaconda
```

---

## 4단계: 가상환경 생성 및 활성화

```bash
conda create -n lstm_env python=3.14.6
conda activate lstm_env
```

프롬프트 앞에 `(lstm_env)`가 표시되면 성공입니다. 이후 모든 작업은 이 환경 안에서 진행합니다.

설치 여부 확인:

```bash
pip list
pip show torch
```

PyTorch(XPU 빌드) 설치:

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu
```

데이터 처리 라이브러리 확인 및 설치:

```bash
pip list
pip show pandas
conda install numpy pandas scikit-learn matplotlib seaborn -y
```

---

## 5단계: GPU 사용 여부 결정

- **NVIDIA GPU 보유 시**: conda로 CUDA/cuDNN을 쉽게 설치할 수 있습니다.
- **GPU 없음 / CPU로 충분**: 6단계로 바로 이동
- **Intel Arc(XPU) GPU 보유 시 (본 환경)**: 아래 XPU 인식 확인 절차를 따릅니다.

### NVIDIA GPU 사용 시 (해당 시에만)

```bash
conda install -c conda-forge cudatoolkit=11.8 cudnn=8.6
```

버전은 설치할 TensorFlow/PyTorch와의 호환성을 공식 문서에서 반드시 확인하세요.

### Intel Arc(XPU) 인식 확인

```bash
python -c "import torch; print(torch.__version__, torch.xpu.is_available(), torch.xpu.get_device_name(0))"
```

**확인된 결과:**

```
2.13.0+xpu  True  Intel(R) Arc(TM) 130T GPU (16GB)
```

- `2.13.0+xpu` → XPU 빌드 정상 설치 확인
- `True` → Arc 130T가 PyTorch에서 정상 인식됨
- 디바이스 이름도 정확히 출력됨

이후 학습 코드에서 `device = torch.device("xpu")`로 지정하고 `model.to(device)`, `tensor.to(device)`만 해주면 GPU 가속이 적용됩니다.

> **참고**: TensorFlow는 Windows에서 Intel Arc(XPU) GPU 가속을 지원하지 않습니다. TensorFlow를 설치하면 CPU 전용으로만 동작하며 GPU는 전혀 사용되지 않습니다.

---

## 6단계: 핵심 라이브러리 설치

```bash
# 딥러닝 프레임워크 (XPU 가속은 PyTorch만 지원 — 5단계 참고)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu

# 데이터 처리 및 분석
conda install numpy pandas scikit-learn matplotlib seaborn -y
```

---

## 7단계: 프로젝트 구조 잡기 및 내보내기

```
lstm-project/
├── data/               # 원본/전처리 데이터
├── models/             # 학습된 모델 저장
├── src/
│   ├── preprocess.py   # 정규화, 윈도우 슬라이딩 등
│   ├── model.py        # LSTM 모델 정의
│   └── train.py        # 학습 실행 스크립트
└── environment.yml     # 환경 정의 파일
```

## 8단계: 환경 내보내기/가져오기/삭제하기

환경 내보내기:

```bash
conda env export > environment.yml
```

환경 가져오기:

```bash
[동일한 이름]
conda env create -f environment.yml

[새이름으로 생성]
conda env create -f environment.yml -n 새이름

[갱신]
conda env update -f environment.yml --prune
```

환경 삭제하기:

```bash
conda deactivate
conda env remove -n lstm_env
```

환경 리스트:

```bash
conda env list
```

---

## 9단계: 코드 에디터 설정 (VS Code 권장)

1. https://code.visualstudio.com 에 접속해 파란색 'Download for Windows' 버튼을 클릭해 설치 파일(.exe)을 받습니다.
2. 다운로드된 .exe 파일을 실행합니다. 사용 라이선스 동의 후, 경로는 기본값 그대로 진행하면 됩니다.
3. 설치 중 '추가 작업 선택' 화면에서 다음 두 항목을 체크하는 것이 좋습니다: 'PATH에 추가'와 'Code로 열기(파일/폴더 우클릭 메뉴에 추가)'. 터미널이나 탐색기에서 바로 열 때 편해집니다.
4. 설치 완료 후 VS Code를 실행하고, 왼쪽 확장(Extensions) 아이콘(네모 모양)에서 'Python'을 검색해 Microsoft의 Python 확장을 설치합니다. Pylance는 함께 자동 설치됩니다.
5. `lstm-project` 폴더를 VS Code로 열고, 하단 상태바에서 Python 버전을 클릭해 `lstm_env` 가상환경을 선택합니다. 안 보이면 `Ctrl+Shift+P` → 'Python: Select Interpreter'로 찾을 수 있습니다.

이제 VS Code 하단 상태바에 `lstm_env`가 표시됩니다. `.py` 파일을 열고 우측 상단 ▶ 버튼(Run) 또는 터미널에서 직접 실행하면 됩니다.
