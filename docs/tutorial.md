# Tutorial

## 1. Requests and responses

```shell
$ django-admin startproject $(project_name)
```

> 프로젝트 이름으로 Python, Django에서 사용 중인 이름은 피해야 한다. (예: test, django, ...)

### 파일 구조

```shell
$(project_name)/
    manage.py
    $(project_name)/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

- `manage.py`: Django 프로젝트와 다양한 방법으로 상호작용할 수 있는 커맨드라인 유틸리티
- `$(project_name)/`: 프로젝트 디렉토리
    - `__init__.py`: Python 패키지로 인식되도록 하는 빈 파일
    - `settings.py`: 프로젝트 설정
    - `urls.py`: URL 선언
    - `wsgi.py`: WSGI 호환 웹 서버 진입점

### 개발 서버

```shell
$ python manage.py migrate
$ python manage.py runserver $(ip:port)
```

- migrate는 데이터베이스에 모델을 생성한다.
- port를 생략하면 기본값인 8000번 포트로 실행된다.

### 앱 생성

```shell
$ python manage.py startapp $(app_name)
```

- `$(app_name)/`: 앱 디렉토리
    - `__init__.py`: Python 패키지로 인식되도록 하는 빈 파일
    - `admin.py`: 관리자 페이지 설정
    - `apps.py`: 앱 설정
    - `migrations/`
        - `__init__.py`: 마이그레이션 파일
    - `models.py`: 데이터베이스 모델
    - `tests.py`: 테스트 코드
    - `views.py`: 뷰 코드

> ### include() 함수
>
> 다른 URLconf를 참조할 때 사용한다. `include()` 함수는 다른 URLconf의 일부를 가져와서, 해당 요청이 그 URLconf의 일부를 처리하도록 한다.
> 즉, URLconf가 polls/urls.py라면, /polls/, /fun_polls/, /content/polls/ 등으로 시작하는 URL은 모두 polls/urls.py로 전달된다.

### path() 함수

```python
path('polls/', include('polls.urls'))
```

- path() 함수는 총 네 개의 인수를 받는다.
    - route
        - URL 패턴을 가진 문자열
        - 첫 번째 패턴부터 시작하여, 일치하는 패턴을 찾을 때까지 요청된 URL을 각 패턴과 리스트의 순서대로 비교한다.
    - view
        - URL 패턴이 일치할 때 호출될 뷰 함수
        - HttpRequest 객체를 첫 번째 인수로 받고, 경로로부터 'route'로 추출된 값들을 키워드 인수로 받는다.
    - kwargs
        - 임의의 키워드 인수를 view에 사정형으로 전달한다.
    - name
        - URL 패턴의 이름을 지정한다.

## 2. Models and the admin site

### DB 연동

- 기본적으로 sqlite3 db가 연동되어 있다.
- db 설정을 바꾸려면 `setting.py` 에 들어가서 `DATABASES` 값을 바꿔야 한다.

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

- `ENGINE` : 연결 할 database
- `NAME` : database의 이름

- SQLite를 사용하지 않는 경우, `USER` , `PASSWORD` , `HOST`
  같은 [추가 설정](https://docs.djangoproject.com/ko/5.0/ref/settings/#std-setting-DATABASES)이 반드시 필요하다.
- 또한 이 시점에서 database를 생성히고 `setting.py` 에 설정된 사용자가 권한이 있는지 확인해야 한다.

- 아래와 같이 언어와 시간도 설정할 수 있다.

```python
LANGUAGE_CODE = "ko-kr"  # 한글
TIME_ZONE = "Asia/Seoul"  # 한국 시간
```

### INSTALLED_APPS

- 장고 설치에서 활성화된 모든 앱을 지정하는 목록
- 기본적으론 아래와 같은 앱들이 있고 직접 추가할 수도 있다.
    - `django.contrib.admin` : 관리용 사이트
    - `django.contrib.auth` : 인승 시스템
    - `django.contrib.contenttypes` : 컨텐츠 타입을 위한 프레임워크
    - `django.contrib.sessions` : 세션 프레임워크
    - `django.contrib.messages` : 메세징 프레임워크
    - `django.contrib.staticfiles` : 정적 파일을 관리하는 프레임워크
- 기본 어플리케이션 중 몇몇은 데이터베이스를 사용하기에 미리 설정해주어야 한다.

- `migrate` 명령으로 `INSTALLED_APPS` 설정을 탐색하고 `settings.py` 에 있는 다양한 설정을 토대로 필요한 데이터베이스 테이블을 생성한다.

```shell
$ python manage.py migrate
```

- 어떤 테이블이 생겼는지 궁금하다면 각 명령어로 확인할 수 있다.
    - \dt(PostgreSQL), SHOW TABLES; (MariaDB, MySQL), .tables (SQLite), SELECT TABLE_NAME FROM USER_TABLES; (Oracle)

- 만약 기본 앱 중 필요하지 않다면 주석 혹은 삭제하고 `migrate` 명령을 실행하면 된다.

### Model

- 모델이란 부가적인 메타데이터를 가진 데이터베이스의 구조(layout)를 말한다.
- 각 모델은 `django.db.models.Model` 의 파생 클래스로 표현 되며 클래스 변수는 각 모델의 데이터베이스 필드를 나타낸다.
- 데이터베이스의 각 필드는 `Field` 클래스의 인스턴스로 표현되며 종류는 아래와 같다.

|    Field 이름     |                      설명                      |
|:---------------:|:--------------------------------------------:|
|    CharField    |  문자열을 저장하는 필드, 최대 길이(max_length)를 설정할 수 있다.  |
|    TextField    |          긴 텍스트를 저장하는 필드, 길이 제한이 없다.          |
|  IntegerField   |                 정수를 저장하는 필드                  |
|   FloatField    |              부동 소수점 수를 저장하는 필드               |
|  DecimalField   |              고정 소수점 수를 저장하는 필드               |
|  BooleanField   |            True, False 값을 저장하는 필드            |
|    DateField    |                 날짜를 저장하는 필드                  |
|  DateTimeField  |               날짜와 시간을 저장하는 필드                |
|    TimeField    |                 시간을 저장하는 필드                  |
|   EmailField    |               이메일 주소를 저장하는 필드                |
|    FileField    |                 파일을 업로드하는 필드                 |
|   ImageField    |               이미지 파일을 업로드하는 필드               |
| ForeignKeyField |             다른 모델과의 관계를 나타내는 필드              |
| ManyToManyField |               다대다 관계를 나태내는 필드                |
|  OneToOneField  |               일대일 관계를 나타내는 필드                |
|    SlugField    | URL에서 사용될 수 있는 문자열을 저장하는 필드, 보통 제목이나 이름에 사용  |
|    URLField     |                 URL을 저장하는 필드                 |
| IPAddressField  |                IP주소를 저장하는 필드                 |
|    UUIDField    | UUID(Universally Unique Identifier)를 저장하는 필드 |
|   BinaryField   |               이진 데이터를 저장하는 필드                |
|  DurationField  |                시간 간격을 저장하는 필드                |

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

```

- `pub_date` 처럼 첫번째 인자로 이름을 지정할 수도 있다. 지정하지 않는다면 컴퓨터가 읽기 쉬운 것으로 임의 지정한다.
- `default` 인수로 기본값을 정의하거나 각 필드에 있는 옵션을 통해 설정을 추가할 수 있다.
- `ForeignKey` 를 통해 각각의 Choice 가 하나의 Question 에 관계된다는 것을 장고에게 알려준다.

#### 모델의 활성화

- 위처럼 모델을 정의하는 것으로 이 앱을 위한 데이터베이스 스키마 생성(CREATE TABLE 문), 각 객체에 접근하기 위한 Python 데이터베이스 접근 API 생성 등의 동작이 수행된다.
- 그러나 가장 먼저 위 앱이 설치되었다는 것을 `settings.py` 에 알려주어야 한다.

```python
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    ...
]
```

- 위와 같이 앱의 구성을 추가한 뒤 아래의 명령을 실행하면 장고가 모델을 변경시킨 사실과 이 사항을 migration으로 저장시키고 싶다는 것을 알게 된다.

```shell
$ python manage.py makemigrations $(app_name)
```

- Migration은 Django가 모델의 변경사항을 디스크에 저장하는 방법이다. `$(app_name)/migrations/0001_initial.py` 파일에서 직접 확인할 수도 있다.

```shell
$ python manage.py sqlmigrate polls 0001
``` 

- 위 명령어로 실행되는 SQL 문장을 직접 볼 수도 있다.
- 그 외 참고사항
    - 사용하는 데이터베이스의 따라서 출력 결과는 다를 수 있다.
    - 테이블 이름은 앱 이름 + 모델 이름(소문자)이 조합되어 자동으로 생성된다. (override하여 다른 동작 가능)
    - 기본 키(ID)가 자동으로 추가된다. (override하여 다른 동작 가능)
    - 관례에 따라 장고는 외래 키 필드명에 `_id` 이름을 자동으로 추가한다. (override하여 다른 동작 가능)
    - 외래 키 관계는 `FOREIGN KEY` 라는 제약에 의해 명시된다. `DEFERRABLE` 부분은 PostgreSQL에 트랜잭션이 끝날 때까지 외래 키를 강제하지 말라고 알려주는 것 뿐이다.
    - 사용하는 데이터베이스에 따라 데이터베이스 고유의 필드타입이 조정된다.
    - `sqlmigrate` 명령은 실제로 데이터베이스에서 마이그레이션을 실행하지 않고 확인만 할 수 있게 도와준다.
    - `python manage.py check` 를 통해 마이그레이션을 수행하거나 데이터베이스를 건드리지 않고 프로젝트의 문제를 확인할 수 있다.

```shell
$ python manage.py migrate
```

- 이제 `migrate` 명령을 통해 아직 적용되지 않은 마이그레이션을 모두 수집해 이를 실행한다. (이때 django는 **django_migrations** 테이블을 두어 마이그레이션 적용 여부를 확인한다.)
- 마이그레이션 기능은 동작 중인 데이터베이스를 자료 손실 없이 업그레이드 하는 데 최적화 되어 있다.

#### 요약

1. `models.py` 에서 모델을 변경
2. `python manage.py makemigrations` 명령으로 변경사항에 대한 마이그레이션 생성
3. `python manage.py migrate` 명령으로 변경사항을 데이터베이스에 적용

### API

```python
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

- `__str__` 메소드로 객체의 표현을 편하게 볼 수 있다. 또한 장고가 자동으로 생성하는 관리 사이트에서도 객체의 표현이 사용되기에 사용하는 것이 좋다.
- 그 외 [모델의 관계에 대한 더 많은 정보](https://docs.djangoproject.com/ko/5.0/ref/models/relations/)
  나 [API 관련 정보](https://docs.djangoproject.com/ko/5.0/topics/db/queries/) 들도 있다.

### Django 관리자

- 장고는 모델에 대한 관리용 인터페이스를 모두 자동으로 생성한다.
- 관리자 생성은 아래와 같이 할 수 있다.

```shell
$ python manage.py createsuperuser
```

- 관리자 사이트는 `로컬 도메인 + /admin/` 으로 접근할 수 있다.
- 편집 가능한 그룹과 사용자와 같은 몇 종류의 컨텐츠를 확인할 수 있으며 이는 `django.contrib.auth` 모듈에서 제공된다.

- 만약 내가 만든 앱도 관리 페이지에서 확인하려면 `$(app_name)/admin.py` 에 다음과 같이 추가해야 한다.

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

## 3. Views and templates

### View

- view는 일반적으로 특정한 기능을 제공하고 특정한 템플릿을 가진 장고 앱에 있는 웹 페이지의 type이다.
- 장고에서는 웹 페이지와 기타 콘텐츠가 view로 전달된다.
- 각 view는 파이썬 함수로 표현된다.
- URL로 부터 view를 얻기 위해 장고는 `URLconfs` 를 사용한다.

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

- 위와 같이 `$(app_name)/views.py` 파일에 view를 추가할 수 있으며 위 view들은 인수를 받기 때문에 모양이 조금 다르다.

```python
urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

- 그리도 `$(app_name)/urls.py` 에 새로운 view를 연결해주어야 한다.
- 만약 "/polls/34" 를 요청한다면 아래와 같은 동작으로 처리된다.
    1. 장고는 `ROOT_URLCONF` 설정이 가리키고 있는 `$(project_name).urls` python 모듈을 로드한다.
    2. 이 모듈은 `urlpatterns` 에 `$(app_name)/` 을 찾아 패턴을 순서대로 이동한다.
    3. 찾았다면 `$(app_name)` 을 제거하고 나머지 텍스트인 `34/` 를 추가 처리하기 위해 `$(app_name).urls` URLconf로 보낸다.
    4. 거기서 일치하는 패턴을 찾아 `detail(request=<HttpRequest object>, question_id=34)` 와 같은 `detail()` view를 호출한다.
    5. 이때 **question_id=34** 는 **<int:question_id>** 에서 왔다.

- 각 view는 요청된 페이지의 내용이 담긴 HttpResponse 객체를 반환하거나 Http404 같은 예외를 발생하게 해야 한다.

### Template

- 만약 `index()` view를 하나 만들었다고 가정했을 때 디자인이 하드코딩 되어 있다면 코드를 계속 편집해야 한다.
- 이러한 상황을 방지하기 위해 **template** 을 사용한다.

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```

- 위 코드와 같이 `settings.py` 에서 장고가 템플릿을 어떻게 불러오고 렌더링 할 것인지 기술한다.
- 기본 설정은 다음과 같다.
    - `APP_DIRS` 가 True 로 `DjangoTemplates` 백엔드를 구성한다.
    - 관례에 따라 `DjangoTemplates` 는 `INSTALLED_APPS` 폴더의 하위 디렉토리인 `templates` 를 탐색한다.
    - 즉 `$(app_name)/templates/$(app_name)` 경로에서 템플릿을 찾는다.

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

- 위와 같이 `loader.get_template()` 함수를 통해 `polls/index.html` 템플릿을 불러와서 context에 전달한다.
- 이때 context는 템플릿에서 쓰이는 변수명과 python 객체를 연결하는 사전형 값이다.

- 템플릿에 context를 채워넣어 표현한 결과를 HttpResponse 객체와 함께 돌려주는 구문은 자주 사용하는 방법이다. 그렇기에 아래와 같은 단축 기능을 제공한다.

```python
from django.shortcuts import render

from .models import Question


def index(request):
  latest_question_list = Question.objects.order_by("-pub_date")[:5]
  context = {"latest_question_list": latest_question_list}
  return render(request, "polls/index.html", context)
```

- `render()` 함수는 request 객체를 첫번째 인수로 받고, 템플릿 이름을 두번째 인수로 받으며 context 사전형 객체를 세번째 선택적 인수로 받는다.
- 그리고 인수로 지정된 context로 표현된 템플릿의 HttpResponse 객체를 반환한다.

### 404 error

```python
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```

- 위와 같이 요청된 질문의 ID가 없는 경우 404 에러를 발생시킨다.
- 이는 자주 쓰이는 용법이며 장고에선 단축 기능을 제공한다.

```python
from django.shortcuts import get_object_or_404, render

from .models import Question

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```

- `get_object_or_404()` 함수는 장고 모델을 첫 번째 인자로 받고, 몇 개의 키워드 인수를 모델 관리자의 `get()` 함수에 넘긴다.
- 만약 객체가 존재하지 않는다면 `Http404` 예외가 발생한다.

### Template 사용하기

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

- 위와 같은 템플릿이 있을 때, 템플릿 시스템은 dot-lookup 문법을 사용한다.
- 예제의 `{{ question.question_text }}` 구문을 보면 다음과 같다.
    - 먼저 **question** 객체에 대해 사전형으로 탐색한다.
    - 탐색에 실패하면 속성값으로 탐색한다.
    - 속성 탐색도 실패하면 리스트의 인덱스 탐색을 시도한다.

### 이름 공간

- 실제 장고 앱은 여러 개의 앱을 사용한다. 그렇기에 장고는 앱들의 url을 구별해야 한다.
- 이를 위해 URLconf에 이름공간을 추가하여 구별하도록 만들어 주어야 한다.

```python
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

이제 템플릿에서 아래와 같이 이름공간으로 나눠진 상세 뷰를 사용할 수 있다.

```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

## 4. Forms and generic views