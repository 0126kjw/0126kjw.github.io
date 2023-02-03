---
layout: post
title: "[MongoDB] querySrv ENODATA 에러"
categories: MongoDB
tags: MongoDB
---

![%E1%84%8C%E1%85%A6%E1%84%86%E1%85%A9%E1%86%A8%E1%84%8B%E1%85%B3%E1%86%AF-%E1%84%8B%E1%85%B5%E1%86%B8%E1%84%85%E1%85%A7%E1%86%A8%E1%84%92%E1%85%A2%E1%84%8C%E1%85%AE%E1%84%89%E1%85%A6%E1%84%8B%E1%85%AD_-003](https://user-images.githubusercontent.com/108377235/216546484-e0105380-8be2-49a9-88b6-f996805925cd.png)

<br/>

## Problem

작업 중 `MongoDB Atlas`에 연결하는 부분에서 갑자기 에러가 발생하였습니다.

```bash
[Nest] 46520  - 2023. 02. 03. 오후 3:48:08   ERROR [ExceptionHandler] querySrv ENODATA _mongodb._tcp.simple-board-cluster.mongodb.net
Error: querySrv ENODATA _mongodb._tcp.simple-board-cluster.mongodb.net
    at QueryReqWrap.onresolve [as oncomplete] (node:internal/dns/promises:240:17)
```

<br/>

기존에 잘 작동하던 코드에서 에러가 났기 때문에 코드 문제는 아닐 것이라고 생각했습니다.

에러 코드를 검색해보니 저처럼 갑작스럽게 에러를 만나게 된 사람들이 많았고

문제점을 금방 파악할 수 있었습니다.

<br/>

바로 `스타벅스 공용 와이파이`를 사용하면 이런 에러가 뜬다는 것입니다.

모바일 핫스팟과 같은 방법으로 해결할 수도 있었지만 와이파이는 포기하지 못하기에..

해결 방법을 [stackoverflow](https://stackoverflow.com/questions/54484673/error-querysrv-enodata-mongodb-tcp-blog-cluster-0hb5z-mongodb-net-at-queryreq)에서 찾을 수 있었습니다.

<br/>

## Solve

제가 사용하는 `Mac OS`를 기준으로 설명하도록 하겠습니다.

우측 상단에 위치한 wifi 버튼을 클릭한 다음 `Wi-Fi 설정`을 클릭합니다.

<img width="346" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-03_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3 58 28" src="https://user-images.githubusercontent.com/108377235/216546502-c4dfce98-0918-47fc-a7e9-556e61bf7807.png">

<br/>

사용 중인 wifi의 `세부사항…`을 클릭한 다음, `DNS` 탭을 클릭합니다.

<img width="827" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-03_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4 00 11" src="https://user-images.githubusercontent.com/108377235/216546504-b3e4032a-5cb9-426d-a400-c8c297f90ed2.png">

<br/>

`+` 버튼을 클릭한 다음 아래와 같이 구글의 DNS 주소인 `8.8.8.8`을 입력합니다.

<img width="827" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-03_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4 01 27" src="https://user-images.githubusercontent.com/108377235/216546515-48ca22cd-8347-4f97-8d00-14a6dd97c9cb.png">

<br/>

마지막으로 wifi를 껐다 킨 다음 다시 wifi에 접속하면 됩니다.

<br/>

이제 스타벅스 공용 와이파이를 사용해서 MongoDB Atlas에 접속할 수 있습니다👍
