---
layout: post
categories: Blog
tags: jekyll comment utterances
---

## Intro

깃허브 블로그에 google analytics를 설정하는 것에 이어서

이번에는 게시물에 댓글 기능을 추가해보도록 하겠습니다.

게시물마다 달린 댓글의 반응을 보는 것이

블로그 운영의 소소한 행복이라고 생각합니다.

<br/>

제가 지금 사용하고 있는 블로그의 테마에서는

Disqus, Gitment, utterances 총 3가지를 지원하고 있었습니다.

저는 이 중에서 광고가 없고 디자인이 깔끔한 utterances를 선택하였습니다.

이 외에도 utterances는 다음과 같은 장점들이 있습니다.

- 오픈 소스입니다.
- 추적X, 광고X, 항상 무료입니다.
- 다크 테마를 지원합니다.
- Github에서 지원하는 CSS 툴킷인 Primer로 이루어져 있습니다.
- 가볍습니다.

<br/>

### utterances

기능을 추가하기에 앞서 우선적으로 해주어야 할 설정이 있습니다.

바로 repository 연결입니다.

<br/>

utterances는 깃허브 레포지토리에 연동되어 작동합니다.

누군가가 어느 게시물에 댓글을 작성하게 된다면 그 댓글이 해당 게시물의 이슈에 저장되고 그것을 불러오는 방식으로 작동합니다.

만약 처음으로 작성된 댓글일 시 utterances는 자동으로 이슈를 생성해줍니다.

그렇기에 우선적으로 댓글을 저장할 레포지토리를 설정해야 합니다.

<br/>

다른 블로그들을 보니 깃허브 블로그 레포지토리를 그대로 사용하시는 분들도 계셨고,

댓글을 저장하는 레포지토리를 새로 생성 후 사용하시는 분들도 계셨습니다.

저는 깃허브 블로그 레포지토리를 그대로 이용해보겠습니다.

<br/>

[https://github.com/apps/utterances](https://github.com/apps/utterances)

위 링크에 접속하게 되면 GitHub App인 utterances를 설정할 수 있습니다.

설정을 하실 때에는 본인이 원하는 레포지토리에 App을 설치해주시면 됩니다.

저 같은 경우에는 깃허브 블로그 레포지토리를 지정해주었습니다.

<br/>

정상적으로 설정이 완료되셨으면 해당 레포지토리의 설정에 들어간 후

Integrations - GitHub apps 설정을 확인해보시면 설치된 GitHub Apps에

아래 사진과 같이 utterances가 추가되어있는 모습을 확인하실 수 있습니다.

![utterances](https://user-images.githubusercontent.com/108377235/199655800-68c39da1-6b25-401b-8167-f8a1934c3c0e.png)

<br/>

### \_config.yml

레포지토리에 정상적으로 utterances가 설치되었으면 이제 \_config.yml을 설정해주어야 할 차례입니다.

그 이전에 utterances를 사용하는 파일의 구조를 먼저 살펴본 후 진행해보겠습니다.

<br/>

현재 제가 사용하고 있는 테마에서 utterances에 관한 코드는

\_includes/extensions/comments/utterances.html에 있었습니다.

해당 코드는 다음과 같습니다.

```html
{% raw %} {%- if site.utterances.follow_site_theme -%}
<div id="utterances-placeholder"></div>
<script>
  const utterancesThemeFromDataTheme = () => {
    const dataTheme = document.documentElement.getAttribute("data-theme");
    return `github-${dataTheme}`;
  };

  const setUtterancesTheme = () => {
    const iframe = document.querySelector(".utterances-frame");
    if (iframe) {
      const message = {
        type: "set-theme",
        theme: utterancesThemeFromDataTheme(),
      };
      iframe.contentWindow.postMessage(message, "https://utteranc.es");
    }
  };

  // dynamic change
  const observer = new MutationObserver((mutationsList, observer) => {
    for (let mutation of mutationsList) {
      if (
        mutation.type === "attributes" &&
        mutation.attributeName === "data-theme"
      ) {
        setUtterancesTheme();
      }
    }
  });
  observer.observe(document.documentElement, {
    attributes: true,
    childList: false,
    subtree: false,
  });

  let utterancesScript = document.createElement("script");
  utterancesScript.async = true;
  utterancesScript.src = "https://utteranc.es/client.js";
  utterancesScript.crossOrigin = "anonymous";
  utterancesScript.setAttribute(
    "issue-term",
    "{{ site.utterances.issue_term }}"
  );
  utterancesScript.setAttribute("label", "{{ site.utterances.label }}");
  utterancesScript.setAttribute("repo", "{{ site.utterances.repo }}");
  utterancesScript.setAttribute("theme", utterancesThemeFromDataTheme());

  const placeholder = document.getElementById("utterances-placeholder");
  placeholder.parentNode.replaceChild(utterancesScript, placeholder);
</script>
{%- else -%}
<script
  async
  crossorigin="anonymous"
  issue-term="{{ site.utterances.issue_term }}"
  label="{{ site.utterances.label }}"
  repo="{{ site.utterances.repo }}"
  src="https://utteranc.es/client.js"
  theme="{{ site.utterances.theme }}"
></script>
{%- endif -%} {% endraw %}
```

<br/>

또한 게시물의 하단에 댓글을 작성할 수 있도록 \_layouts/post.html 파일의 하단에 다음과 같이 코드를 추가해둔 모습을 확인할 수 있었습니다.

```html
{% raw %}
<div class="post-comments">
  {%- if page.comments != false -%} {%- if site.disqus.shortname -%} {%- include
  extensions/comments/disqus.html -%} {%- endif -%} {%- if site.gitment.username
  -%} {%- include extensions/comments/gitment.html -%}\ {%- endif -%} {%- if
  site.utterances.repo -%} {%- include extensions/comments/utterances.html -%}
  {%- endif -%} {%- endif -%}
</div>
{% endraw %}
```

<br/>

이처럼 기능을 추가할 때 단순히 \_config.yml에 코드만 바꾸는 것이 아니라

내 블로그의 구조가 어떻게 이루어져있고 어떤 방식으로 작동하는지 확인하는 습관을 기르게 된다면 많은 도움이 될 것입니다.

<br/>

이제 구조도 한번 살펴보았으니 \_config.yml에서 설정을 해줄 차례입니다.

앞서 utterances를 설정해주는 코드를 보았을 때 \_config.yml에서 정보를 가져오는 코드를 확인할 수 있었습니다.

저는 \_config.yml에 약 190번대 줄에 작성된 코드가 utterances를 설정해주는 코드였습니다.

이것은 블로그 테마마다 전부 상이할 것이니 확인해주시고

만약 미리 기본적으로 설정해준 코드가 없다면 현재 사용하고 계신 블로그 테마의 문서를 찾아보시면 힌트를 얻으실 수 있을 것입니다.

<br/>

저는 코드를 다음과 같이 수정하였습니다.

```yml
utterances:
  repo: "0126kjw/0126kjw.github.io" # 레포지토리의 경로
  issue_term: "title" # 이슈 매핑
  label: "utterances comment" # 이슈 라벨
  theme: "github-light" # 댓글 창 테마
  follow_site_theme: true # 사이트의 테마에 따를 것인지
```

<br/>

추가적인 설정과 설명이 필요하시다면 아래 링크에서 확인해보시고 코드를 원하는 대로 수정하시면 됩니다.

[https://utteranc.es/](https://utteranc.es/)

---

<br/>

이렇게 utterances 설정이 끝났습니다.

정상적으로 잘 설정되었다면 이 게시물의 하단에 있는 댓글 창과 같은 모습을 확인하실 수 있을 것입니다.

다음에는 깃허브 블로그를 포털 사이트에서 검색할 수 있도록 하는 방법에 대해 작성하도록 하겠습니다.
