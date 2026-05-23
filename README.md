# HRV 기반 실시간 감정 상태 분석 시스템

ESP32 Feather와 EmotiBit FeatherWing을 이용하여 사용자의 생체 신호를 수집하고, HRV(Heart Rate Variability) 기반 피처를 계산하여 사용자의 평온/긴장 상태를 실시간으로 분석하는 임베디드 프로젝트입니다.

측정된 심박 및 IBI 데이터를 기반으로 SDNN, RMSSD, LF, HF, Total Power 등의 HRV 피처를 계산하고, 사전에 학습된 MLP 모델 파라미터를 이용해 `calm`과 `tense` 값을 추론합니다. 추론 결과는 BLE Nordic UART Service를 통해 안드로이드 앱 또는 외부 수신 장치로 전송할 수 있습니다.

## 프로젝트 개요

이 프로젝트는 웨어러블 생체신호 기반 감정 분석 시스템을 목표로 합니다. EmotiBit 센서를 통해 수집한 심박 관련 데이터를 ESP32에서 실시간으로 처리하고, HRV 분석 결과를 기반으로 사용자의 긴장도와 평온도를 확률 형태로 제공합니다.

## 주요 기능

- EmotiBit FeatherWing 기반 생체 신호 수집
- 심박수 및 IBI 데이터 기반 HRV 분석
- 시간 영역 HRV 피처 계산
  - SDNN
  - RMSSD
- 주파수 영역 HRV 피처 계산
  - LF
  - HF
  - Total Power
- FFT 기반 주파수 분석
- 사전 학습된 MLP 모델을 이용한 실시간 상태 추론
- `calm`, `tense` 확률값 계산
- BLE Nordic UART Service 기반 실시간 데이터 전송
- JSON 형식의 결과 패킷 출력

## 시스템 구조

```text
EmotiBit FeatherWing
        ↓
ESP32 Feather
        ↓
IBI / Heart Rate 수집
        ↓
HRV 피처 계산
        ↓
MLP 모델 추론
        ↓
calm / tense 결과 생성
        ↓
BLE Notify 전송
        ↓
Android App 또는 외부 수신 장치
```

## 사용 기술

- C++
- Arduino Framework
- PlatformIO
- ESP32 Feather
- EmotiBit FeatherWing
- BLE Nordic UART Service
- NimBLE-Arduino
- ArduinoJson
- HRV Feature Extraction
- FFT
- MLP Neural Network Inference

## 개발 환경

```ini
platform = espressif32
board = featheresp32
framework = arduino
monitor_speed = 115200
```

## 주요 라이브러리

```ini
emotibit/EmotiBit FeatherWing
bblanchon/ArduinoJson
h2zero/NimBLE-Arduino
```

## 주요 처리 흐름

1. ESP32와 EmotiBit 센서를 초기화합니다.
2. EmotiBit에서 심박수와 IBI 데이터를 수집합니다.
3. 일정 개수의 IBI 샘플이 모이면 HRV 피처를 계산합니다.
4. SDNN, RMSSD, LF, HF, Total Power 값을 생성합니다.
5. 계산된 피처를 스케일링한 뒤 MLP 모델에 입력합니다.
6. 모델 출력값을 기반으로 평온도와 긴장도를 계산합니다.
7. 결과를 JSON 형식으로 생성합니다.
8. BLE Notify를 통해 외부 장치로 전송합니다.

## 출력 데이터 예시

```json
{
  "temp": 36.5,
  "battery": 87,
  "hr": 78.2,
  "calm": 0.73,
  "tense": 0.27
}
```

## HRV 피처 설명

| 피처 | 설명 |
|---|---|
| SDNN | IBI 값의 표준편차로, 심박 변동성의 전체적인 변화를 나타냅니다. |
| RMSSD | 연속된 IBI 차이의 제곱 평균 제곱근으로, 단기 심박 변동성을 나타냅니다. |
| LF | 저주파 영역 파워로, 자율신경계 반응과 관련된 피처입니다. |
| HF | 고주파 영역 파워로, 부교감신경 활성과 관련된 피처입니다. |
| Total Power | 전체 HRV 주파수 영역의 파워를 나타냅니다. |

## BLE 전송 방식

본 프로젝트는 Nordic UART Service UUID를 사용하여 BLE Notify 방식으로 데이터를 전송합니다.

```cpp
6E400001-B5A3-F393-E0A9-E50E24DCCA9E
```

BLE 장치 이름은 다음과 같이 설정됩니다.

```cpp
Vitals
```

## 프로젝트 특징

- 센서 데이터 수집부터 피처 계산, 모델 추론, BLE 전송까지 ESP32 내부에서 처리
- 서버 없이 임베디드 보드 단독으로 실시간 감정 상태 추론 가능
- Python에서 사용한 HRV 주파수 분석 흐름과 최대한 유사하도록 FFT 기반 로직 구현
- 안드로이드 앱과 연동 가능한 BLE UART 구조 사용
- 웨어러블 감정 분석 및 스트레스 모니터링 시스템으로 확장 가능

## 파일 구조

```text
.
├── platformio.ini
├── src
│   └── main.cpp
├── best_hrv_params.json
├── hrv_emotion_model.pkl
└── README.md
```

## 빌드 전 준비 사항

`hrv_model_params.h` 파일에 사전 학습된 모델 파라미터가 정의되어 있어야 합니다.

필요한 주요 파라미터는 다음과 같습니다.

```cpp
scaler_mean[]
scaler_scale[]
W1[][]
b1[]
W2[][]
b2[]
```

해당 값들은 Python에서 학습한 모델의 스케일러 및 MLP 가중치를 C/C++ 배열 형태로 변환하여 사용합니다.

## 실행 방법

1. PlatformIO가 설치된 VSCode를 실행합니다.
2. ESP32 Feather 보드를 PC에 연결합니다.
3. 프로젝트 폴더를 엽니다.
4. `platformio.ini` 설정을 확인합니다.
5. `hrv_model_params.h` 파일이 포함되어 있는지 확인합니다.
6. 빌드 및 업로드를 진행합니다.

```bash
pio run
pio run --target upload
pio device monitor
```

## 향후 개선 방향

- Android 앱과의 실시간 BLE 연동 고도화
- 감정 상태 시각화 UI 추가
- 사용자별 HRV 기준값 기반 개인화 모델 적용
- 긴장 상태 지속 시간 분석 기능 추가
- 저장된 생체 데이터를 활용한 장기 스트레스 패턴 분석
- 무드등, 알림 시스템 등 외부 장치와의 연동

## 프로젝트 목적

본 프로젝트는 생체 신호 기반 웨어러블 감정 분석 시스템 구현을 목표로 합니다. 단순히 심박수를 표시하는 수준을 넘어, HRV 피처와 머신러닝 모델을 활용하여 사용자의 긴장도와 평온도를 실시간으로 추정하는 것을 핵심 목표로 합니다.
