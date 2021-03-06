# Offline A/B 테스트
### Offline A/B 테스트 진행 이유
Online A/B를 진행할 텐데 굳이 Offline A/B 테스트를 진행하는 이유는 비즈니스 손실을 최소화하기 위함입니다. 실제로 모델을 만들어서 배포를 하기 전까지는 성능이 얼마나 나올지 알 수 없습니다. 이는 비즈니스 손실을 야기할 수 있습니다. 

예를 들어, 기존에 적용하던 A라는 모델이 있었는데 이 모델을 사용하여 현재 10억의 비즈니스 성과를 얻고 있다고 할 때, 한 Data Scientist 가 최근에 발표한 논문을 보다가 B라는 모델이 A라는 모델보다 좋다는 것을 실험적으로 증명한 것을 보고 적용해서 비즈니스 성과를 더 증가시키려 한다고 가정해봅시다. 

문제는 논문에서는 B라는 모델이 A라는 모델보다 더 좋다고 저자가 보유한 데이터를 사용해 실험적으로 증명을 하였지만 실제 산업 현장에 적용하여 더 좋을지 유무에 대해서는 적용해보기 전까지 알 수 없습니다. 논문의 저자가 보유한 데이터 우연히 잘 적용되는 알고리즘이었을 수 도 있고 적용할 때 알고리즘을 그대로 적용이 아니라 일부 수정 혹은 튜닝을 해야하는 상황이어서 수정 혹은 튜닝을 할 경우 예기치 못한 성능 상의 이슈가 발생할 수 있기 때문입니다. 

그렇기 때문에 비즈니스 성과를 창출해야하는 기업의 입장에서는 새로운 알고리즘을 적용하기 이전에 성능이 더 안좋게 나올 수 있는 상황을 고려하게 됩니다. 실제 성능이 더 안좋을 경우 Online A/B 테스트를 진행하였던 시간과 비용의 손실을 초래하게 됩니다. 특히나 SKT의 경우 고객의 전체 수는 갑자기 폭팔적으로 늘어나기엔 어려운 비즈니스 모델을 가지고 있기 때문에 이러한 손실은 비즈니스에 직접적으로 영향을 주게 됩니다. 

이 때 우리가 사전에 정확한 성능은 아니더라도 어느 정도의 성능이 나올 수 있는지 가늠을 해볼 수 있다면, 적어도 손실을 크게 볼 수 있는 상황은 방지할 수 있을 것 입니다. Offline A/B 테스트는 이러한 비즈니스 니즈하에서 제안된 방법이며 정확한 미래 성능은 알 수 없어도 안좋을 경우를 필터링 하는 용도로써는 적극적으로 활용해도 좋을 도구 일 것입니다. 

Offline A/B 테스트를 위한 여러 접근 방법이 있지만 본 패키지의 구현 방법은 추천 시스템에서의 Offline A/B 테스트를 위하여 2018년에 발표된 [1]_ "Offline a/b testing for recommender systems" 논문에 기반하여 구현되었습니다. 


### 추천 시스템 Offline A/B 테스트 
추천 시스템 Offline A/B 테스트를 설명드리기 앞서, 몇가지 사전 가정이 필요합니다.

- [2]_isolation assumption

    - randomly assigned

    n명의 유저 x가 무작위로 적용 중인 알고리즘과 테스트 대상 알고리즘에 할당이 되어 있어야 합니다. 의도적으로 편향이 생길 수 있는 할당이 있다면 신뢰할 수 없는 결과를 도출해내게 됩니다.

    - independent
    
    n명의 유저 x가 무작위로 적용 중인 알고리즘과 테스트 대상 알고리즘에 할당이 되어 있을때 효과는 유저들 간에 독립적으로 작용해야 합니다. 또한 유저가 적용 중인 알고리즘과 테스트 대상 알고리즘에 독립적으로 존재해야 합니다.
    
- stochastic algorithm

    - 확률에 기반한 추천 모델이어야 적용이 가능합니다. 예를 들어 룰베이스로 보내는 "a 어플을 사용한 사람은 A 부가서비스를 가입할 것이다"라는 가정하에 보내는 것들은 Offline A/B 테스트에 적용하기 어렵습니다.
    
이러한 가정하에, 우리가 Offline A/B 테스트를 통해 얻고자 하는 값은 테스트 대상 알고리즘을 적용해서 얻는 결과의 보상(클릭 or 정의된 0보다 큰 실수 값)의 평균 기대 값과 기존 적용 알고리즘을 적용해서 얻는 결과의 보상의 평균 기대값의 차이를 추정하는 것입니다. 결국 테스트 대상 알고리즘을 적용해서 얻는 결과의 기대 값과 기존 적용 알고리즘을 적용해서 얻는 결과의 보상의 기대 값의 차이를 추정하는 것이 우리가 원하는 값이고 이는 아래의 식으로 정리할 수 있습니다.

![expected reward1](./images/1.png)

πp는 강화학습에서 이야기하는 현재 적용 중 인 정책을 의미하며 πt는 테스트하고자 하는 정책을 의미합니다. 즉, πp는 현재 적용 중인 알고리즘으로 어떻게 행동(추천)을 선택(알고리즘으로 추정된 확률 등)하는 방법이며 πt는 테스트하고자 하는 알고리즘으로 어떻게 행동(추천)을 선택(알고리즘으로 추정된 확률 등)하는 방법을 의미합니다. 즉, R은 ATE(average treatment effect)를 의미하며 개입으로 인한 평균 처치 효과를 의미하며 이는 각 정책하에서 행동에 따른 보상의 집단 별 평균의 차이로 정의될 수 있습니다.

![expected reward2](./images/2.png)

이를 직접적으로 계산할 수 없으므로 우리는 이를 추정해야 합니다. 기존 알고리즘을 적용할 대상 그룹을 Pp라 정의하고 이 그룹에 속한 유저 x 가 행동 a를 했을때 보상 r의 집합을 Sp라 할때 Sp = {(xi,ai,ri) : i ∈ Pp}와 같이 정의할 수 있고 테스트 알고리즘을 적용할 대상 그룹을 Pt라 정의하고 이 그룹에 속한 유저 x 가 행동 a를 했을때 보상 r의 집합을 Sp라 할때 St = {(xi,ai,ri) : i ∈ Pt}와 같이 정의할 수 있습니다. 이 Sp와 St의 보상들의 기대 값들을 추정해서 이 들의 차이를 통해 ∆R(πp , πt)를 추정할 수 있습니다. 

다른 말로 하면 고객(유저 x)에게 기존 알고리즘을 적용(행동 a)해서 얻은 보상(클릭 or 구매 등 사전 정의된 값)을 통해 얻은 데이터들(Sp)을 사용해서 보상 값(πp)을 구할 수 있고 이를 사용하여 E(πp)를 추정할 수 있습니다. 그렇다면 결국 우리에게 남은 것은 고객(유저 x)에게 테스트 알고리즘을 적용(행동 a)해서 얻을 보상(클릭 or 구매 등 사전 정의된 값)으로 구성된 데이터들(St)을 사용해 E(πt)를 구하면 해결이 됩니다. 즉 우리는 아래의 식을 해결하는 문제로 귀결 되게 됩니다.

![expected reward3](./images/3.png)

Rˆ(S)는 Online A/B 테스트 동안 수집된 집합 S의 실험적 평균 보상으로 추정하게 됩니다. 추정 방법은 여러가지 접근 방법과 구체적인 해결 방법이 있지만 현재까지 Battleground 에 구현된 알고리즘은 3가지로 Basic Importance Sampling(BIS or IS), Capped importance sampling(CIS), Normalised Capped importance sampling(NCIS) 입니다. 각 알고리즘에 대한 자세한 설명은 세부 페이지를 참조하실 수 있으며 보다 자세한 증명은 [1]을 참조하시면 보실 수 있습니다.

#### Basic Importance Sampling

통계 및 머신 러닝 문제에서 많은 경우는 어떠한 확률 분포 기댓값 (expected value)을 구하는 것과 밀접한 연관이 있습니다. Importance sampling은 효율적으로 기댓값을 추정하기 위해 고안되었으며, 확률 밀도 추정 및 강화 학습 등의 다양한 활용에 이용되고 있습니다. Importance sampling의 목적은 기댓값을 계산하고자 하는 확률 분포 p(x)의 확률 밀도 함수 (probability density function, PDF)를 알고는 있지만 p에서 샘플을 생성하기가 어려울 때, 비교적 샘플을 생성하기가 쉬운 q(x)에서 샘플을 생성하여 p의 기댓값을 계산하는 것입니다. 

![proof Importance Sampling](./images/importance_sampling.png)

이에 기반하여 기존 알고리즘을 활용한 정책에 따라 수집된 보상 이력을 통해 테스트 알고리즘의 기대되는 보상을 계산하는 주요 방법은 Hammersley and Handscomb[3] 가 소개한 importance sampling 혹은 inverse propensity score 라고 불리는 방법입니다. 그 뒤로도 많은 알고리즘이 소개 되었으며 향후 언급되는 이 알고리즘은 IS 혹은 BIS 라고 사용될 예정입니다.

![BIS](./images/BIS.png)

사용 예시는 아래와 같으며 deVDI에서 수행하시면 보실 수 있습니다.

```
>>> #데이터 불러오기 
>>> from skt.gcp import bq_to_pandas #데이터를 불러오기 위한 함수

>>> query = """
>>> SELECT DISTINCT *
>>>   FROM x1112019.ab_model_a_click_prods_tbl
>>>  WHERE impress_yn = 1
>>> """
>>> bucket_a = bq_to_pandas(query) #기존 알고리즘의 추천 결과

>>> query = """
>>> SELECT DISTINCT *
>>>   FROM x1112019.ab_model_b_click_prods_tbl
>>>  WHERE impress_yn = 1
>>> """
>>> bucket_b = bq_to_pandas(query) #새로운 알고리즘의 추천 예시
```

```
>>> #Offline A/B Test Importance Sampling 사용 예시
>>> offline_test = OffLineTest(base = bucket_a[['ci', 'prod_id', 'score', 'reward']]
>>>                            , target = bucket_b[['ci', 'prod_id', 'score']]
>>>                            , item_col = 'prod_id'
>>>                            , user_col = 'ci'
>>>                            , reward = 'reward')
>>> print("\n기존 반응 : ", (bucket_a['reward'] == 1).sum() / bucket_a.shape[0])
>>> is_weight = offline_test.IS() #Importance Sampling

Warning!! You should becareful using this fuctions. Too many rewards are zero.

기존 반응 :  0.01483019427554501

IS avg :  2.5835871313642425e+52
IS method estimated average response rate. : 2583587131364242606636433587958967074568836528100868096.00%
As a result of estimating with IS method, it can be seen that the expected response rate of the new algorithm is superior to that of the existing algorithm.
The difference between the estimated average expected response rate and the existing response rate.(New algorithm - Existed algorithm) :  2.5835871313642425e+52
```

실 사용 예시는 https://github.com/sktaiflow/dag-advanced_analytics/blob/develop/experiment_platform/model_code_templates/make_offline_test_dataset.ipynb 를 참조하시면 보실 수 있습니다.

#### Capped importance sampling

IS 방법은 치명적인 오류가 있는데 이는 importance sampling 의 분산이 원래 분산보다 매우 커질 수 있다는 점 입니다. 확률밀도함수 p(x)에 기반한 함수 f(x)의 분산과 importance sampling 의 분산은 아래와 같이 계산이 됩니다. 

![f variance](./images/p_var.png)
![is variance](./images/is_var.png)

이를 보면 importance sampling의 분산이 p(x) / q(x) 에 따라서 원래 분산보다 매우 커질 수 있음을 알 수 있습니다. 예를 들어 기존에 적용하던 알고리즘의 A 상품에 대한 추천 확률이 0.1 일때 테스트 알고리즘의 A 상품에 대한 추천 확률이 0.9일 경우 importance sampling weight 값은 9가 됩니다. 이럴 경우 importance sampling weight 평균 값이 어느 한 개의 값에 크게 좌우되는 경향을 보이게 될 수 있습니다(이상치 발생 가능성). 

이를 해결하기 위하여 값의 범위를 지정하는 Capping 방법을 활용한 Capped importance sampling 방법이 제안됩니다. IS 방법의 치명적인 오류에 따라 값의 weight 에 제약을 가하는 방식을 CIS(capped importance sampling)이라 합니다. 구체적인 수식은 아래와 같습니다.

![CIS](./images/CIS.png)

c는 weight에 제약을 가하는 상수 값이라고 생각하시면 됩니다. 예를 들어 maxCIS 수식에서 보면 IS 를 통해 산출된 weight가 c 보다 큰 경우 c 값을 산출하게 하여 IS에서 발생 할 수 있는 이상치를 보정해주게 됩니다. 

사용 예시는 아래와 같으며 마찬가지로 deVDI에서 수행하시면 보실 수 있습니다.

이 방법들을 사용하기 위해서는 사전에 알아야하는 몇몇 값들이 있습니다. 
- p : 기존 알고리즘의 전환율입니다. 전환율의 정의는 클릭 혹은 가입 등 다양하게 정의될 수 있으며 예시에서는 클릭으로 진행될 예정입니다. 
- alpha : 1종 오류의 크기입니다. 1종 오류는 간단하게 말하면 맞는데 틀리다고 하는 경우를 의미합니다. 
- power : 2종 오류도 알아야합니다. 2종오류는 틀린데 맞다고 할 경우입니다. 
- expected : 새로운 알고리즘의 기대되는 전환율입니다. 예시에서는 Offline A/B 테스트의 결과를 입력하였습니다. 
- bucket_min_size : 기존 알고리즘의 버킷당 1일 최소 유입 유저 수 입니다. 이는 최소 몇일 간 테스트를 진행할 지 모니터링 기간을 계산하기 위함이며 계산된 샘플크기로부터 최소 유저수를 나누어 계산하며 1주일보다 작은 경우 1주일을 반환하게 됩니다. 
- mde_method : MDE 방법입니다. MDE 값은 Minimum detectable effect, 최소한의 검출 가능 효과를 의미하며 두 알고리즘의 전환율의 차이를 어느 정도로 볼 지 결정하는 값입니다. 현재는 두 알고리즘 각각의 전환율의 차이, 배수, Cohen's h가 구현되어있으며 기본값은 Cohen's h를 따라 계산하는 방법입니다. 그 외에도 사용자가 직접 추정한 두 알고리즘 간의 전환율의 차이를 직접 입력하여 계산할 수 있을 수 있도록 개발되었습니다. 
- case : 동일 분산의 가정 유무입니다. 각 사전에 정의된 값을 따라 계산되어 각 버킷 당 최소 샘플 수를 반환하게 됩니다. 기본 값은 동일 분산이 아니라는 가정하에 계산됩니다.

```
>>> #Offline A/B Test Capped Importance Sampling 사용 예시
>>> offline_test = OffLineTest(base = bucket_a[['ci', 'prod_id', 'score', 'reward']]
>>>                            , target = bucket_b[['ci', 'prod_id', 'score']]
>>>                            , item_col = 'prod_id'
>>>                            , user_col = 'ci'
>>>                            , reward = 'reward')
>>> print("\n기존 반응 : ", (bucket_a['reward'] == 1).sum() / bucket_a.shape[0])
>>> cis_weight = offline_test.CIS() #Capped Importance Sampling
Warning!! You should becareful using this fuctions. Too many rewards are zero.

기존 반응 :  0.01483019427554501

IS avg :  2.5835871313642425e+52
IS method estimated average response rate. : 2583587131364242606636433587958967074568836528100868096.00%
As a result of estimating with IS method, it can be seen that the expected response rate of the new algorithm is superior to that of the existing algorithm.
The difference between the estimated average expected response rate and the existing response rate.(New algorithm - Existed algorithm) :  2.5835871313642425e+52

CIS avg :  0.06893937460876202
CIS method estimated average response rate. : 6.89%
As a result of estimating with CIS method, it can be seen that the expected response rate of the new algorithm is superior to that of the existing algorithm.
The difference between the estimated average expected response rate and the existing response rate.(New algorithm - Existed algorithm) :  0.054109180333217016
```

IS 방법과의 비교를 위해 IS weight 계산 결과도 함께 출력을 해줍니다. 

#### Normalised Capped importance sampling

CIS와 같은 이상치 보정방식은 bias와 variance 에 Trade off 를 가지고 있습니다. 값을 크게 하면 bias는 감소하지만 variance가 증가하며 값을 작게하면 variance는 작아지지만 bias는 크게 됩니다. 이를 보정해주기 위한 여러 방법이 있지만 이 챕터에서는 Normalize 방식을 설명할 예정입니다. 이는 아래의 수식을 따라서 전개됩니다.

![NCIS](./images/NCIS.png)

IS 에서 보이던 특정 local 에서의 bias 발산과 bias 와 variance의 Trade off 문제를 극복하기 위한 일반적인 방법은 NCIS(Normalised Capped importance sampling) 방식입니다. 이에 대한 증명과 설명은 참조된 [1] 논문과 논문의 appendix를 참조하시면 자세히 보실 수 있습니다. 

이 외에도 논문에서 제안한 pieceNCIS(Piecewise constant model) 이나 pointNCIS(Pointwise model)와 같은 방법을 제안하고 있습니다. 간략히 말하면 pieceNCIS는 데이터를 적절한 그룹별로 나누어서 Normalization 하는 방법이며 pointNCIS는 데이터 각각을 Normalization 하는 방법입니다. 이 방법들을 사용하기 위해서는 추천 알고리즘(stochastic algorithm)에서 Context 별 item 추천 확률을 n번 산출하여 수행해야하지만 우리의 현실 여건 상 모델을 여러번 수행하는 것이 어려운 경우가 있으며 모델의 안정성을 위해 random state를 고정하는 등 여러번 수행해도 동일한 값이 나오도록 보장을 하는 경우도 더러 있기 때문에 일단 NCIS 까지만 구현하고 향후 필요한 경우 pieceNCIS나 pointNCIS를 만들어 탑재할 예정입니다.

```
>>> #Offline A/B Test Normalised Capped importance sampling 사용 예시
>>> offline_test = OffLineTest(base = bucket_a[['ci', 'prod_id', 'score', 'reward']]
>>>                            , target = bucket_b[['ci', 'prod_id', 'score']]
>>>                            , item_col = 'prod_id'
>>>                            , user_col = 'ci'
>>>                            , reward = 'reward')
>>> print("\n기존 반응 : ", (bucket_a['reward'] == 1).sum() / bucket_a.shape[0])
>>> ncis_weight = offline_test.NCIS() #Normalised Capped Importance Sampling
Warning!! You should becareful using this fuctions. Too many rewards are zero.

기존 반응 :  0.01483019427554501

NCIS avg :  0.0
NCIS method estimated average response rate. : 0.00%
As a result of estimating with NCIS method, the expected response rate of the new algorithm cannot be considered to be superior to that of the existing algorithm.
The difference between the estimated average expected response rate and the existing response rate.(New algorithm - Existed algorithm) :  -0.01483019427554501
```
-------------------
[1] Gilotte, Alexandre, et al. "Offline a/b testing for recommender systems." Proceedings of the Eleventh ACM International Conference on Web Search and Data Mining. 2018. `<https://github.com/sktaiflow/dag-advanced_analytics/blob/develop/experiment_platform/refer/1801.07030-2.pdf>`_

[2] Léon Bottou and Jonas Peters. 2013. Counterfactual reasoning and learning systems: the example of computational advertising. Proceedings of Journal of Machine Learning Research (JMLR).

[3] JM Hammersley and DC Handscomb. 1964. Monte Carlo Methods. Chapter.

[4] https://statistics.stanford.edu/technical-reports/determining-appropriate-sample-size-confidence-limits-proportion

https://doc-guide.readthedocs.io/ko/latest/insert_test.html#hyperlink
