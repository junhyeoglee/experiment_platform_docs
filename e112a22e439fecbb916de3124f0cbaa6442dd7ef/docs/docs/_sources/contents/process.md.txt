# 활용 Process

## 프로세스 정리

모델 개발 -> offline test -> online test -> 실험 결과 평가

![Experiment Cycle](./images/experiment_cycle.png)

---

### 모델 Part

각자 모델 개발하는데, 그 때 효율화 도구
```
TODO
```


### Model Evaluation(Offline test)

오프라인 테스트를 왜 하는지?

어떤 경우에 어떤 지표로 평가할 것인지?
- binary classification
    - 기존의 Train/valid/test 셋으로 구분하여 auc ...
- Recommendation
    - AUC
    - F1 score
    - NDCG
    - IS(Importance Sampling)
    - CIS 등

위 지표가 어느 정도 나와야 Offline test를 통과하고 Online test로 넘어갈 수 있는지?


### Online test

A/B test를 수행함에 있어서 적절한 Sample Size는 어느 정도인지?
그걸 어떤 수식을 통해, 어떤 code로 계산하는지?

실험 등록은 어떻게 하고, 실험이 진행됨에 따라 모니터링은 어떻게 되는지

### 실험 평가

실험이 진행완료 후 생성되는 리포트의 예시 (이미지)

주요 지표에 대한 설명은 어떻게 되는지

