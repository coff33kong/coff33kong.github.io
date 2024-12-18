---
published: true
layout: single
title:  "대형 언어 모델(LLMs)은 사용자의 검색 선호도를 얼마나 정확히 예측할 수 있을까?"
header:
  overlay_image: /images/unsplash-image-2.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
  actions:
    - label: "Learn more"
      url: "https://dl.acm.org/doi/10.1145/3626772.3657707"
      
categories: [SearchModeling]
tags: [LLM, SearchModeling]
comments: true
---

안녕하세요! 오늘은 **대형 언어 모델(LLMs)**이 **검색 엔진에서의 검색 결과 평가**에 어떻게 활용될 수 있는지를 다룬 연구를 소개합니다. **Microsoft** 연구팀이 발표한 논문 **"Large Language Models Can Accurately Predict Searcher Preferences”**를 바탕으로, **LLMs이 사람처럼, 혹은 그 이상으로 검색 의도를 이해하고 관련성(relevance)을 평가할 수 있는지**에 대해 자세히 살펴보겠습니다.

---

## **1. 검색 시스템과 관련성 레이블의 중요성**

### **1-1. 관련성 레이블이란?**

검색 엔진은 사용자가 입력한 **검색어(query)**에 대해 적절한 **문서(document)**를 제공해야 합니다. 이를 위해 각 문서가 검색 의도에 얼마나 부합하는지를 판단하는 **관련성 레이블(relevance label)**이 필요합니다.

### **1-2. 관련성 레이블의 역할**

- **검색 결과 평가:** 검색 엔진이 제공한 결과가 사용자에게 얼마나 **유용한지**를 측정
- **머신러닝 모델 학습:** 검색 시스템을 개선하기 위한 **학습 데이터**로 사용

**관련성 레이블의 평가 기준**은 일반적으로 다음과 같습니다:

- **높은 관련성(2점):** 문서가 검색어에 대해 매우 유용하고 정확한 정보를 제공할 때
- **보통 관련성(1점):** 문서가 검색어와 어느 정도 관련이 있지만, 불필요한 정보가 포함될 때
- **관련 없음(0점):** 문서가 검색어와 거의 관련이 없을 때

하지만 **사람이 직접 레이블을 작성하는 방식**은 다음과 같은 문제점이 있습니다:

- **시간과 비용이 많이 소요됨**
- **일관성 부족:** 평가자마다 **기준이 다를 수 있음**
- **편향:** 평가자가 사용자 의도를 잘못 이해하거나 **개인적인 선입견**을 반영할 수 있음

---

## **2. 연구 목표와 질문**

이 연구의 목표는 **대형 언어 모델이 사람을 대체해 검색 결과의 관련성을 평가할 수 있는지**를 확인하는 것입니다. 이를 위해 세 가지 주요 질문에 답을 찾고자 했습니다:

1. **LLMs은 사람과 비교해 얼마나 정확하게 문서의 관련성을 평가할 수 있는가?**
2. **프롬프트(prompt) 구성에 따라 LLM의 성능은 어떻게 변하는가?**
3. **LLMs이 실제 검색 환경에서도 신뢰할 수 있는 결과를 제공할 수 있는가?**

---

## **3. 실험 설정**

### **3-1. 데이터셋: TREC-Robust 2004**

- **TREC-Robust 2004 데이터셋**은 250개의 **검색 주제(topic)**로 구성되어 있으며, 각 주제는 하나의 **검색어(query)**와 관련된 문서들로 이루어져 있습니다.
- 각 문서는 이미 **전문 평가자(human assessors)**에 의해 **정답 레이블(gold label)**이 부여되어 있습니다.
- 문서의 레이블은 **0(관련 없음)**, **1(보통 관련성)**, **2(높은 관련성)**의 세 가지로 분류되었습니다.

### **3-2. 모델: GPT-4**

- 실험에는 **GPT-4**가 사용되었습니다.
- **온프레미스 환경**에서 실행되었으며, 모델의 출력은 **0에서 2까지의 정수값**으로 변환되었습니다.
- **출력 형식:** 각 문서에 대해 **JSON 형식**으로 결과를 반환했습니다.

---

## **4. 프롬프트 설계 및 변형**

LLM이 문서의 관련성을 평가할 때, **프롬프트의 구성**이 결과에 중요한 영향을 미칩니다. 연구진은 다양한 프롬프트 변형을 실험했습니다.

### **4-1. 프롬프트 구성 요소**

프롬프트는 크게 네 부분으로 구성되었습니다:

1. **역할(Role):** 모델이 **검색 품질 평가자**로 행동하도록 설정
`role : 당신은 웹 페이지의 품질을 평가하는 검색 품질 평가자입니다.`
2. **검색어 및 문서:** 사용자가 입력한 **검색어**와 해당 문서 제공
3. **세부 지침:** 문서의 **토픽 적합성**과 **신뢰성**을 별도로 평가하도록 요청
4. **출력 형식:** 결과를 **JSON 형식**으로 반환하도록 요구

---

### **4-2. 프롬프트 변형**

연구진은 다음과 같이 다양한 프롬프트 변형을 통해 **프롬프트의 세부 요소**가 성능에 미치는 영향을 분석했습니다.

| 프롬프트 요소 | 설명 |
| --- | --- |
| **역할 (Role)** | 모델에게 검색 품질 평가자로 행동하도록 요청 |
| **설명 (Description)** | 검색어에 대한 추가 설명 제공 |
| **서술 (Narrative)** | 문서의 배경 정보 추가 |
| **세부 평가 (Aspects)** | 문서를 **토픽 적합성**과 **신뢰성**으로 분리하여 평가 |
| **다중 심사자 (Multiple Judges)** | 모델이 여러 평가자를 시뮬레이션하여 각기 다른 시점에서 평가 |

---

## **5. 실험 결과**

### **5-1. 프롬프트 구성에 따른 성능 차이**

프롬프트 구성 요소에 따라 **Cohen's Kappa(κ)** 점수가 크게 달라졌습니다.

| 프롬프트 구성 | Cohen's Kappa(κ) |
| --- | --- |
| 기본 프롬프트 | 0.34 |
| 역할+설명+서술 | 0.61 |
| 설명+서술+세부 평가 | **0.64** |
| 다중 심사자 포함 | 0.51 |

### **5-2. 인간 평가자와의 비교**

- **사람 평가자 평균:** Cohen’s κ = 0.58
- **GPT-4 최고 성능:** Cohen’s κ = **0.64**

LLM은 **일부 프롬프트 설정에서 사람보다 더 높은 일치도**를 보였습니다.

---

## **6. Bing 검색 엔진에의 적용**

### **6-1. LLM을 활용한 검색 품질 평가**

이 연구는 **Bing 검색 엔진**에 실제로 적용되었습니다.

- **평가 속도:** 기존에는 평가에 수 시간이 걸렸던 작업이 **몇 분 내에 완료**되었습니다.
- **비용 절감:** 기존 크라우드 워커에 비해 **20배 이상의 비용 절감**이 이루어졌습니다.
- **정확도:** LLM은 **사람 평가자보다도 일관된 결과**를 제공했습니다.

### **6-2. 품질 관리**

Bing은 **LLM이 생성한 레이블을 매주 샘플링**하여 **사람 평가자가 다시 검토**하고, 모델의 성능 변화를 지속적으로 모니터링했습니다.

---

## **7. 결론 및 미래 과제**

### **7-1. 연구의 시사점**

이 연구는 **대형 언어 모델이 검색 엔진의 관련성 평가에 강력한 도구**로 사용될 수 있음을 보여줍니다.

### **7-2. 미래 과제**

1. **프롬프트 최적화:** 다양한 프롬프트 구성과 변형을 지속적으로 탐구할 필요가 있습니다.
2. **편향 문제 해결:** LLM의 편향을 줄이기 위해 더 다양한 데이터셋을 활용해야 합니다.
3. **환경적 비용:** LLM 운영에 필요한 에너지 소비를 줄이는 방법을 모색해야 합니다.

---

## **마무리**

이 글이 **검색 시스템의 작동 원리**와 **LLM의 활용 가능성**에 대한 이해를 높이는 데 도움이 되었길 바랍니다! 😊 더 궁금한 점이 있다면 댓글로 남겨주세요.

**참고문헌**

이 연구는 **SIGIR 2024**에서 발표되었으며, 자세한 내용은 [여기](https://doi.org/10.1145/3626772.3657707)에서 확인할 수 있습니다.