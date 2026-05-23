# HRV 기반 실시간 감정 상태 분석 시스템

ESP32 Feather와 EmotiBit FeatherWing을 이용하여 사용자의 생체 신호를 수집하고, HRV(Heart Rate Variability) 기반 피처를 계산하여 사용자의 평온/긴장 상태를 실시간으로 분석하는 임베디드 프로젝트입니다.

측정된 심박 및 IBI 데이터를 기반으로 SDNN, RMSSD, LF, HF, Total Power 등의 HRV 피처를 계산하고, 사전에 학습된 MLP 모델 파라미터를 이용해 `calm`과 `tense` 값을 추론합니다. 추론 결과는 BLE Nordic UART Service를 통해 안드로이드 앱 또는 외부 수신 장치로 전송할 수 있습니다.

## 프로젝트 개요

이 프로젝트는 웨어러블 생체신호 기반 감정 분석 시스템을 목표로 합니다.  
EmotiBit 센서를 통해 수집한 심박 관련 데이터를 ESP32에서 실시간으로 처리하고, HRV 분석 결과를 기반으로 사용자의 긴장도와 평온도를 확률 형태로 제공합니다.

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
