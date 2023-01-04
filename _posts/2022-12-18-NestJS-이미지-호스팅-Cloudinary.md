---
layout: post
title: "[NestJS] 이미지 호스팅 - Cloudinary"
categories: NestJS
tags: NestJS Cloudinary
---

![cloudinary_logo_for_white_bg](https://user-images.githubusercontent.com/108377235/210595537-b691c8a9-535b-43ff-a5c6-68edc1fb0b88.png)

<br/>

## Intro

부트캠프에서 진행했던 팀 프로젝트가 끝이 났습니다.

5주라는 기간동안 다양하고 많은 시행착오를 겪으면서

한층 더 성장할 수 있는 계기가 되었던 것 같습니다.

<br/>

이번 게시물은 NestJS를 사용한 팀프로젝트를 진행하면서

이미지 호스팅 서비스인 Cloudinary를 사용했던 방법에 대해 간단하게 작성해보도록 하겠습니다.

<br/>

## Cloudinary?

Cloudinary는 **클라우드 기반 이미지 및 비디오 관리를 제공하는 서비스**입니다.

이를 통해 사용자는 간편하게 웹사이트와 앱의 이미지와 비디오를 업로드, 저장, 관리, 조작 및 전달할 수 있습니다.

<br/>

이번 프로젝트에서 Cloudinary를 통해 이미지를 관리하게 된 이유는

박물관과 전시관의 전경 및 내부 사진을 한 장소 당 2장씩 보여줘야하는 상황이었는데,

장소가 총 127개이기 때문에 이미지의 개수는 대략 254개 이상이 되었습니다.

<br/>

이미지를 VM Local에 저장해서 불러오는 방식을 사용할 수 있었지만

이미지마다 해상도가 전부 제각각이며 그로 인해 크기도 천차만별이라

VM의 용량을 **무의미하게 많이 차지**하는 문제가 생겼습니다.

<br/>

이때 백엔드를 담당해주셨던 코치님에게 이러한 문제에 대해 질문을 드리니

Cloudinary와 같은 이미지 호스팅 서비스를 활용해보는 것이 어떤지에 대해 답변을 해주셔서

적극적으로 새로운 서비스를 활용해보기로 하였습니다.

<br/>

## Quick Start

[Cloudniary 사이트](https://cloudinary.com)에 접속한 후 우측 상단에 있는 `SIGN UP FOR FREE` 버튼을 누르면

회원가입을 하는 페이지로 넘어가게 됩니다.

<br/>

![Untitled](https://user-images.githubusercontent.com/108377235/210596340-3a229902-f06e-4368-9fa0-56c97b00c75e.png)

저는 Github 계정으로 회원가입을 하였습니다.

회원가입을 완료하고 간단한 설정을 끝마치게 된다면 다음과 같은 화면을 보실 수 있으실 겁니다.

<br/>

![Untitled 1](https://user-images.githubusercontent.com/108377235/210596283-dcd3ea1e-f136-401a-b2f7-943bce17fa89.png)

이 페이지에서 Cloudinary API를 사용하는 Key와 env 등을 확인할 수 있으며

최근 30일동안 어느 정도 사용했는지 확인할 수 있습니다.

<br/>

회원가입을 하게 되면 자동적으로 Free Plan으로 시작되며

기본적으로 25 크레딧을 제공합니다.

이는 25GB의 용량을 무료로 사용할 수 있는 것을 의미합니다.

<br/>

좌측 네비게이션에서 Media Library를 누르면 이미지 및 비디오를 관리할 수 있는 페이지로 넘어갈 수 있습니다.

<br/>

![Untitled 2](https://user-images.githubusercontent.com/108377235/210596304-6951534b-ec28-4f14-8f85-311f03928f5a.png)

이곳에서 이미지 및 비디오를 업로드하고 관리할 수 있습니다.

<br/>

![Untitled 3](https://user-images.githubusercontent.com/108377235/210596327-2382d521-287c-495f-8a3c-810f1326ced1.png)

이미지 우측 상단에 있는 <>를 클릭하면 이미지 주소를 복사할 수 있습니다.

저는 이번 프로젝트를 진행할 때 이미지 주소와 이미지 이름을 활용해서 사용했습니다.

<br/>

## How?

`https://res.cloudinary.com/example/image/upload/v1670411853/01_image01.jpg`

위 URL은 이미지 주소를 복사하였을 때 볼 수 있는 모습입니다.

`https://res.cloudinary.com/example/image/upload/v1670411853/`

이 때, 위 부분은 다른 이미지 주소에서도 동일하게 적용되는 것을 확인할 수 있었습니다.

<br/>

저는 이미지 이름을 `01_image01.jpg` 와 같이 설정하기로 하였습니다.

앞 `01` 은 ID 값을 의미하고 `image01`, `image02` 와 같이 첫 번째, 두 번째 사진을 표시할 수 있도록 설정하였습니다.

예로 들자면 `93_image02`는 ID 값이 93인 장소의 두 번째 이미지를 의미하는 파일명임을 알 수 있습니다.

<br/>

이번 프로젝트에서 위와 같은 방법을 사용해

이미지를 **효율적**으로 관리하고 프론트단에서 **간편하게** 이미지를 불러올 수 있도록 하였습니다.

<br/>

아래 사진은 실제 프로젝트에서 적용된 모습입니다.

![Untitled 4](https://user-images.githubusercontent.com/108377235/210597248-965b34db-02af-4e16-a56c-3e1f5265e657.png)

<br/>

## Next..

이번 포스팅에서는 간단하게 Cloudinary를 사용해 이미지를 관리하고 불러오는 방법에 대해 다뤄봤습니다.

세상에는 다양한 서비스가 있지만 그 서비스들을 어떻게 적재적소에 잘 사용할 수 있을지에 대한 고민을 하게 되었던 것 같습니다.

이러한 고민을 꾸준히 하는 것이 저 스스로를 성장시킨다고 생각합니다.

<br/>

다음에는 Cloudinary API를 사용하여 이미지를 업로드하고 목록을 받아오는 것에 대해 작성해보도록 하겠습니다.
