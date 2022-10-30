---
layout: post
categories: Blog
tags: jekyll google analytics
---

## Intro

좋은 기회로 깃허브 블로그를 만들게 되면서 이것저것 설정해보면서 알게된 점을 정리해보려 합니다.

블로그 운영을 하면서 가장 원동력이 되는 것은

방문자 수와 포스팅에 대한 반응을 확인하는 것이라고 생각합니다.

<br/>

첫 번째로 방문자 수를 확인할 수 있는 방법이 이번 포스팅에서 정리하는 구글 애널리틱스(Google Analytics) 입니다.

두 번째로 포스팅에 대한 반응을 확인하는 방법은 댓글을 확인하는 것 입니다.

이 블로그에서는 포스팅 하단에 utterances를 사용한 댓글 기능을 구현하였습니다.

이것에 대해서는 다음 포스팅을 통해 정리하도록 하겠습니다.

<br/>

지금 보고 계시는 이 블로그는 [jekyll-theme-yat](https://github.com/jeffreytse/jekyll-theme-yat) 을 사용해 만들어졌습니다.

그렇기에 이번 구글 애널리틱스 설정법은 위 블로그 테마를 기준으로 설명하도록 하겠습니다.

다른 jekyll 테마들도 비슷하게 구성되어 있으니

구글 애널리틱스 설정에 어려움을 느끼고 계신 분들에게 도움이 됬으면 좋겠습니다.

<br/>

### Sign up Google Analytics

가장 먼저 해야할 일은 구글 애널리틱스에 가입하는 것입니다.

구글 애널리틱스 가입 방법은 다른 블로그에서도 많이 다루고 있기 때문에

이 글에서는 생략하도록 하겠습니다.

<br/>

### Google Analytics 4 VS Universal Analytics

정상적으로 구글 애널리틱스에 가입 후 사이트 등록까지 완성하셨다면

`G-L72W******` 과 같은 측정(추적) ID 값을 확인하실 수 있을 것입니다.

만약, 자신이 가지고 있는 측정 ID의 값이 UA-\*\*\*\*\*\*로 이루어져 있다고 해도 잘못된 것이 아닙니다.

<br/>

그런 다음 자신의 블로그 테마의 코드를 확인해볼 시간입니다.

저 같은 경우에는 \_includes/extentions/google_analytics.html 경로에 위치해 있었습니다.

위 경로에 작성된 코드는 다음과 같았습니다.

```jsx
<script>
  function initGoogleAnalytics() {
    var doNotTrack =
      window.doNotTrack === "1" ||
      navigator.doNotTrack === "1" ||
      navigator.doNotTrack === "yes" ||
      navigator.msDoNotTrack === "1";
    var enableDNT = "{{ site.enableDNT | default: true }}" == "true";

    if (!enableDNT || !doNotTrack) {
      (function (i, s, o, g, r, a, m) {
        i["GoogleAnalyticsObject"] = r;
        (i[r] =
          i[r] ||
          function () {
            (i[r].q = i[r].q || []).push(arguments);
          }),
          (i[r].l = 1 * new Date());
        (a = s.createElement(o)), (m = s.getElementsByTagName(o)[0]);
        a.async = 1;
        a.src = g;
        m.parentNode.insertBefore(a, m);
      })(
        window,
        document,
        "script",
        "https://www.google-analytics.com/analytics.js",
        "ga"
      );

      ga("create", "{ site.google_analytics }", "auto");
      ga("send", "pageview");
    }
  }
  window.addEventListener("load", initGoogleAnalytics);
</script>
```

여기서 눈 여겨보아야 할 부분은 `ga`로 시작하는 부분입니다.

만약 본인의 코드가 `ga`로 시작한다면 유니버셜 애널리틱스를 기준으로 만들어진 코드입니다.

저희가 사용하려는 구글 애널리틱스 4는 `gtag`로 시작해야 합니다.

이 차이점을 몰라서 한참을 해멨습니다..

<br/>

만약 코드가 `gtag`로 이루어져 있다면 \_config.yml 파일에

```jsx
google_analytics: G-L72W******
```

와 같은 코드를 추가해주시면 됩니다.

추가하는 코드의 양식은 블로그 테마에 따라 다르기 때문에

본인의 블로그 테마의 문서를 살펴보시거나 주석을 확인해보신다면 코드 양식을 찾으실 수 있을 겁니다.

<br/>

### Setting Head Tag

만약 위 과정에서 ga로 이루어진 코드의 테마를 사용하시는 분이라면 다음 과정을 따라오시면 됩니다.

구글 애널리틱스에서 제공해주는 Google 태그를 Head에 설정해주는 방법입니다.

구글 애널리틱스의 관리 - 속성 - 데이터 스트림을 클릭해 웹 스트림 세부정보를 확인합니다.

아래로 스크롤 하면 Google 태그의 ‘태그 안내 보기’가 보이실 텐데 그걸 클릭해줍니다.

<br/>

![Untitled](https://i.ibb.co/TkpCtVN/Untitled.png)

'직접 설치' 탭에 들어가보시면 수동으로 Google 태그를 설치할 수 있도록 코드를 제공해주고 있습니다.

저희는 이 코드를 저희 블로그의 head 부분에 추가할 것 입니다.

<br/>

제 블로그 테마의 구조에서 head를 설정하는 부분은 \_includes/head.html 이었습니다.

코드를 잘 살펴보니 custom-head.html을 가지고 오는 부분이 있어서 자세히 보니

헤드 부분을 보기 쉽게 관리하기 위해서 custom-head 로 따로 구분지어둔 모습을 볼 수 있었습니다.

블로그 테마마다 head 부분을 표시하는 파일이 있을 것이므로 거기서 작업하시면 됩니다.

<br/>

저는 custom-head.html에 위 코드를 추가했습니다.

```jsx
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-L72W******"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-L72WRP127J');
</script>
```

그 후 코드를 깃허브에 push 한 후 deploy까지 성공적으로 잘 되었다면 블로그 페이지에 접속해봅니다.

정상적으로 페이지에 접속이 된다면 구글 애널리틱스에서 확인해봅니다.

<br/>

![Untitled](https://i.ibb.co/VVx1xY0/Untitled-1.png)

위 사진처럼 사용자 수가 올라가있다면 성공입니다!

---

<br/>

이렇게 깃허브 블로그에서 방문자를 확인할 수 있도록 구글 애널리틱스를 적용해보았습니다.

블로그 테마마다 설정방법이 다 다르기 때문에 차이점이 있겠지만

`ga` 와 `gtag`의 차이점을 확인해보신다면 어렵지 않게 적용하실 수 있을 것입니다.
