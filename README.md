# django 튜토리얼


### View 다루기 - HttpResponse 객체


```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5] 
    output = ', '.join([q.question_text for q in latest_question_list]) # system에 저장된 최소한 5개의 투표 질문이 콤마로 분리되어, 발행일에 따라 출력
    return HttpResponse(output)
```
* <h5 style="color:red;"> 문제점 </h5>
    <h6>view에서 페이지의 디자인이 하드코딩 되어있을 때 페이지의 output 을 변경하고 싶다면 python 코드를 편집해야함 </h6>

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

* ###### polls/index.html 템플릿을 불러온 후, context를 전달합니다. context는 템플릿에서 쓰이는 변수명과 python 객체를 연결하는 딕셔너리 값

```python
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

* ###### 템플릿에 context 를 채워넣어 표현한 결과를 HttpResponse 객체와 함께 돌려주는 구문

### View 다루기 - 404 에러

```python
def detail(request,question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request,'polls/detail.html',{'question':question})
```
* ###### view는 요청된 질문의 ID가 없을 경우 Http404 에러 예외를 발생시킨다.

```python
def detail(request,question_id):
    question = get_object_or_404(Question,pk=question_id)
    return render(request,'polls/detail.html',{'question':question})
```

* ###### get_object_or_404 함수는 model을 첫번째 인자로 받고, 키워드 인수를 모델 관리자의 get함수에 넘깁니다. 만약 객체가 존재하지 않을 경우 Http 404 에러 예외가 발생


### View 다루기 - GenericView 사용하기
```python
from django.views.generic import ListView, DetailView

class IndexView(ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        return Question.objects.filter(pub_date__lte=timezone.now()).order_by('-pub_date')[:5]

class QuestionDetailView(DetailView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(DetailView):
    model = Question
    template_name = 'polls/results.html'
```
###### 제너릭 뷰는 일반적인 패턴을 추상화하여 앱을 작성하기 위해 python 코드를 작성하지 않게 해주는 django만의 특별한 기능


### django automation test - 버그 식별하기

* polls/tests.py
```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```
* ###### 미래의 pub_date를 가진 Question 인스턴스 객체를 생성하는 메소드를 가진 django.test.TestCase 하위 클래스를 생성한 후 was_published_recently() 의 출력이 False가 되는지 확인하는 작업
```bash
> python manage.py test polls
```

### 정적 파일 다루기
* polls/static/polls/style.css
```css
li a {
    color: green;
}
```
* polls/templates/polls/index.html
```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

### Django Admin Customizing
* polls/admin.py
```python
from django.contrib import admin

# Register your models here.
from polls.models import Question, Choice

class ChoiceInline(admin.TabularInline): # TabularInline: 테이블 기반 형식
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    # Change question 관련 
    fieldsets = [
        (None, {'fields':['question_text']}),   
        ('Date information', {'fields':['pub_date'],'classes':['collapse']}),
    ]
    inlines = [ChoiceInline]
    # admin Select question to change 관련 
    list_display = ('question_text','pub_date','was_published_recently')  # 보여지는 list
    list_filter = ['pub_date']    # date 관련 필터
    search_fields = ['question_text'] # 검색 필드
admin.site.register(Question,QuestionAdmin)
```
