# 재활용품 이미지 기반 VQA 모델 개발


### 데이터 셋
4지선다 Visual Question-Answering 객관식 문제 
|Dataset|Number of samples|
|---|---:|
|train|5073|
|test|5074|

### 개발 프로세스
a. Model Selection: task에 특화된 모델 선택하기

b. Fine-tuning: 사전학습된 VQA 모델을 데이터 셋에 맞게 학습하기

c. Prompt Engineering: 추론 성능 향상을 위한 프롬프트 설계하기

d. Inference & Evaluation: 학습된 모델을 활용하여 test 셋에 대한 추론 및 성능 평가하기

---

### Model
- model: Qwen/Qwen2.5-VL-3B-Instruct
- Input: Image + Question + 4 Choices
- Output: Answer

---

### Low-Rank Adaptation (LoRA) 
|hyperparameter|value|description|
|---|---|---|
|r|8|LoRA의 low-rank dimension|
|alpha|16|LoRA adaptation의 scaling factor|
|dropout|0.05|과적합 방지를 위한 dropout|
|target modules|q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj|LoRA를 적용하여 학습할 모듈|

---

### Prompt design
- 영어/한글 표현 통일하기
- Persona를 활용하여 VQA task에 특화된 역할 부여하기
- 모델이 4개의 선택지 중 하나만 출력하도록 output format 제한하기
- 다양한 prompt template을 적용하여 추론 성능 비교하기

---

### Fine-tuning
|hyperparameter|value|description|
|---|---|---|
|learning rate|1e-4|parameter update를 위한 학습률|
|batch size|2|한 번의 iteration에서 사용하는 sample 수|
|epoch|1|전체 training dataset을 학습하는 횟수|
|weight decay|0.01|과적합 방지를 위한 regularization|
|gradient accumulation|4|여러 mini-batch의 gradient를 누적한 후 parameter update|

*구동 환경: kaggle GPU T4*

---

### Inference 
Image + Question + Choices를 모델의 input으로 사용하여 정답 choice를 생성

모델의 생성 결과에서 A/B/C/D 선택지를 추출하여 prediction 수행

---

### Result
Leaderboard score: **0.8569** 

---

### What I Learned
더 많은 데이터셋을 학습할수록 모델 성능이 향상됨
-> 학습 데이터 수만 늘려도 +0.1017 만큼의 성능이 향상됨

GPU memory와 연산 성능을 고려하여 구동환경에 맞는 precison 을 선택하는 것이 중요
-> Kaggle **NVIDIA Tesla T4 GPU** 환경에서 `bfloat16` 대신 `float16`을 사용하여 모델을 구동
-> `batch size`를 **1 → 2**로 증가시켜 학습 효율을 개선
