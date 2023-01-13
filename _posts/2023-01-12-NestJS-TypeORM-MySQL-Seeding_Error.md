---
layout: post
title: "[NestJS] TypeORM-MySQL Seeding Error"
categories: NestJS
tags: NestJS TypeORM Error
---

![112730670-de09a480-8f58-11eb-9875-0d9ebb87fbd6](https://user-images.githubusercontent.com/108377235/212341218-a9d01f05-3235-40d9-8b04-c6c5db97bf19.png)

<br/>

## Error

`NestJS`에서 `TypeORM`을 사용해 데이터베이스 서버를 구축하는 작업을 하던 중이었습니다.

TypeORM의 `Seeding` 기능을 사용해 User와 Post 데이터를 넣기 위해서

Seeding 명령어를 입력하니 다음과 같은 에러가 떴습니다.

<br/>

<img width="634" alt="Untitled" src="https://user-images.githubusercontent.com/108377235/212337922-c59d9911-b6d8-4253-a011-cba370bdd928.png">

```bash
Error: Cannot find module 'src/posts/entities/posts.entity'
```

에러 메세지의 `Require stack` 부분을 확인해보니 `ormconfig.ts` 에서

`entity`를 가져올 때 나오는 에러처럼 보였습니다.

<br/>

구글링 후 [stackoverflow](https://stackoverflow.com/questions/66991600/typeorm-migration-error-cannot-find-module)에서 힌트를 얻을 수 있었는데

경로 설정을 변경해주면 해결된다고 해서 한번 수정해보았습니다.

<br/>

### Seeding Code

- 이전
  <img width="532" alt="Untitled 1" src="https://user-images.githubusercontent.com/108377235/212338253-6b5eca2e-6cf1-4600-88b6-5b8c8f7d8a5d.png">

<br/>

- 이후
  <img width="538" alt="Untitled 2" src="https://user-images.githubusercontent.com/108377235/212338348-6f10f70e-04a2-4482-9086-73a05c55e46b.png">

### Entity Code

- 이전
  <img width="521" alt="Untitled 3" src="https://user-images.githubusercontent.com/108377235/212338363-daa57b25-ca2d-4f0a-ba81-4faf6cd3f54b.png">
  <br/>
  <img width="525" alt="Untitled 4" src="https://user-images.githubusercontent.com/108377235/212338423-bc5987be-5b4e-4301-918f-61950cb5ad9d.png">

<br/>

- 이후
  <img width="546" alt="Untitled 5" src="https://user-images.githubusercontent.com/108377235/212338428-da1a5f8c-25dc-42f2-adff-1d8da8dbb8f1.png">
  <br/>
  <img width="535" alt="Untitled 6" src="https://user-images.githubusercontent.com/108377235/212338435-09a22da9-0c96-4b91-ae45-543731b45f4d.png">

<br/>

## Complete

`Entity` 경로를 수정한 후 `Seeding`을 해본 결과 에러가 뜨지 않고 **정상적**으로 Seeding 되는 것을 확인할 수 있었습니다.

<img width="386" alt="Untitled 7" src="https://user-images.githubusercontent.com/108377235/212338436-bfb8c9ae-ae48-4887-a4e7-57c21cc894e2.png">

<br/>

`Database`에도 정상적으로 잘 저장된 것을 확인할 수 있었습니다.

<img width="748" alt="Untitled 8" src="https://user-images.githubusercontent.com/108377235/212338440-e16cf358-cb0e-467e-a0e2-f13fc6f22f3d.png">

<br/>

### P.S.

`TypeORM-Seeding`을 할 때는 다음과 같이 전역 변수를 지정해주어야 합니다.

저는 제 디렉토리 경로에 맞게 경로 수정 후 변수를 지정하였습니다.

<img width="447" alt="Untitled 9" src="https://user-images.githubusercontent.com/108377235/212338443-5467d737-2b29-43eb-9d21-2607723d61ef.png">

<br/>

Seeding과 관련한 보다 자세한 정보는 [Link](https://www.npmjs.com/package/typeorm-seeding)에서 확인하실 수 있습니다.
