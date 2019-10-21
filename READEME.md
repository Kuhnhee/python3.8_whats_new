# What's New In Python 3.8

본 리포지토리에는 Python 3.8(Oct. 14th. 2019) 에서 추가된 주요사항들을 정리합니다.

[원문 링크](https://docs.python.org/3/whatsnew/3.8.html)



### 할당 연산자

표현식 내에서 변수에 값을 할당하는 "The walrus operator" `:=`가 추가되었습니다. 아래와 같은 예시가 있습니다.

```python
if (n := len(a)) > 10:
    print(f"리스트가 너무 깁니다 ({n}개 요소 발견, <= 10 여야 합니다.)")
```

위 예에서 `n := len(a)` 는 `n` 변수에 `len(a)`를 할당함과 동시에, `if`문 내에서 조건을 확인해주는 `n == len(a)`과 동일한 기능을 합니다. 결과적으로, `len()`함수가 두 번 호출되어야 하는 코드에서 한 번만 호출해도 되도록 해줍니다.

비슷한 예로, 정규 표현식 탐색 과정에서도 패턴 일치 발새여부 확인과 부분집합 추출을 동시에 할 수 있습니다.

```python
discount = 0.0
if (mo := re.search(r'(\d+)% discount', advertisement)):
    discount = float(mo.group(1)) / 100.0
```

위 예에서 찾고자 하는 정규 표현식과 매칭되는 문자열을 탐색하고, `mo` 변수에 할당하는 과정을 한 번에 했음을 확인할 수 있습니다.

`while`문의 조건절에서도 "walrus operator"를 사용할 수 있습니다.

```python
while (block := f.read(256)) != '':
	process(block)
```

List comprehension에서도, filtering condition에서 값 연산이 필요한 경우 위의 표현식을 사용할 수 있습니다.

```python
[clean_name.title() for name in names 
if (clean_name := normalize('NFC', name)) in allowed_names]
```



### Positional-only parameters

`/`를 사용하는 새로운 문법에 의해, 위치가 고정되어야 하는 매개변수를 지정해주고 키워드 인자로 사용되지 않도록 할 수 있다.

다음 예에서 매개변수 a, b는 위치 인자(positional-only)로, c, d는 위치인자 혹은 키워드로, e, f는 키워드로 사용된다.

```python
def f(a, b, /, c, d, *, e, f):
    print(a, b, c, d, e, f)
```

아래와 같이 함수 `f()`를 호출할 수 있다.

```python
f(10, 20, 30, d=40, e=50, f=60)
```

반면, 아래는 유효하지 못한 함수 호출법이다.

```python
f(10, b=20, c=30, d=40, e=50, f=60) # b는 키워드 인자로 사용할 수 없다.
f(10, 20, 30, 40, 50, f=60)	# e는 반드시 키워드 인자로 사용해야 한다.
```

이 기능을 이용해 C로 작성된 파이썬 내장함수의 emulate 또한 가능하다. `pow()` 함수는 아래와 같이 키워드 인자를 허용하지 않는다.

```python
def pow(x, y, z=None, /):
    r = x ** y
    return r if z is None else r%z
```

키워드 인자를 배제하기 위한 용도로도 `/` 문법을 사용할 수 있다. 내장함수 `len()` 함수는 `len(obj, /)`와 같이 작성되어 있어, 아래와 같은 잘못된 함수호출을 금지한다.

```python
len(obj='hello')
```

클라이언트의 코드를 망가트리지 않고, 함수의 매개변수 이름을 수정하는 것을 허용하기도 한다. `statistics` 모듈의 `quantiles` 함수는 `dist`라는 매개변수의 이름이 바뀔 수 있음을 고려해 아래와 같이 구현되어 있다.

```python
def quantiles(dist, /, *, n=4, method='exclusive')
	...
```

`/` 의 좌측에 위치하는 매개변수들은 가능한 키워드로서 노출되지 않기 때문에, 매개변수의 이름들을 `**kwargs`에서 사용할 수 있다.

```python
def f(a, b, /, **kwargs):
    print(a, b, kwargs)
    
f(10, 20, a=1, b=2, c=3)	#a,b가 f()에 정의된 a,b와 다른 의미로 사용된다.
OUT: 10 20 {'a': 1, 'b': 2, 'c': 3}
```



### f-strings `=` 연산자 사용법

`f'{expr=}'` 는 `expr`에 어떠한 값이 저장되어 있는지 보기 쉽게 출력해준다. 

```python
>>> user = '유저이름'
>>> member_since = date(1975, 7, 31)
>>> f'{user=} {member_since=}'
OUT: "user='유저이름' member_since=datetime.date(1975, 7, 31)""
```

[f-string format specifiers](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)를 사용해 더 구체적으로 출력시킬 수 있다.

```python
>>> delta = date.today() - member_since
>>> f'{user=!s} {delta.days=:,d}'
OUT: 'user=유저이름 delta.days=16,075'
```



### Other Language Changes

`yield` 혹은 `return` 명령에 의해 반환되는 iterable 값을 괄호 안에 담을 필요가 없어졌다.

```python
def parse(family):
    lastname, *members = family.split()
    return lastname.upper(), *members

>>> parse('simpsons homer marge bart lisa sally')
OUT: ('SIMPSONS', 'homer', 'marge', 'bart', 'lisa', 'sally')
```



`replace()` 함수의 사용으로 이미 존재하는 함수의 복제판(매개변수가 달라진)을 생성할 수 있다. 아래 예는 `statistics.mean()`함수의 `data` 매개변수가 키워드 인자로 쓰이는 것을 방지하는 코드다.

```python
>>> from statistics import mean
>>> mean(data=[10,20,90])
OUT: 40
    
>>> mean.__code__ = mean.__code__.replace(co_posonlyargcount=1)
>>> mean(data=[10,20,90])
TypeError: mean() got some positional-only arguments passed as keyword arguments: 'data'
```



Dict comprehension도 dict literals와 마찬가지로 key를 먼저 연산하고, value를 나중에 연산하도록 한다.

```python
#Dict comprehension
>>> cast = {input('role? '): input('actor? ') for i in range(2)}
role? King Arthur
actor? Chapman
role? Black Knight
actor? Cleese

#Dict Literal
>>> cast = {input('role? '): input('actor? ')}
role? Sir Robin
actor? Eric Idle
```

key, value 연산 순서가 고정됨에 따라, value를 연산하는 과정에서 key expression을 사용할 수 있게 된다.

```python
>>> names = ['Martin vopn Lowis', 'Lukas Langa', 'Walter Dorwald']
>>> {(n := normalize('NFC', name)).casefold() : n for name in names}
{
    'martin von lowis': 'Martin von Lowis',
	'lukas langa': 'Lukas Langa',
    'walter dorwald': 'Walter Dorwad'
}
```



### New Module

`importlib.metadata`를 통해 third-party 패키지의 메타데이터를 읽을 수 있다. 아래 예는 설치되어 있는 내장 페키지의 버전, 진입점들의 리스트 등을 출력해주는 코드다.

```python
>>> from importlib.metadata import version, requires, files
>>> version('requests')
'2.22.0'
>>> list(requires('requests'))
['chardet (<3.1.0,>=3.0.2)']
>>> list(files('reuquests'))[:5]
[PackagePath('requests-2.22.0.dist-info/INSTALLER'),
 PackagePath('requests-2.22.0.dist-info/LICENSE'),
 PackagePath('requests-2.22.0.dist-info/METADATA'),
 PackagePath('requests-2.22.0.dist-info/RECORD'),
 PackagePath('requests-2.22.0.dist-info/WHEEL')]
```



### Imporved Modules

**ast**

AST의 노드들이 이제 `end_lineno`, `end_col_offset` 인자들을 갖습니다. 이를 활용하여 끝 노드의 정확한 위치를 파악할 수 있습니다. (단, `lieno`, `col_offset`를 가지고 있는 노드에서만 가능합니다.)

새 함수 `ast.get_source_segment()`는 특정 AST 노드의 소스 코드를 반환합니다.

함수 `ast.parse()`에 몇몇 새로운 flag들이 추가되었습니다.

- `type_comments=True`: 특정 AST 노드의 [PEP 484](https://www.python.org/dev/peps/pep-0484/), [PEP 526](https://www.python.org/dev/peps/pep-0526/) 타입의 주석 내용을 출력합니다.

- `mode='func_type'`: [PEP 484](https://www.python.org/dev/peps/pep-0484/) 타입의 주석을 파싱하는데 사용합니다.
- `feature_version=(3, N)`: 이전 Python 3 버전을 특정하는 데 사용할 수 있습니다. 예를 들어, `feature_version=(3, 4)`를 사용할 경우 [async](https://docs.python.org/3/reference/compound_stmts.html#async)와 [await](https://docs.python.org/3/reference/expressions.html#await)를 non-reversed word로 취급합니다.



**asyncio**

`python -m asyncio`를 실행시 natively async REPL을 실행합니다. 이를 활용하여 top-level `await`를 가진 코드를 빠르게 실험할 수 있으며, invokation(프로그램 혹은 함수의 실행) 때마다 새로운 event loop을 생성하는 `asyncio.run()`을 직접 실행할 필요가 없어집니다.

```shell
$ python -m asyncio
asyncio REPL 3.8.0
Use "await" directly instead of "asyncio.run()"
Type "help", "copyright", "credits" or "license" for more information.
>>> import asyncio
>>> await asyncio.sleep(10, result='hello')
hello
```

윈도우 상에서, 기본 event loop는 이제 [`ProcatorEventLoop`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.ProactorEventLoop)입니다. `ProcatorEventLoop`는 이제 UDP를 지원하고, `KeyboardInterrupt`(CTRL+C)로 인터럽트할 수 있습니다.



**builtins**

[`compile()`](https://docs.python.org/3/library/functions.html#compile) 함수가 이제 `ast.PyCF_ALLOW_TOP_LEVEL_AWAIT` 플래그를 받을 수 있습니다. 이 플래그를 통해, `compile()`은 이제 top-level `await`, `async for`, `async with` 를 허용합니다. `CO_COROUTINE` 플래그로 표시된 비동기 코드 객체가 반환됩니다.



**collections**

[`collections.namedtuple()`](https://docs.python.org/3/library/collections.html#collections.namedtuple)의 `_asdict()` 메소드는 이제 `collections.OrderedDict`객체가 아닌 `dict` 객체를 반환한다. 이는 Python 3.7 이후 dict 데이터들의 순서가 보장되도록 바뀌었기 때문에 가능하다. `OrderedDict`의 추가적인 



**cProfile**

[`cProfile.Profile`](https://docs.python.org/3/library/profile.html#profile.Profile) 클래스가 이제 context manager로 사용될 수 있다. 아래와 같은 코드로 코드 블록을 profile할 수 있다.

```python
import cProfile

with cProfile.Profile() as profiler():
    # code to be profiled
	...
```



**csv**

[`csv.DictReader`](https://docs.python.org/3/library/csv.html#csv.DictReader)가 이제 `collections.OrderedDict`객체가 아닌 `dict` 객체를 반환한다.



**curses**

구조화된 버전 정보를 저장하는 변수 `ncurses_version`가 추가되었다.



**ctypes**

윈도우 상에서, `CDLL`클래스와 서브클래스들이 이제 `LoadLibraryEx` 호출의 플래그들을 특정하기 위한 `winmode` 매개변수를 받아들일 수 있다. 



**datetimes**

새로운 생성자인 `datetime.date.fromisocalendar()`와 `datetime.datetime.fromisocalendar()`가 추가되었다. 이들은 ISO year, week number, weekday로부터 각각 `date`, `datetime` 객체를 생성한다. 이는 각 클래스의 `isocalendar` 메소드의 inverse이다.



**functools**

`functools.lru_cache()`가 데코레이터를 반환하는 함수가 아닌, 그 자체를 데코레이터로써 사용할 수 있게 되었다.

```python
@lru_cache
def f(x):
    ...
    
@lru_cache(maxsize=256)
def f(x):
    ...
```

인스턴스의 life time 동안 캐싱된 연산값들을 위한 데코레이터 `functools.cached_property()`가 추가되었다.

```python
import functools
import statistics

class Dataset:
    def __init__(self, sequence_of_numbers):
        self.data = sequence_of_numbers
        
    @functools.cached_property
    def variance(self):
        return statistics.variance(self.data)
```

[Single dispatch](https://docs.python.org/3/glossary.html#term-single-dispatch)를 사용해서 메서드를 [generic functions](https://docs.python.org/3/glossary.html#term-generic-function)로 변환시켜주는 데코레이터인 `functools.singledispatchmethod()`가 추가되었다.

```python
from functools import singledispatchmethod
from contextlib import suppress

class TaskManager:

    def __init__(self, tasks):
        self.tasks = list(tasks)

    @singledispatchmethod
    def discard(self, value):
        with suppress(ValueError):
            self.tasks.remove(value)

    @discard.register(list)
    def _(self, tasks):
        targets = set(tasks)
        self.tasks = [x for x in self.tasks if x not in targets]
```



**gc**

[`get_objects()`](https://docs.python.org/3/library/gc.html#gc.get_objects) 함수가 이제 가져올 객체의 세대를 정하는 *generation* 값을 매개변수로 받을 수 있다. 



**gettext**

[`pgettext()`](https://docs.python.org/3/library/gettext.html#gettext.pgettext)와 그 변형 함수들이 추가되었다.



**gzip**

[`gzip.compress()`](https://docs.python.org/3/library/gzip.html#gzip.compress) 함수에 재생산성 있는 출력을 위한 *mtime* 매개변수가 추가되었다.

특정 잘못된 타입 혹은 손상된 gzip 파일일 경우 [`OSError`](https://docs.python.org/3/library/exceptions.html#OSError) 대신 [`BadGzipFile`](https://docs.python.org/3/library/gzip.html#gzip.BadGzipFile) 예외가 발생하게 됩니다.



**IDLE and idlelib**

N개(기본값 50) 이상의 출력 라인이 발생할 경우 버튼 하나로 묶이게 됩니다. Settings dialog의 General page의 PyShell 구간에서 N값을 변경할 수 있다. 

Customize 설정된 모듈을 실행시키기 위한 "Run Customized" 기능이 Run 메뉴에 추가되었습니다. 커맨드 라인에 입력된 인자값들은 모두 `sys.argv`에 추가되게 되며, 이들은 다음 customized 실행 때 box에 다시 나타나게 됩니다. 이용자는 필요에 따라 일반 Shell main module의 재시작을 막을 수 있습니다.

Python string과 Tcl 객체 간의 변환을 위한 OS native encoding이 사용됩니다. 이로 인해 IDLE 상에서 emoji 등 non-BMP character를 사용할 수 있습니다.



**inspect**

`__slots__` 인자가 value가 doctsring인 `dict`타입일 경우, [`inspect.getdoc()`](https://docs.python.org/3/library/inspect.html#inspect.getdoc) 가 `__slots__`의 docstring을 찾을 수 있습니다.

```python
class AudioClip:
    __slots__ = {'bit_rate': 'expressed in kilohertz to one decimal place',
                 'duration': 'in seconds, rounded up to an integer'}
    def __init__(self, bit_rate, duration):
        self.bit_rate = round(bit_rate / 1000.0, 1)
        self.duration = ceil(duration)
```



**io**

개발 모드(`-X env`)와 디버그 빌드에서, `close()` 메서드가 실패할 경우 `io.IOBase` finalizer가 예외를 로깅합니다. 배포 빌드에서는 기본적으로 위의 예외는 무시됩니다.



**itertools**

[`itertools.accumulate()`](https://docs.python.org/3/library/itertools.html#itertools.accumulate) 함수에 초기값을 지정하기 위한 *initial* 키워드가 추가되었습니다.

```python
>>> from itertools import accumulate
>>> list(accumulate([10, 5, 30, 15], initial=1000))
[1000, 1010, 1015, 1045, 1060]
```



**json.tool**

`--json-lines` 옵션을 추가하여 모든 입력 라인들을 파싱하여 분리된 JSON object를 생성할 수 있습니다.



**logging**

[`logging.basicConfig()`](https://docs.python.org/3/library/logging.html#logging.basicConfig) 에 *force* 키워드가 추가되었습니다. 이 키워드가 *True*일 경우 root logger와 연결된 모든 핸들러가 제거되며, 다른 인자에 의해 정의된 구성을 수행하기 전까지 닫히게 됩니다.

이를 통해 logger 혹은 *basicConfig()*을 호출했을 때, *basicConfig()*을 뒤따르는 호출들이 무시되던 문제를 해결할 수 있게 됩니다. 이 문제로 인해 수정, 테스트, logging 설정들에 대한 교육 등을 interactive prompt 혹은 Jupyter notebook으로 실행하는 데 어려움을 겪었었습니다.



**math**

두 점 간의 *유클리드 거리*를 구하기 위한 [`math.dist()`](https://docs.python.org/3/library/math.html#math.dist) 가 추가되었습니다.

[`math.hypot()`](https://docs.python.org/3/library/math.html#math.hypot) 를 확장하여 다중차원을 다룰 수 있도록 하였습니다. (기존에는 2-D의 경우에만 지원)

[`math.prod()`](https://docs.python.org/3/library/math.html#math.prod) 함수를 추가하였습니다. *start*값을 시작으로, *iterable* 값들을 모두 곱한 값들을 반환합니다.

```python
>>> prior = 0.8
>>> likelihoods = [0.625, 0.84, 0.30]
>>> math.prod(likelihoods, start=prior)
0.126
```

조합론 함수인 [`math.perm()`](https://docs.python.org/3/library/math.html#math.perm) 와 [`math.comb()`](https://docs.python.org/3/library/math.html#math.comb)가 추가되었습니다.

```python
>>> math.perm(10, 3)    # Permutations of 10 things taken 3 at a time
720
>>> math.comb(10, 3)    # Combinations of 10 things taken 3 at a time
120
```

부동소수점으로 변환하지 않고, 정밀한 정수 제곱근을 계산하기 위한 [`math.isqrt()`](https://docs.python.org/3/library/math.html#math.isqrt) 함수가 추가되었습니다. 이 함수는 상대적으로 큰 정수에 대해서도 지원되며, `floor(sqrt(n))`보다 빠르고 [`math.sqrt()`](https://docs.python.org/3/library/math.html#math.sqrt)보다 느립니다.

```python
>>> r = 650320427
>>> s = r ** 2
>>> isqrt(s - 1)         # correct
650320426
>>> floor(sqrt(s - 1))   # incorrect
650320427
```

[`math.factorial()`](https://docs.python.org/3/library/math.html#math.factorial)가 더이상 int가 아닌(int-like이 아닌) 인자를 받지 않습니다.



**mmap**

[`mmap.mmap`](https://docs.python.org/3/library/mmap.html#mmap.mmap) 클래스가 이제 `madvise()` 시스템 콜에 접근하기 위한 [`madvise()`](https://docs.python.org/3/library/mmap.html#mmap.mmap.madvise)메서드를 가집니다.



**multiprocessing**

[`multiprocessing.shared_memory`](https://docs.python.org/3/library/multiprocessing.shared_memory.html#module-multiprocessing.shared_memory) 모듈이 추가되었습니다.



**OS**







References

- https://docs.python.org/3/whatsnew/3.8.html

