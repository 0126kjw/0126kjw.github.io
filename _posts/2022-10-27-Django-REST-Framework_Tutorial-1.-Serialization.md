---
layout: post
title: "[Django-REST-Framework] 1.Serialization"
categories: Django
tags: Django DRF
---

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

---

<br/>

이제부터 본격적인 Django REST Framework의 튜토리얼 시작입니다.

첫 번째 챕터는 Serialization 입니다.

Serialization 즉, 직렬화는 컴퓨터 과학의 데이터 스토리지 문맥에서 데이터 구조나 오브젝트 상태를 동일하거나 다른 컴퓨터 환경에 저장하고 나중에 재구성할 수 있는 포맷으로 변환하는 과정이라고 합니다. (출처: 위키백과)

간단하게 메모리를 디스크에 저장하거나 네트워크 통신에 사용하기 위한 형식으로 변환하는 것을 의미한다고 이해하시면 좋습니다.

공식 문서에서 소개하는 말로 이 튜토리얼은 상당히 심도있는 내용이기 때문에 시작하기 전 쿠키와 좋아하는 술 한잔을 준비해야 한다고 적혀 있습니다.

우스갯소리로 소개를 했지만 튜토리얼의 첫 번째 챕터가 Serialization인 만큼 굉장히 중요하고 심도깊게 다루는 요소라고 이해할 수 있습니다.

그러면 본격적으로 공식 문서의 내용을 따라가면서 학습해보도록 하겠습니다.
<br/>

## Setting up a new environment

다른 작업을 수행하기 전에 venv를 사용하여 새로운 가상 환경을 만들도록 이야기하고 있습니다.

이 방법을 사용한다면 패키지의 구성이 작업 중인 다른 프로젝트와 잘 분리되어 유지할 수 있다고 합니다.

이 방법은 콘솔창에서 다음과 같은 코드를 사용해 진행할 수 있습니다.

```shell
$ python3 -m venv env
$ source env/bin/activate
```

이 과정을 수행하면 현재 가상 환경에 있는 것 입니다.

다음으로 패키지에서 필요로 하는 것들을 설치해줍니다.

```shell
$ pip3 install django
$ pip3 install djangorestframework
$ pip3 install pygments  # 코드 강조를 위해 이것을 사용한다고 합니다.
```

가상 환경을 종료하기 위해서는 deactivate를 사용하면 됩니다.

자세한 내용을 원하시면 아래 링크에서 확인하실 수 있습니다.

[https://docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html)

<br/>

### Getting started

이제 코딩할 준비가 다 되었으니 작업한 새로운 프로젝트를 만들어 보겠습니다.

```shell
$ cd ~
$ django-admin startproject tutorial
$ cd tutorial
```

완료되면 간단한 Web API를 만드는 데 사용할 앱을 만들 수 있습니다.

```shell
$ python3 manage.py startapp snippets
```

이렇게 만들어진 snippets 앱과 rest_framework 앱을 INSTALLED_APPS에 추가해야 합니다.

그러기 위해선 tutorial/settings.py 파일의 INSTALLED_APPS 부분을 아래와 같이 수정합니다.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'snippets',
]
```

<br/>

### Creating a model to work with

이 튜토리얼의 목적을 위해서 코드 스니펫을 저장하는 데 사용되는 간단한 Snippet 모델을 만들어 보겠습니다.

snippets/model.py 파일을 아래와 같이 작성합니다.

```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ['created']
```

참고로 공식 문서에서 이야기하는 바로는 좋은 프로그래밍 방법에는 주석이 포함되지만 코드 자체에 집중하기 위해서 이 예제에서는 주석을 생략했다고 하니 참고해주시면 감사하겠습니다.

<br/>

다음으로 snippet 모델에 대한 초기 migration을 생성하고 처음으로 데이터베이스를 동기화합니다.

```
$ python3 manage.py makemigrations snippets
$ python3 manage.py migrate snippets
```

migration(마이그레이션)에 대한 자세한 내용은 아래 링크에서 확인하실 수 있습니다.

[https://docs.djangoproject.com/en/4.1/topics/migrations/](https://docs.djangoproject.com/en/4.1/topics/migrations/)

<br/>

### Creating a Serializer class

Web API에서 시작해야 할 첫 번째 사항은 snippet 인스턴스를 json과 같은 표현으로 직렬화하고 역직렬화하는 방법을 제공하는 것입니다.

Django의 양식과 매우 유사하게 작동하는 Serializer를 선언하여 이를 수행할 수 있습니다.

snippets 라는 경로에 serializers.py 라는 파일을 생성 후 다음과 같은 코드를 추가합니다.

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

serializer 클래스의 첫 번째 부분에는 직렬화/역직렬화되는 필드를 정의합니다.

create() 및 update() 메서드는 serializer.save()를 호출할 때 완전히 분리된 인스턴스를 만들거나 수정하는 방법을 정의합니다.

serializer 클래스는 Django Form 클래스와 매우 유사하며

required, max_length 및 default와 같은 다양한 필드에 유사한 유효성 검사 플래그를 포함하고 있습니다.

필드 플래그는 HTML로 렌더링할 때와 같은 특정한 상황에서 serializer를 표시하는 방법도 제어할 수 있습니다.

그리고 {'base_widget': 'textarea.widget'} 플래그는 Django의 Form 클래스에서 widget = widget을 사용하는 것과 동일하게 동작합니다.

이 기능은 나중에 진행될 튜토리얼에서 확인할 수 있듯이 탐색 가능한 API를 표시하는 방법을 제어하는 데 특히 유용하다고 합니다.

나중에 배우게 될 ModelSerializer 클래스를 사용하면 시간을 절약할 수 있지만, 지금은 serializer 정의를 명시적으로 유지하며 진행하도록 하겠습니다.

<br/>

### Working with Serializers

더 나아가기 전에 새로운 Serializer 클래스를 사용하는 방법에 대해 알아보도록 하겠습니다.

Django의 shell로 진입해보겠습니다.

```shell
$ python3 manage.py shell
```

이제 몇 가지 가져오기를 완료하고 나면 몇 가지의 코드 스니펫을 만들어 보도록 하겠습니다.

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print("hello, world")\n')
snippet.save()
```

이제 몇 가지의 스니펫 인스턴스를 사용할 수 있습니다.

이러한 인스턴스 중 하나를 직렬화하는 방법을 알아보겠습니다.

```python
serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': '', 'code': 'print("hello, world")\n', 'linenos': False, 'language': 'python', 'style': 'friendly'}
```

이 시점에서 모델 인스턴스를 Python native datatypes로 변환했습니다.

serialization 프로세스를 완료하기 위해서 데이터를 json으로 렌더링합니다.

```python
content = JSONRenderer().render(serializer.data)
content
# b'{"id": 2, "title": "", "code": "print(\\"hello, world\\")\\n", "linenos": false, "language": "python", "style": "friendly"}'
```

역직렬화도 비슷합니다.

먼저 스트림을 Python native datatypes로 구분 분석합니다.

```python
import io

stream = io.BytesIO(content)
data = JSONParser().parse(stream)
```

그런 다음 이러한 native datatypes를 완전히 채워진 개체 인스턴스로 복원합니다.

```python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print("hello, world")\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object
```

API가 Form 작업과 얼마나 유사한지 확인할 수 있습니다.

serializer를 사용하는 뷰를 작성하기 시작하면 유사성이 훨씬 더 분명해질 것입니다.

모델 인스턴스 대신 querysets를 직렬화할 수도 있습니다.

그렇게 하려면 간단하게 many = True 플래그를 serialzer 인수에 추가하면 됩니다.

```python
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', ''), ('code', 'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', ''), ('code', 'print("hello, world")\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', ''), ('code', 'print("hello, world")'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

<br/>

### Using ModelSerializers

SnippetSerializer 클래스는 Snippet 모델에 포함된 많은 정보를 복제하고 있습니다.

Django가 Form 클래스와 ModelForm 클래스를 모두 제공하는 것과 마찬가지로 REST Framework는 Serializer 클래스와 ModelSerializer 클래스를 모두 포함하고 있습니다.

ModelSerializer 클래스를 사용하여 serializer를 리팩터링하는 방법을 알아보겠습니다.

snippets/serializers.py을 다시 열고 SnippetSerializer 클래스를 다음과 같이 수정합니다.

```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']
```

serializer의 좋은 특성 중 하나는 serializer 인스턴스의 모든 필드를 print 하여 검사할 수 있다는 것입니다.

python manage.py shell을 사용해 Django shell을 열고 아래 코드를 작성하십시오.

```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```

ModelSerializer 클래스는 특별히 마법같은 일을 하지 않는다는 것을 기억하는 것이 중요합니다.

단순히 직렬화된 클래스를 생성하기 위한 지름길입니다.

<br/>

### Writing regular Django views using our Serializer

새로운 Serializer 클래스를 사용하여 몇 가지 API views를 작성하는 방법을 살펴보겠습니다.

당분간 REST Framework의 다른 기능을 사용하지 않고, Django의 일반적인 views로 작성하겠습니다.

snippets/views.py 파일을 수정해야 하는데 다음 코드를 추가합니다.

```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

이 API 경로는 기존의 모든 snippets를 나열하거나 새로운 snippets를 만드는 것을 지원하는 views가 될 것입니다.

그리고 다음 코드를 추가합니다.

```python
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

CSRF 토큰이 없는 클라이언트에서 이 view에 POST 할 수 있도록 하려면 view를 csrf_exempt로 표시해야 합니다.

이는 보통 원하는 작업이 아닙니다.

또한 REST Framework views는 실제로 이것보다 더욱 합리적인 행동을 사용하지만, 지금은 목적을 위해 이렇게 진행합니다.

우리는 개별 snippet에 해당하는 view가 필요하며, snippet을 검색, 수정 또는 삭제하는 데 사용할 수 있습니다.

다음 코드를 추가해줍니다.

```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

마지막으로 우리는 이 views를 정리할 필요가 있습니다.

snippets/urls.py 파일을 생성한 다음 아래와 같은 코드를 추가합니다.

```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]
```

snippet 앱의 URLs를 include하기 위해 tutorial/urls.py 파일에 urlconf를 연결해야 합니다.

```python
from django.urls import path, include

urlpatterns = [
    path('', include('snippets.urls')),
]
```

주목할 점은 현재 여기에서 제대로 다루지 못하고 있는 몇가지 엣지 케이스가 있다는 것입니다.

잘못된 형식의 json을 보내거나 view에서 처리할 수 없는 방법으로 요청이 이루어진 경우 500(서버 오류) 응답이 발생합니다.

그렇지만 지금은 이것으로 충분합니다.

<br/>

### Testing our first attempt at a Web API

이제 snippet을 제공하는 샘플 서버를 시작할 수 있습니다.

우선 shell에서 나갑니다.

```shell
quit()
```

그리고 Django의 개발 서버를 시작합니다.

```shell
$ python manage.py runserver

Validating models...

0 errors found
Django version 4.0,1 using settings 'tutorial.settings'
Starting Development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

다른 터미널 창에서 서버를 테스트할 수 있습니다.

저희는 여기에서 curl 이나 httpie를 사용하여 API를 테스트할 수 있습니다.

Httpie는 파이썬으로 작성된 사용자 친화적인 http 클라이언트입니다.

한번 설치해봅시다.

```shell
$ pip3 install httpie
```

마지막으로 모든 snippet 목록을 얻을 수 있습니다.

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

또는 ID를 참조하여 특정 snippet을 얻을 수 있습니다.

```shell
$ http http://127.0.0.1:8000/snippets/2/

HTTP/1.1 200 OK
...
{
  "id": 2,
  "title": "",
  "code": "print(\"hello, world\")\n",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}
```

마찬가지로 웹 브라우저에서 이러한 URL에 접속하여 동일한 json을 표시할 수 있습니다.

---

지금까지 Django REST Framework의 첫 번째 튜토리얼 Serializations를 진행해보았습니다.

튜토리얼을 진행하면서 작성된 API views는 현재 json 응답을 제공하는 것 외에 특별하게 특별한 것은 하고 있지 않습니다.

현재 여전히 정리하고 싶은 일부의 오류 처리 에지 케이스가 있지만 우선은 잘 작동하는 WEB API 입니다.

다음에 진행할 튜토리얼 2부에서 이를 개선할 방법을 알아보도록 하겠습니다.

<br/>
<br/>
<br/>

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
