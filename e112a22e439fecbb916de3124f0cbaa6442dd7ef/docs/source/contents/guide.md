# Sphinx 편집 Guideline

- 설치하기

```
$ conda install Spinx 
$ pip install Sphinx
```

- 최초 build 하기

```
$ sphinx-quickstart docs
$ sphinx-build -b html docs/source/ docs/build/html
```

- RTD theme 적용

```python
html_theme = 'sphinx_rtd_theme'  # in conf.py file
```

- markdown extention 적용
```
$ pip install myst_parser
$ pip install sphinxcontrib-napoleon
```

```python
extensions = ['myst_parser']
source_suffix = {
    '.rst': 'restructuredtext',
    '.txt': 'markdown',
    '.md': 'markdown',
}
```

- 문서 편집할 때
    - 위 패키지 (Spinx, myst_parser) 설치
    - contests 폴더 밑에 문서 작성 후, 
    - ```$ sphinx-build -b html docs/source/ docs/build/html``` 혹은 ```$ make build```
    - 모듈 구성이 변경되었을 때는 
    - ```$ sphinx-apidoc -F -o ./docs/source ./battleground```  수행해서 source밑에 ```.rst``` 파일을 업데이트해주고 build를 재수행