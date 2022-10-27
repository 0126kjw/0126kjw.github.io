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

벌써 세 번째 튜토리얼입니다.

이번 튜토리얼의 주제는 Class-based views(클래스 기반 뷰) 입니다.

지금까지의 튜토리얼에서 views를 작성할 때 function-based views(함수 기반 뷰)로 작성을 하였습니다.

이번에는 클래스 기반으로 views를 작성하는 방법에 대해 알아보겠습니다.

클래스 기반으로 views를 작성하게 되면 코드 재사용성이 높아지고 코드를 DRY한 상태로 유지하는 데 도움을 준다고

공식 문서 상에서는 이야기하고 있습니다.

문서를 천천히 따라가보면서 학습해보도록 하겠습니다.

<br/>

### Rewriting our API using class-based views(클래스 기반 뷰를 사용하여 API 재작성)

우선 root view를 클래스 기반 뷰로 다시 작성해보겠습니다.

views.py 파일을 아래와 같은 코드로 재 작성합니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

이전 예제와 비슷하게 보일 수 있지만 다른 HTTP 메서드를 더욱 잘 구분할 수 있게 되었습니다.

그리고 views.py의 인스턴스 뷰를 수정합니다.

```python
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

지금은 함수 기반 뷰와 비슷하게 보일 수 있습니다.

하지만 매우 좋아졌습니다.

클래스 기반 뷰를 사용할 것이기 때문에 snippets/urls.py 파일도 아래와 같이 수정합시다.

```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

이렇게 지금까지 작성한 코드를 클래스 기반 방식으로 변경하였습니다.

이전과 같이 개발 서버를 시작해보면 이전과 마찬가지로 작동하는 것을 확인하실 수 있습니다.

<br/>

### Using mixins(믹스인 사용)

클래스 기반 뷰를 사용하는 가장 큰 이점 중 하나는 재사용이 가능한 동작 비트를 구성할 수 있다는 점입니다.

지금까지 생성해온 Create/Retrieve/Update/Delete 작업은 우리가 만드는 모든 model-backed API view와 매우 유사할 것입니다.

이러한 일반적은 동작을 하는 비트는 REST Framework의 mixin 클래스에서 구현됩니다.

mixin 클래스를 사용하여 뷰를 구성하는 방법을 살펴 보겠습니다.

views.py 파일을 확인해보겠습니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

위 코드를 자세히 보겠습니다.

GenericAPIView를 사용하여 뷰를 빌드하고 ListModelMixin과 CreateModelMixin을 추가하였습니다.

기본 클래스는 핵심 기능을 제공하고 mixin 클래스는 .list() 및 .create() 작업을 제공합니다.

그런 다음 get 및 post 메서드를 적절한 작업에 바인딩합니다.

```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

GenericAPIView 클래스를 사용하여 핵심 기능을 제공하고 mixin을 추가하여 .retrieve(), .update() 및 .destroy() 작업을 제공합니다.

<br/>

### Using generic class-based views(일반 클래스 기반 뷰 사용)

mixin 클래스를 사용하여 이전보다 약간 적은 코드를 사용하도록 뷰를 다시 작성했지만 한 단계 더 나아갈 수 있습니다.

REST 프레임워크는 우리의 views.py 모듈을 훨씬 더 다듬는 데 사용할 수 있는 이미 mixed-in된 일반 view 세트를 제공합니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

정말 간결하게 표현이 가능합니다.

이 코드들은 훌륭하고 깨끗하며 관용적으로 사용하는 장고와 같이 보입니다.

---

<br/>

이번 튜토리얼을 통해 클래스 기반 뷰를 작성하는 방법을 알아보았습니다.

함수 기반 뷰는 구현이 간단하고 읽기가 편하며 코드가 직관적이라는 장점을 가지고 있습니다.

하지만 코드를 확장하거나 재사용하기 어렵고 조건문으로 HTTP 메소드 구분한다는 단점이 있습니다.

클래스 기반 뷰는 코드를 확장하거나 재사용하기 쉽고, mixin(다중 상속) 같은 객체지향 기술을 사용할 수 있다는 장점이 있습니다.

또한 위 예제와 같이 분리된 메소드로 HTTP 메소드 구분할 수 있습니다.

하지만 한 눈에 읽기 힘들다는 것과 코드가 직관적이지 않고 숨어있다는 단점이 있습니다.

둘 중 무엇이 더 좋고 나쁜지는 따질 수 없지만 상황에 맞게 사용하면 될 것 같습니다.

다음 튜토리얼으로는 API에 대한 인증 및 권한을 처리하는 방법을 알아보겠습니다.

<br/>
<br/>
<br/>

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
