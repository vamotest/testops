# tests-utils


# Index
- [Allure Integration](#allure-integration)
  - [allure-pytest](#allure-pytest)
  - [pytest-cov](#pytest-cov)
  - [CI Artifacts](#ci-artifacts)
  - [Local Allure](#local-allure)
- [Allure](#allure)
  - [step](#step)
  - [issue](#issue)
  - [feature](#feature)
  - [story](#story)
- [tests-utils](#tests-utils)
  - [allure_utils](#allure_utils)
    - [attach_json](#attach_json)
    - [attach_txt](#attach_txt)
    - [attach_html](#attach_html)
  - [compare_utils](#compare_utils)
    - [compare_str](#compare_str)
    - [compare_sql](#compare_sql)
    - [compare_dict](#compare_dict)


<br><br>


## Allure Integration
The `Allure Framework' is a flexible and lightweight multilingual test reporting tool that not only shows a very concise representation of what has been tested in a convenient web report form but also allows everyone involved in the development process to get the most out of the day-to-day test execution.

<div align="center">
  <img width=700" height="597" src="https://i.ibb.co/gSbnmP9/allure-integrations.jpg">
</div>


<br><br>


**[⬆ Back to Index](#index)**
### allure-pytest
[allure-pytest](https://pypi.org/project/allure-pytest/) is a plugin for the [pytest](https://github.com/pytest-dev/pytest) test framework.
Documentation is available [online](https://docs.qameta.io/allure-report/), you can also get help on the [gitter](https://gitter.im/allure-framework/allure-core) channel.

Poetry installation:
```shell script
poetry add --dev allure-pytest
```

Then in `pytest.ini_options` you need to add:
```
[tool.pytest.ini_options]
addopts = "--alluredir=artifacts/allure_results"
```

The `artifacts` hierarchy is as simple as possible. It includes the `allure_results` directory, where the results of our runs will be directly recorded.
```
artifacts
└── allure_results
```

It is also better to add the `artifacts` directory to `.gitignore` to keep artefacts out of the index and not interfere with local development


<br><br>


**[⬆ Back to Index](#index)**
### pytest-cov
При этом `artifacts` можно расширить и сохранять там также результаты тестового покрытия вашего приложения, 
для этого потребуется установить плагин [pytest-cov](https://github.com/pytest-dev/pytest-cov):
```shell script
poetry add allure-pytest
```

После в `pytest.ini_options` необходимо добавить:
```
[tool.pytest.ini_options]
addopts = "--cov-report term --cov=. --cov-report html:artifacts/coverage_report"
```

При этом `artifacts` должен содержать директорию `coverage_report`:
```
artifacts
├── allure_results
├── coverage_report
└── .gitignore
```


<br><br>


**[⬆ Back to Index](#index)**
### CI Artifacts
С точки зрения сохранения тестовых артефактов для job'ы используется блок artifacts.
Он нужен для того, чтобы данные артефакты мы могли передавать на Allure-сервер.

На текущем этапе пока можно скачивать артефакты из job'ы. С порядком установки allure локально вы можете ознакомиться [тут](https://docs.qameta.io/allure/#_installing_a_commandline).
**Для сборки артефактов:**
```bash
allure serve path/to/allure_results
```


<br><br>


**[⬆ Back to Index](#index)**
### Local Allure
Для удобства локального тестирования вы можете добавить [allure-docker-service](https://github.com/fescobar/allure-docker-service-examples/blob/master/allure-docker-python-pytest-example/docker-compose.yml)
в свой `docker-compose.yml`

```yaml
version: '3.9'

services:
  app: &django_conf
    container_name: "appname"
    volumes:
      - ./app:/app
      - ./app/artifacts/allure_results:/app/artifacts/allure_results
 
  allure:
    container_name: "appname-allure"
    image: "frankescobar/allure-docker-service"
    environment:
      CHECK_RESULTS_EVERY_SECONDS: 5
    ports:
      - "5050:5050"
    volumes:
      - ./app/artifacts/allure_results:/app/allure-results
```

По-умолчанию стоит порт **5050**, но при большом количестве приложений в рамках кластера удобно было сделать так:
* **S.Calendar** - 5001
* **S.Chart** - 5002
* **S.Control** - 5003
* **S.Home** - 5004
* **S.Progress** - 5005
* **S.Roomer** - 5006

Отчет по-дефолтному порту можно посмотреть [тут](http://localhost:5050/allure-docker-service/projects/default/reports/latest/index.html).

**Параметры:**
* `CHECK_RESULTS_EVERY_SECONDS` - генерирует отчет каждое заданное время из артефактов директории `/app/allure-results`
* `KEEP_HISTORY` - отвечает за сохранение истории. 

Но так как результаты прогонов при выполнении тестов записываются не мгновенно, то они разбиваются визуально 
на несколько прогонов, что не совсем корректно. Для решения данный проблемы нужно сильно увеличивать 
значение параметра `CHECK_RESULTS_EVERY_SECONDS`, но это не так удобно в рамках локального тестирования.
Именно поэтому данный параметр отсутствует в блоке `environment`


<br><br>


**[⬆ Back to Index](#index)** 
## Allure
<div align="center">
  <img width=700" height="373" src="https://i.ibb.co/drhVgmK/allure-stats-photo-resizer-ru.png">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
### step

```python
@allure.step('Запускаем')
def launch_dags(dag_ids):
    for dag_id in dag_ids:
        with allure.step(f'Запускаем {dag_id}'):
            dag_run_id = launch_dag(dag_id)

            with allure.step('Запущен?'):
                assert dag_run_id
```
В данном случае для абстрактной сущности [DAG](https://airflow.apache.org/docs/apache-airflow/1.10.12/concepts.html#:~:text=Core%20Ideas-,DAGs,and%20their%20dependencies) мы делаем запуск по его `dag_id`.
Данные о полученных `dag_ids` сохраняются в другом методе.

Каждый из таких `allure.step`'ов очень удобен для отделения шагов. Также, в самом отчете будет отображаться конечное 
время на выполнение каждого такого степа. Что позволит анализировать на каком этапе у вас проблемы.

<div align="left">
  <img width=500" height="573" src="https://i.ibb.co/0stbfbd/photo-2023-01-13-14-13-24-1.jpg">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
### issue
Декоратор `@allure.issue` позволяет в отчете сохранять ссылку на задачу в JIRA. 
Безумно удобно нажать и узнать в случае возникновении проблемы когда создавалась задача и как она менялась.

```python
# tests/constants.py
class Constants:
    JIRA_URL: str = f'https://{BASE_JIRA_URL}/browse/{task_id}'

# tests/my_awesome_test.py
@allure.issue(Constants.JIRA_URL.format(task_id='DEV-5019'))
def test_one():
    ...
```

<div align="left">
  <img width=300" height="210" src="https://i.ibb.co/4TbfQ8z/photo-2023-01-13-14-22-24.jpg">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
### feature
Декоратор `@allure.feature` позволяет группировать тесты для удобного отображения

```python
# tests/constants.py
class Constants:
    class Features:
        sinks: str = 'Sinks'
        extractors: str = 'Extractors'


# tests/my_awesome_test.py
class TestFeatures:

    @allure.feature(Constants.Features.sinks)
    def test_sinks(self):
        ...

    @allure.feature(Constants.Features.extractors)
    def test_extractors(self):
        ...
```

<div align="left">
  <img width=400" height="442" src="https://i.ibb.co/3cMXTtQ/photo-2023-01-13-14-29-03.jpg">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
### story
Декоратор `@allure.story`. Аналогично прошлому декоратору позволяет группировать тесты для удобного отображения, 
при этом вы можете видеть в отчете как `feature`, так и `story`

```python
# tests/my_awesome_test.py
class TestStory:

    @allure.feature(Constants.Features.sinks)
    @allure.story('Story title №1')
    def test_sinks_story_1(self):
        ...

    @allure.feature(Constants.Features.sinks)
    @allure.story('Story title №2')
    def test_sinks_story_2(self):
        ...

    @allure.feature(Constants.Features.extractors)
    @allure.story('Story title №1')
    def test_extractors(self):
        ...
```

<div align="left">
  <img width=400" height="184" src="https://i.ibb.co/MGfrMZK/photo-2023-01-13-14-33-07.jpg">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
## tests-utils

**[⬆ Back to Index](#index)** 
### Установка
Для тех кто использует poetry:
```python
poetry add git+https://...tests-utils@v.1.0.1
```

Релизы смотреть тут, также об изменениях буду писать в канале.
Пространство и репозиторий специально публичные, любой из вас может установить на своих проектах.
Сейчас минимальный набор методов, который покрывает максимальное количество кейсов.

Хочу собрать обратную связь по использованию и пожеланиям.
Обязательно буду еще делать клиенты и обвязки над сетевыми и сервисными запросами, а также их артефактами.

Во всех проектах кластера **Стройка** `tests-utils` уже стоит.
На ревью backend задач теперь буду требовать использование вышеперечисленного рекомендаций


<br><br>


**[⬆ Back to Index](#index)** 
### allure_utils
Чтобы разработчикам и AQA было чуть удобнее с точки зрения написание автотестов, то был реализован класс AllureTestUtils.
Данный класс представляет небольшую обертку к allure, в рамках которого сохранение артефактов идет чуть удобнее,
как с точки зрения вызова, так и с точки зрения форматирования артефактов в итоговом отчете.

Для чего это нужно? По ходу написания вашего теста, у вас могут быть некоторые промежуточные результаты, 
на основе которых у нас формируется конечный результат. При этом каждый из этих результатов будет в красивом и отформатированном виде в Allure.

Например, сходили в какую-то ручку или сервис, получили ответ. Отфильтровали его содержимое и сохранили.
Далее уже в параметризованном тесте проверили каждый элемент данной коллекции.

Это актуально как для фикстур, тестов, хелперов, утилит.

**Использование `allure_utils`:**
```python
from tests_utils import allure_utils
```


<br><br>


**[⬆ Back to Index](#index)** 
#### attach_json
Данный метод позволяет сохранять `data: dict` с `indent=2`.

**Пример:**
```python
class TestAllureUtils:
  def test_attach_json(self, fake_dict):
      with allure.step('Получаем данные'):
          with allure.step('Сохраняем данные'):
              data = allure_utils.attach_json(fake_dict, 'Attach JSON')
  
          with allure.step('Проверяем, что данные корректно кодируются'):
              assert isinstance(data, str)
  
          with allure.step('Проверяем, что кодированые данные корректно декодируются'):
              decode_data = json.loads(data)
              assert isinstance(decode_data, dict)
```

**Отображение:**
<div align="left">
  <img width=500" height="595" src="https://i.ibb.co/L6bfX8c/2023-01-13-15-58-38.png">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
#### attach_txt
Данный метод позволяет сохранять любой тип данных `data: Any`.


**Пример №1:**
```python
class TestAllureUtils:
    def test_attach_txt_str(self, fake_str):
        data = allure_utils.attach_txt(fake_str, 'Attach str')
        assert isinstance(data, str)
```

**Отображение:**
<div align="left">
  <img width=500" height="272" src="https://i.ibb.co/VtwG39w/2023-01-17-10-54-56.png">
</div>

**Пример №2:**
```python
# fake_uuid = uuid.uuid4()
# isinstance(fake_uuid, uuid.UUID)

class TestAllureUtils:
    def test_attach_txt_uuid(self, fake_uuid):
        data = allure_utils.attach_txt(fake_uuid, 'Attach uuid.UUID')
        assert isinstance(data, str)
```

**Отображение:**
<div align="left">
  <img width=500" height="267" src="https://i.ibb.co/tQztWfY/2023-01-17-10-54-43-1.png">
</div>


<br><br>


**[⬆ Back to Index](#index)** 
#### attach_html
Данный метод позволяет сохранять некую строку как HTML, если она такой является. В противном случае как обычную строку.

**Пример №1:**
```python
class TestAllureUtils:
    @pytest.mark.parametrize(
        'html_candidate',
        [
            '<html><body><h1>Hello</h1></body></html>',
            '<html><body><h1>Hello'
        ],
        ids=['with_html', 'with_broken_html']
    )
    def test_attach_html(self, html_candidate):
        data = allure_utils.attach_html(html_candidate, 'Attach HTML')
        assert isinstance(data, str)
        assert allure_utils._is_html(html_candidate)    
```

**Отображение `with_html`:**
<div align="left">
  <img width=500" height="351" src="https://i.ibb.co/RQ2VSCC/2023-01-17-11-58-20.png">
</div>

**Отображение `with_broken_html`:**
<div align="left">
  <img width=500" height="349" src="https://i.ibb.co/GV5BsZD/2023-01-17-12-00-02.png">
</div>

<br>

**Пример №2:**
```python
class TestAllureUtils:
    def test_attach_html_with_str(self, fake_str):
      html_candidate = fake_str
      data = allure_utils.attach_html(html_candidate, 'Attach str')
      assert isinstance(data, str)
      assert not allure_utils._is_html(html_candidate)
```

**Отображение:**
<div align="left">
  <img width=500" height="267" src="https://i.ibb.co/D43NxXX/2023-01-17-12-03-03.png">
</div>

<br><br>


**[⬆ Back to Index](#index)** 
### compare_utils
С точки упрощения зрения написание автотестов был реализован класс CompareUtils
для сравнения некоторого фактического и ожидаемого результат.

Под капотом он также использует [allure_utils](#allure_utils) для форматирования и сохранения артефактов.
Так как методы данного класса содержат `assert`'ы, то класс
является своего рода `check`'ером. Избавляет нас от написания одинаковых конструкций сравнений в тестах.

**Использование `compare_utils`:**
```python
from tests_utils import compare_utils
```

<br><br>



**[⬆ Back to Index](#index)** 
#### compare_str
Данный метод позволяет сравнивать строку с фактическим и ожидаемым результатом.
Если строки идентичны, то наша проверка пройдут успешно. В противном случае мы получим AssertionError, с отличиями

**Пример №1:**
```python
class TestCompareUtilsStr:
    def test_compare_str(self, fake_str):
        assert not compare_utils.compare_str(fake_str, fake_str)
```

**Отображение:**
<div align="left">
  <img width=500" height="344" src="https://i.ibb.co/vH7S9xQ/2023-01-17-14-23-56.png">
</div>


<br>

**Пример №2:**
```python
class TestCompareUtilsStrException:
    def test_compare_str_exception(self, fake_str):
        with pytest.raises(AssertionError):
            replaced_str = random_string_replace(fake_str, calculate_replace_count(fake_str))
            compare_utils.compare_str(fake_str, replaced_str)
```

**Отображение:**
<div align="left">
  <img width=500" height="558" src="https://i.ibb.co/5RDytRR/2023-01-17-14-58-56.png">
</div>

<br><br>

**[⬆ Back to Index](#index)** 
#### compare_dict
Данный метод позволяет сравнивать словари с фактическим и ожидаемым результатом.
При этом через параметр `ignore_order` мы можем указать важен ли нам порядок ключей или нет.
В случае возникновения исключения мы получаем информацию об удаленных или добавленных ключах.

**Пример №1:**
```python
class TestCompareUtilsDict:
    def test_compare_dict(self, fake_dict):
        assert not compare_utils.compare_dict(fake_dict, fake_dict)
```

**Отображение:**
<div align="left">
  <img width=500" height="640" src="https://i.ibb.co/FgCDPmY/2023-01-17-15-23-00.png">
</div>

<br>

**Пример №2:**
```python
class TestCompareUtilsDictException:
    def test_compare_dict_exception(self, fake_dict):
        result_data = fake_dict
        expected_data = random_dict_modification(result_data)
        with pytest.raises(AssertionError):
            compare_utils.compare_dict(result_data, expected_data)
```

**Отображение:**
<div align="left">
  <img width=500" height="615" src="https://i.ibb.co/gJzNW6G/2023-01-17-15-54-51.png">
</div>
