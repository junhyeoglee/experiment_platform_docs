
## 패키지 설명

### 구조

```
battleground
│   Readme.md
└───model
│   │   data.py
│   │   train.py
│   │   eval.py
│   │   serve.py
└──experiment
    │   register.py
    │   report.py
    │   ab_test.py
    │   monitoring.py
```

### Example

- 여러 실험들의 전투장

---

사용 예제 sudo code

> model

1. `model.data` 
    - 데이터 전처리, Features 선택, train/valid/test 셋 생성 등의 작업
```python
from battleground.model.data import Preprocess, FeatureSelection, CreateDataSet

dataset = CreateDataSet(feature_list=[], train_dt=[], infer_dt=[])
X_train, y_train = dataset.load_train_set()
X_test, y_test = dataset.load_test_set()

fs = FeatureSelection(method='lightgbm')
X_train_selected = fs.fit_transform(X_train, y_train)
X_test_selected = fs.transform(X_test)

preprocess = Preprocess(numerical_features=[], categorical_features=[], imputation_methods=[])
X_train, y_train = preprocess.fit_transform(X_train, y_train)
X_test, y_test = preprocess.transform(X_text, y_test)
```

2. `model.train` 
    - 모델 학습
    - gbm 계열 모델에 대해 적절한 hyper parameter tuning 등
```python
from battleground.model.train import Trainer

trainer = Trainer(model='lightgbm')
fit_params = {'categorical_features':categorical_features}
trainer.fit(X_train_selected, y_train, **fit_params)

```


3. `model.eval`
    - 모델 성능평가
```python
from battleground.model.eval import EvalRecommenderModel, EvalClassifierModel, EvalRegressorModel

pred = trainer.predict_proba(X_test)
evaluator = EvalRecommenderModel(mode='offline')
evaluator.get_importance_sampling(pred, y_test)
# evaluator.get_capped_importance_sampling(pred, y_test)
# evaluator.get_normalized_cis(pred, y_test)

evaluator = EvalClassifierModel()
evaluator.calculate(pred, y_test)
```

4. `model.serve`
    - 모델 성능평가
```python
from battleground.model.serve import ...
```

---

> experiment

5. `experiment.register`
    - 모델 실험 등록

```python
from battleground.experiment.register import RegisterExperiment

client = RegisterExperiment(client='10.x.x.x') # db
client.add_experiment(meta={"name":"Galileo신규모델", "실험시작일":"2022-03-05"})
client.add_end_condition({"목표samplesize":100}) # 종료조건 입력

```

6. `experiment.report`
    - 실험 종료 이후 리포트 생성

```python
from battleground.experiment.report import MakeReport

report = MakeReport()
cr = report.create_report()
```

7. `experiment.ab_test`
    - ab 테스트에 필요한 정보들 계산
    - ab 테스트 전에 적절한 sample_size 추정
    - ab 테스트 p-value 계산
```python
# input : 
from battleground.experiment.ab_test import ABTest

abtest = ABTest()
abtest.get_t_test()
abtest.get_sample_size()
```

8. `experiment.monotoring`
    - 실험 진행중에 a/b 테스팅에 대한 monitoring
```python
# input : 실험 id, 실험기간 정보
# output : 해당 기간 동안 성과 (CTR 등)
from battleground.experiment.monitoring import Monitor

monitor = Monitor(client='1.1.1.1')
monitor.put_experiment_id(expeirment_id='실험id')
monitor.get_abtest_result(start_dt='2022-03-01', end_dt='2022-03-10')
```
