---
layout: post
title: "pipenv, Python 패키지 관리자 사용하기"
description: "pipenv 사용 방법과 후기"
date: 2018-04-27
tags: [python, pipenv]
comments: true
share: true
---

처음 파이썬을 접하면 virtualenv이라는 가상환경 툴을 추천하곤 합니다. 여러 Python 프로젝트를 진행하면 다양한 패키지를 사용하고 각 패키지가 서로 충돌되는 경우가 생길 수 있는데 이를 막기위한 방법이었습니다. 그러나 virtualenv를 사용해보신 분들이라면 알겠지만 패키지를 변경할 때마다 `requirements.txt` 를 수정해주는 것은 여간 귀찮은 것이 아닙니다. 또한 `requirements.txt`와 `requirements-dev.txt`를 나눠서 관리는 것도 상당히 불편합니다.

이런 불편함을 해소할 수 있는pipenv 라는 패키지관리자를 소개시켜드리려고 합니다.

Pipenv : https://github.com/pypa/pipenv

## Pipenv가 특별한 이유

* pip와 virtualenv를 한 번에 쓸 수 있다.

* `Pipfile, Pipfile.lock` 로 모든 의존성 관리

* 깔끔한 인터페이스

  Pipenv 개발자인 [Kenneth Reitz](https://github.com/kennethreitz) 는 [requests](https://github.com/requests/requests)를 만든 제작자이기도 합니다. request도 깔끔한 인터페이스로 많은 Python 사용자에게 사랑받고 있듯 Pipenv도 깔끔한 인터페이스를 가지고 있습니다.

* python.org 에서 공식적으로 추천하는 패키지 관리툴

## 설치하기 & 세팅하기

pip을 이용해 pipenv 설치하기

```bash
$ pip install pipenv
```

프로젝트 폴더로 이동해서 세팅하기

```bash
$ pipenv install
```

이 명령어로 pipenv는 폴더에서 Python 파일을 찾고 virtualenv를 만들어줍니다. 그다음 프로젝트 폴더에 `Pipfile` 과 `Pipfile.lock` 이 생성된 것을 확인할 수 있습니다.

만약 프로젝트의 Python 버전을 따로 지정해주고 싶다면

```bash
$ pipenv --python 3.6
```

버전을 명시해주면 됩니다.

## Pipenv 사용하기

그럼이제 pipenv를 사용해 원하는 패키지를 설치해보겠습니다

```bash
$ pipenv install requests
```

이렇게 패키지를 설치하면 Pipenv는 의존성을 고려하여 적절한 버전을 설치해줍니다. 이를 확인하기 위해 `Pipfile`을 살펴보겠습니다.

```bash
[[source]]
url = "https://pypi.python.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"

[dev-packages]

[requires]
python_version = "3.6"
```

source에는 설치를 어디서 어떻게 할지가 명시되어 있으며 그 밑에 packages는 우리가 방금 설치한 request에 대한 내용이 있습니다. 버전을 따로 명시해주지 않았기 때문에 *로 표시하고 있습니다. 그리고 dev-packages는 개발할때 필요한 패키지를 관리해주는 것으로

```bash
$ pipenv install --dev pytest
```

이렇게 설치 할 수 있습니다.

마지막으로 requires는 프로젝트의 파이썬 버전을 설정해주고 있습니다.

### 의존성 검사

프로젝트를 진행하다 보면 새로운 패키지를 계속해서 추가하게 됩니다. 물론 Pipenv는 의존성을 확인해서 적절한 버전을 설치해주지만 의존성을 버전 업해야할 상황은 언제든지 찾아올 수 있습니다.

```bash
$ pipenv check
```

이 명령어를 통해서 보안 취약점들 문제가 있는 것을 확인하고 

```bash
$ pipenv update
```

를 통해 `Pipfile` 에서 지정한 규칙안에서 `Pipfile.lock` 을 최신의 버전으로 업데이트해줍니다.

Pipenv는 현재 의존성 상태를 보기 좋게 정리해주기도 합니다.

```bash
$ pipenv graph
```

이 명령어는 지금 프로젝트의 모든 패키지 의존성을 글목록으로 정리해서 보여 줍니다.

###  virtualenv 환경 활성화 하기

Pipenv로 설치한 virtualenv를 사용하는 것은 설치만큼이나 간단합니다.

```bash
$ pipenv shell
```

이 명령어 만으로 우리는 virtualenv 화경 속으로 들어와서 작업을 수행할 수 있습니다. 만약 virtualenv 확여 밖에서 안에 있는 코드를 실행하고 싶다면

```bash
$ pipenv run python app.py
```

이런식으로 실행시킬 수 도 있습니다.

## 협업에서 Pipenv

Github와 같은 곳에 Pipfiles를 올리고 다른 사용자가 clone을 했다면

```bash
$ pipenv install
```

명령어 한 번만으로 Pipfile에 명시된 모든 패키지가 설치됩니다. 만약 dev-packages까지 설치하고 싶다면

```bash
$ pipenv install --dev
```

로 dev에 있는 패키지까지 모두 설치가능합니다.

## 사용해본 결과

기존에 사용하던 다른 패키지 관리툴 보다 너무 간단해서 처음 도입때도 어려움 없이 적용할 수 있어서 좋았습니다. 그러나 Pipenv는 계속해서 안정성을 찾아가는 단계이고 확실히 불안정한 부분이 있었습니다. 실제 제가 사용중에 Pipenv가 최신버전으로 업데이트 되면서 Flask 모듈과 충돌이 일어나 Docker환경이 죽는 경우가 생기기도 했습니다. 이렇게 아직은 조금 부족한 부분이 있지만 Pipenv는 점점 많은 사람들이 찾고 있고 그만큼이나 안정성도 빠르게 높아지고 있습니다. 그렇기 때문에 저 포함 많은 Python 사용자들이 프로젝트를 진행할때 Pipenv를 이용하여 관리할 것 같다라는 기대를 하면서 글을 마치겠습니다.