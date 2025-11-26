# 🎙️ LightGrad Diffusion TTS – Korean KSS Dataset 실험 프로젝트

이 프로젝트는 **LightGrad Diffusion 기반 TTS 모델**을 한국어(KSS 데이터셋)로 직접 학습하고,
텍스트 전처리 방식(영문화 / g2pk / 자모 분해)에 따른 성능 차이를 비교하기 위한 실험 프로젝트입니다.

Diffusion 기반 TTS를 처음부터 끝까지 직접 구축·학습해보며  
한국어 TTS 적용 시 필요한 전처리, 모델 구조, 학습 설정 등을 실험적으로 확인하는 것이 목표입니다.

---

# 📌 1. 전체 목표

- KSS 단일 화자 데이터로 **LightGrad Diffusion TTS 학습**
- 입력 텍스트 전처리 **3종 비교 실험**
  - 단순 영문화
  - g2pk phoneme 변환
  - 자모 분해(jamo)
- Griffin-Lim vocoder를 사용하여 빠르게 audio 결과 확인

> "Diffusion 기반 TTS를 처음부터 끝까지 직접 학습해보고,
> 한국어 TTS 적용 시 어떤 전처리와 구조가 필요한지 이해하기 위한 실험 프로젝트."

---

# 📁 2. 데이터셋

### ✔ KSS 한국어 단일 화자 음성 데이터셋
- 약 12시간 분량
- 4,000 문장
- wav + 스크립트(metadata.csv) 제공
- 기본 샘플 레이트 22050Hz

---

# 🧱 3. LightGrad 모델 구성 개요

LightGrad는 **Diffusion Probabilistic Model** 기반으로 Mel-Spectrogram을 생성하는 TTS 모델입니다.

일반적인 Diffusion TTS보다 훨씬 적은 연산량으로 동작하며,  
한국어처럼 음절 구조·음운 변화가 많은 언어에서도 안정적인 합성 결과를 보이는 것이 특징입니다.

---

## 🔷 주요 구성 요소

### **1) Encoder**
- 텍스트(jamo/g2pk 등) 임베딩 → Convolution → Linear layer  
- 한국어 자모 기반 입력에도 적합한 경량 구조  
- 출력값은 Duration Predictor와 Diffusion Decoder의 conditioning으로 사용

---

### **2) Duration Predictor (MAS Alignment)**
- MAS(Monotonic Alignment Search) 기반  
- Attention collapse 없음  
- phoneme → mel frame 길이(duration) 직접 예측  
- 긴 문장에서도 안정적  
- 한국어 음절 길이 변화(받침, 장단음 등)에 강함

---

### **3) Lightweight U-Net Diffusion Decoder**

#### U-Net 구성
- Encoder–Decoder + skip connections  
- 시간/주파수 정보를 동시에 복원  
- Depthwise Separable Convolution 적용 (연산량 감소)  
- Linear Attention으로 긴 길이 처리 가능

#### Diffusion 작동 방식
- **Training**  
  - clean mel → 점진적 noise 추가  
  - U-Net이 noise 제거 방향 학습
- **Inference**  
  - Gaussian noise에서 시작  
  - 4-step DPM-Solver로 fast denoising  
  - 최종 mel 생성

---

### **4) Vocoder (Griffin-Lim 사용)**  
- mel → waveform 변환  
- 실험 단계에서 빠르고 가벼움  
- 추후 HiFi-GAN 교체 가능

---

## 🔥 경량화 및 효율성

- **Depthwise Conv** → 파라미터 수 대폭 감소  
- **Linear Attention** → O(N) 메모리  
- **4-step inference** → 매우 빠른 추론  
- **Streaming inference 지원**

짧은 step 수만으로 자연스러운 mel 생성이 가능하며  
긴 문장에서도 noise 누적 없이 안정적입니다.

---

# 🔤 4. 입력 텍스트 전처리 실험

한국어는 음절 문자이며 음운 변동이 많기 때문에 텍스트 표현 방식에 따라 TTS 성능 차이가 커집니다.

본 프로젝트에서는 아래 3가지 방식을 비교했습니다.

---

### **① 단순 영문화**
- ex. "가나다라" → "ganadara"
- 구조 단순
- 발음 정보 손실 → baseline용

---

### **② g2pk 기반 phoneme 변환**
- 한국어 발음을 영문 phoneme으로 변환
- 영어 기반 구조와 잘 맞음
- LightGrad와의 호환성이 가장 좋았음

---

### **③ 자모 분해(jamo)**
- 초성/중성/종성 분리
- 한국어 발음 구조를 가장 정확하게 반영
- 토큰 수 증가 → 학습 난이도 증가  
- prosody·리듬 표현에 강점

---

# 🏋️‍♂️ 5. 학습 과정

---
### **마무리**

본 프로젝트는 Diffusion 기반 TTS 모델을
한국어 데이터로 직접 학습하며 전처리·구조·학습 구성 요소의 효과를 확인하는
연구 목적의 실험 프로젝트입니다.

추후 확장 가능 작업:
HiFi-GAN vocoder 적용
자모 기반 학습 고도화
Tacotron2 / FastSpeech2 비교 실험