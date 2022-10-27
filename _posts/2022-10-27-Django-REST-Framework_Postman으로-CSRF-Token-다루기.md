---
layout: post
categories: Django
tags: Django DRF Postman
---

본 게시물은 아래의 링크 내용을 참고하였습니다.

[https://medium.com/hackernoon/automatically-set-csrf-token-in-postman-django-tips-c9ec8eb9eb5b](https://medium.com/hackernoon/automatically-set-csrf-token-in-postman-django-tips-c9ec8eb9eb5b)

---

<br/>

Django, DRF(Django REST Framework), dj-rest-auth, django-allauth를 사용해서

로그인과 회원가입을 구현하는 과정에서 처음 겪는 문제에 직면하였습니다.

<br/>

DRF 웹을 사용한 테스트에서는 위와 같이 정상적으로 작동되는 것을 확인할 수 있습니다.

하지만, POSTMAN에서 JSON 형식으로 같은 요청을 보냈을 때

csrf token 오류가 뜨는 것을 확인할 수 있었습니다.

csrf token에 대한 자세한 내용은 아래 링크에서 확인해보실 수 있습니다.

[https://portswigger.net/web-security/csrf/tokens](https://portswigger.net/web-security/csrf/tokens)

<br/>

Django는 사이트 간 요청 위조를 방지하기 위해 안전하지 않은 방법을 통한 요청에 대한 CSRF 보호 메커니즘이 내장되어 있습니다.

만약, AJAX POST 메서드에서 CSRF 보호를 사용하도록 설정한 경우 요청에서 X-CSRF Token 헤더를 전송해야 합니다.

하지만 X-CSRF Token이 존재하지 않거나 올바르지 않은 경우

제가 겪었던 문제가 생기는 것으로 확인할 수 있었습니다.

### POSTMAN에서 CSRF 설정하기

DJango는 로그인 시 csrf token을 쿠키에 저장합니다.

POSTMAN에서 이것을 확인할 수 있습니다.

<br/>

이 토큰 값을 그대로 가지고 와서 헤더 값에 수동으로 설정할 수 있습니다.

<br/>

하지만 이러한 방법은 불편한 점이 있습니다.

만약, 토큰이 만료된다면 또 다시 위와 같은 과정을 거쳐 수동으로 변경해야 한다는 단점이 있습니다.

이 방법 대신 포스트맨 스크립팅 기술을 사용하여 쿠키에서 토큰을 추출하고 이를 환경변수로 설정하는 방법이 있습니다.

### 자동으로 CSRF 토큰 설정하기

<br/>

POSTMAN에서 Add request 후 Tests에 들어간 후, 아래 코드를 입력합니다.

```javascript
var xsrfCookie = postman.getResponseCookie("csrftoken");
postman.setGlobalVariable("csrftoken", xsrfCookie.value);
```

그러면 csrf 토큰이 추출되어 현재 환경에서 csrftoken이라는 환경 변수로 설정됩니다.

이제 request를 작성할 때 이 변수를 사용하여 아래와 같이 헤더를 설정할 수 있습니다.

<br/>

이제는 토큰이 만료되더라도 다시 로그인을 하면 csrf 토큰이 자동으로 업데이트됩니다.

---

이렇게 Postman에서 csrf 토큰을 자동으로 설정하고 갱신하는 방법에 대해 알아보았습니다.

프로젝트를 진행하다보면 이렇게 생각지도 못한 곳에서 막힐 때가 종종 있습니다.

이러한 문제를 해결하는 과정에서 제가 몰랐던 사실을 알게 되고 그렇게 학습이 되면서

자연스레 제 역량이 늘어가는 것만 같은 기분이 듭니다.

앞으로도 새롭게 알게 되는 정보나 중요하게 다뤄야 할 내용은

꾸준히 업로드 하도록 하겠습니다.

<br/>
<br/>
<br/>

본 게시물은 아래의 링크 내용을 참고하였습니다.

[https://medium.com/hackernoon/automatically-set-csrf-token-in-postman-django-tips-c9ec8eb9eb5b](https://medium.com/hackernoon/automatically-set-csrf-token-in-postman-django-tips-c9ec8eb9eb5b)
