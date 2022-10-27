---
layout: post
categories: Django
tags: Django DRF
---

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

---

<br/>

어느새 DRF의 두 번째 튜토리얼입니다.

이번 튜토리얼의 주제는 Requests and responses(요청과 응답)입니다.

이 시점부터 REST Framework의 핵심 기능을 다루기 시작할 것입니다.

거두절미하고 빠르게 들어가보겠습니다!

<br/>

### Request objects(요청 객체)

REST Framework는 일반 HTTP Request를 확장하고 보다 유연한 요청 구문 분석을 제공하는 Request 객체를 제공합니다.

이 Request 객체의 핵심 기능은 request.data 속성으로 request.POST와 유사합니다.

그러나, Web API 작업에 더 유용한 속성입니다.

```python
request.POST  # form data만 처리합니다.  'POST' 메서드에서만 작동합니다.
request.data  # 임의의 데이터를 처리합니다.  'POST', 'PUT' 및 'PATCH' 메서드에서 작동합니다.
```

<br/>

### Response objects(응답 객체)

REST Framework는 또한 렌더링되지 않은 content를 가져오고 content negotiation을 사용하여

클라이언트에 return할 올바른 content type을 판별하는

TemplateResponse 유형인 Response objects를 제공합니다.

```python
return Response(data)  # 클라이언트가 요청한 content type으로 렌더링합니다.
```

<br/>

### Status codes(상태 코드)

숫자 HTTP 상태 코드를 사용한다고 해서 항상 명확하게 읽을 수 있는 것은 아니며 오류 코드가 잘못 되었을 경우 쉽게 알아차릴 수 없습니다.

REST Framework는 상태 모듈에서 HTTP_400_BAD_REQUEST와 같이 각 상태 코드에 대해 더욱 명시적인 식별자를 제공하고 있습니다.

숫자 식별자를 사용하지 않고 이를 사용하시는 것을 추천합니다.

<br/>

### Wrapping API views

REST Framework는 API views를 작성하는 데 사용할 수 있는 두 개의 wrapper를 제공합니다.

1\. 함수 기반 views 작업을 위한 @api_view 장식자

2\. 클래스 기반 views 작업을 위한 APIView 클래스

이러한 wrapper는 view에서 Request 인스턴스를 수신하는지 확인하고, content negotiation을 수행할 수 있도록 Response 객체에 context를 추가하는 등의 몇 가지 기능을 제공합니다.

또한 wrapper는 적절한 경우 405 Method Not Allowed 응답을 반환하고, 잘못된 형식의 이름으로 request.data에 액세스할 때 발생하는 ParseError 예외를 처리하는 등의 동작을 진행합니다.

<br/>

### Pulling it all together

이제 이러한 새로운 구성 요소를 사용하여 view를 약간 리팩토링해 보겠습니다.

```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

저희의 인스턴스 view는 이전의 예에 비해서 개선되었습니다.

조금 더 간결해졌고, 이제 코드는 Forms API로 작업했을 때와 매우 비슷하게 느껴집니다.

또한 명명된 상태 코드를 사용하여 응답의 의미를 더욱 명확하게 이해할 수 있습니다.

다음은 views\.py 모듈의 개별 snippet view 입니다.

```python
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

이 코드가 매우 친숙하게 느껴지실 것입니다.

이는 일반적으로 Django views를 사용하는 것과 크게 다르지 않기 때문입니다.

더 이상 요청 또는 응답을 지정된 콘텐츠 유형에 명시적으로 연결하지 않습니다.

request.data는 들어오는 json 요청을 처리할 수 있지만 다른 형식도 처리할 수 있습니다.

또한 마찬가지로 응답 객체를 데이터로 반환하지만 REST Framework가 응답을 올바른 콘텐츠 유형으로 렌더링할 수 있도록 해줍니다.

<br/>

### Adding optional format suffixes to our URLs(URL에 선택적 형식 접미사 추가)

응답이 더 이상 단일 콘텐츠 유형에 고정되어 있지 않다는 사실을 활용하기 위해 API endpoints에 형식 접미사에 대한 지원을 추가하겠습니다.

형식 접미사를 사용하면 지덩된 형식을 명시적으로 참조하는 URL이 제공되며 API에서 [http://example.com/api/items/4.json](http://example.com/api/items/4.json과) 과 같은 URL을 처리 할 수 있습니다.

두 가지 views 모두에 형식 키워드 인수를 추가해봅니다.

```python
def snippet_list(request, format=None):
```

그리고

```python
def snippet_detail(request, pk, format=None):
```

이제 snippets/urls.py 파일을 업데이트 하여 기존 URL에 format_syslog_sets를 추가합니다.

```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

이러한 추가 URL 패턴을 반드시 추가할 필요는 없지만 특정 형식을 간단하고 깔끔하게 참조할 수 있습니다.

<br/>

### How's it looking?

첫 번째 튜토리얼에서와 같이 command line에서 API를 테스트 해봅시다.

잘못된 요청을 보내면 오류 처리가 더 잘되지만 모든 것이 거의 비슷하게 작동하는 것을 볼 수 있습니다.

또한, 전과 마찬가지로 모든 snippet의 목록을 얻을 수 있습니다.

```shell
$ http http://127.0.0.1:8000/snippets/

HTTP/1.1 200 OK
...
[
  {
    "id": 1,
    "title": "",
    "code": "foo = \"bar\"\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  },
  {
    "id": 2,
    "title": "",
    "code": "print(\"hello, world\")\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  }
]
```

Accept 헤더를 사용하여 반환되는 응답 형식을 제어할 수 있습니다.

```shell
$ http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
$ http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
```

또한 형식 접미사를 추가하여 다음과 같이 제어할 수 있습니다.

```shell
$ http http://127.0.0.1:8000/snippets.json  # JSON suffix
$ http http://127.0.0.1:8000/snippets.api   # Browsable API suffix
```

마찬가지로 Content-Type 헤더를 사용하여 보내는 요청 형식을 제어할 수 있습니다.

```shell
# POST using form data
$ http --form POST http://127.0.0.1:8000/snippets/ code="print(123)"

{
  "id": 3,
  "title": "",
  "code": "print(123)",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}

# POST using JSON
$ http --json POST http://127.0.0.1:8000/snippets/ code="print(456)"

{
    "id": 4,
    "title": "",
    "code": "print(456)",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```

만약 위의 HTTP 요청에 --debug를 추가하면 요청 헤더에서 요청 유형을 볼 수 있습니다.

이제 http://127.0.0.1:8000/snippets/ 를 방문하여 웹 브라우저에서 API를 열어 봅시다.

<br/>

### Browsability(검색 가능성)

API는 클라이언트 요청을 기반으로 응답의 콘텐츠 유형을 선택하기 때문에 기본적으로 웹 브라우저에서 해당 리소스를 요청할 때 HTML 형식의 리소스 표현을 반환합니다.

이를 통해 API는 완전히 웹 브라우징이 가능한 HTML 표현을 반환할 수 있습니다.

웹 브라우징 API가 있다는 것은 사용성의 측면에서 큰 이점이며 API를 훨씬 쉽게 개발하고 사용할 수 있습니다.

또한 API를 검사하고 작업하려는 다른 개발자의 진입 장벽을 크게 낮춰줍니다.

탐색 가능한 API 기능 및 사용자 지정 방법에 대한 자세한 내용은 아래 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/topics/browsable-api/](https://www.django-rest-framework.org/topics/browsable-api/)

---

<br/>

여기까지 두 번째 튜토리얼인 요청과 응답에 대해서 알아보았습니다.

지금까지의 내용을 보면 확실히 DRF는 개발자들을 편하게 작업할 수 있도록 도와주는 Framework라는게 느껴집니다.

앞으로 꾸준히 공식 문서를 보면서 DRF 만의 장점을 함께 배워갈 수 있었으면 좋겠습니다.

다음 튜토리얼은 클래스 기반 view를 사용해보고 우리가 작성해야하는 코드의 양을 줄이는 방법에 대해 알아보겠습니다.

<br/>
<br/>
<br/>

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
