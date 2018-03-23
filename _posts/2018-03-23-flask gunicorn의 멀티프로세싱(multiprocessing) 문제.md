---
layout: post
title: "flask gunicorn의 멀티프로세싱(multiprocessing) 문제"
description: "gunicorn 멀티프로세싱(multiprocessing)에 Lock 걸기"
date: 2018-03-2
tags: [flask, gunicorn, multiprocessing]
comments: true
share: true
---
이 포스트에선 학교 인트라넷 서비스 개발 중 있었던 일을 소개하려 합니다.
인트라넷 개발팀은 Flask를 이용한 API 서버와 Vue.js를 이용한 프론트 이렇게 구성되어 있으며 저는 API 서버 개발을 담당하고 있습니다.

**문제발생**
인트라넷의 기능 중 하나인 수강신청을 받던 도중 신청 인원이 가끔 최대인원을 1명 넘는 문제가 발생하였습니다.
굉장히 치명적인 문제였기 때문에 발생직후 바로 문제를 찾기위해 팀원이 다같이 모여 회의를 했습니다.
처음에는 제 실수로 인한 if 문 문제이거나 설계오류라고 생각하고 코드를 보았으나 문제가 없었습니다.
```python
if (count_afterschool_apply(afterschool_idx) >= afterschool.max_of_count):
    ns.abort(403)
```
최대신청 인원과 현재 신청 인원을 비교해 더 이상 신청을 못 하게 하는 부분이며, count_afterschool_apply는 모델에서 현재 신청 인원을 받아오는 함수이고 afterschool.max_of_count는 최대신청인원수입니다.

다시 문제를 살펴보던 도중 가장 이상한 점은 최대인원을 넘는 1명이 항상 발생하는 것이 아니라 가끔 발생한다는 것이었습니다. 그리곤 수강신청 특성상 많은 사용자가 한 번에 몰리기 때문에 혹시 병렬처리 방식에서 문제가 발생하는 것이 아니겠냐는 생각을 했습니다.

당시 저희는 gunicorn이라는 WSGI Server를 사용 중이었고 검색결과 아니나 다를까 gunicorn에는 Workers라는 멀티프로세싱이 있었고 gunicorn초기 설정에서 이 Worker를 4개나 열어 준 것이었습니다.
1번 프로세스에서 count_afterschool_apply로 현재 신청인원을 가져오고 신청하는 명령어를 처리하기 전에 2번 프로세스에서 또 count_afterschool_apply로 현재 신청 인원을 가져오기 때문에 최대 인원을 넘기는 인원이 발생한 것입니다.

**해결방법**
문제를 확인하고 이를 해결하는 방법으로 처음 떠오른 것은 'Worker를 1개만 열어주면 되겠다' 였습니다. 그러나 단순히 이 부분 하나를 위해 멀티프로세싱을 포기한다면 많이 느려질 것이라 생각을 하였고 다른 방법을 찾아보았습니다.
그리고 찾은 것이 바로 Lock이었습니다. 우선 해결 방법은
```python
from multiprocessing import Lock

lock = Lock()

lock.acquire()
if (count_afterschool_apply(afterschool_idx) >= afterschool.max_of_count):
	ns.abort(403)
lock.release()
```
이렇게 해당하는 코드에 lock을 걸어주면 되는 것이었습니다. ~~(이 문제를 찾기 위해 고생을 했는데 이렇게 간단하다니...)~~
위 코드 처럼 lock이 acquire 하면 다른 프로세스는 release 될 때까지 그 부분을 간섭하지 못하게 됩니다. 한 마디로 저 lock이 해제될 때까지 다음 프로세스는 기다리는 거죠. 그럼 lock이 걸린 부분은 동시에 실행될 수 없으므로 위와 같은 문제가 해결되는 것입니다.

**후기**
멀티프로세싱(multiprocessing)의 이해에 대한 부족에서 시작된 문제였고 결국 그 부분을 다시 처음부터 보고 알게 되었습니다. 이 문제를 겪고 멀티프로세싱뿐만 아니라 다른 좋은 기능들을 사용하기 위해선 그 기능에 대한 이해도가 높아야 하겠다고 생각이 드는 순간이었습니다.
