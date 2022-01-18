# Chapter 02 - 시스템 특유의 값을 나타내기 위한 '값 객체'

## 2.1 값 객체란?

프로그래밍 언어에는 원시 데이터 타입이 있다. 이 원시 데이터 타입만 사용해 시스템을 개발할 수도 있지만, 때로는 시스템 특유의 값을 정의해야 할 때가 있다. 이러한 **시스템 특유의 값을 표현하기 위해 정의하는 객체를 값 객체라고 한다.**

```c#
// 원시 데이터 타입의 값으로 '성명' 나타내기

var fullName = "naruse masanobu";
Console.WriteLine(fullName); // naruse masanobu라는 값을 출력
```

시스템마다 다양한 방법으로 이 성명을 다룰 것이다. 예를 들면 우리나라에서는 성명을 성과 이름 순서로 출력하며, 외국이라면 성만 표시하는 시스템도 있을 것이다. 다음 코드는 이렇게 서로 다른 요구 사항에 대응하기 위해 변수 fullName을 사용해 성씨만 출력하는 예제다.

```c#
// 이름 중 성씨만 출력하기

var fullName = "naruse masanobu";
var tokens = fullName.Split(' '); // ["naruse", "masanobu"]와 같은 배열이 만들어짐
var lastName = token[0];
Console.WriteLine(lastName); // naruse가 출력됨
```

조금 번잡스러워 보이지만 fullName 변수가 단순한 문자열 타입이기 때문에 다른 방법이 없다. 다른 곳에서도 성씨만 출력할 일이 생긴다면 이 부분의 로직을 붙여넣으면 되니 문제가 없을 것이다. 정말 과연 그럴까?

이 로직이 제대로 동작하지 않는 상황이 있다.

```c#
// 성씨만 출력할 때 문제가 생기는 경우

var fullName = "john smith";
var tokens = fullName.Split(' '); // ["john", "smith"]와 같은 배열이 만들어짐
var lastName = token[0];
Console.WriteLine(lastName); // john이 출력됨
```

"john smith"씨의 성은 'smith'다. 코드가 제대로 동작하지 않는 것이다. 이름을 쓰는 관습에 따라 성씨가 앞에 오는 경우도 있고 뒤에 오는 경우도 있기 때문이다.

객체 지향 프로그래밍에서는 이런 문제를 해결하기 위해 일반적으로 클래스를 사용한다. 앞에서는 성명을 문자열 타입으로 다뤘지만, 새로 FullName 클래스를 정의했다."

```c#
// 이름을 나타내기 위한 FullName 클래스

class FullName
{
    public FullName(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
 
    public string FirstName { get; }
    public string LastName { get; }
}
```

성씨 값이 필요하다면 FullName 클래스의 LastName 프로퍼티를 사용하면 된다.

```c#
// FullName 클래스의 LastName 프로퍼티 사용하기

var fullName = new FullName("masanobu", "naruse");
Console.WriteLine(fullName.LastName); // naruse가 출력됨
```

변수 fullName은 이름 그대로 성명을 나타내는 객체로, 값을 표현한다. **객체이기도 하고 동시에 값이기도 하다. 따라서 값 객체라고 부른다.**

시스템에서 필요로 하는 값이 원시 데이터 타입이 아닐 수도 있다는 것을 알았다. **시스템에서 어떤 처리를 해야 하는지에 따라 값을 나타내는 적합한 표현이 정해진다. 도메인 주도 설계에서 말하는 값 객체는 이렇듯 시스템 특유의 값을 나타내는 객체다.**



## 2.2 값의 성질과 값 객체 구현

값에도 일정한 성질이 있다. 값의 성질을 아는 것은 값 객체를 이해하기 위해 중요한 사항이다.

값의 성질로는 대표적으로 다음 세가지가 있다.

- 변하지 않는다.
- 주고받을 수 있다.
- 등가성을 비교할 수 있다.



**값 객체는 시스템 특유의 값에 대한 표현이며, 값의 한 종류다. 값의 성질은 값 객체에도 그대로 적용된다.**



### 2.2.1 값의 불변성

값은 변화하지 않는 성질을 갖는다.

```c#
// 값을 수정하는 예

var greet = "안녕하세요";
Console.WriteLine(greet); // '안녕하세요'가 출력됨
greet = "Hello";
Console.WriteLine(greet); // 'Hello'가 출력됨
```

**우리가 값을 수정할 때는 새로운 값을 대입한다. 사실 대입은 값을 수정하는 과정이 아니다. 대입을 통해 수정되는 것은 변수의 내용이지, 값 자체가 수정되는 것은 아니다. 값은 처음부터 끝까지 변하지 않는다.**

값이 변했다면 다음과 같은 일이 일어나야한다.

```c#
// '값을 수정'하는 의사 코드

var greet = "안녕하세요";
greet.ChangeTo("Hello"); // 실제로는 이런 메서드가 없다
Console.WriteLine(greet); // 'Hello'가 출력된다
```

ChangeTo 메서드는 값을 수정하는 역할을 한다. 위와 같은 코드가 가능하다면 다음과 같은 코드도 가능하다는 말이다.

```c#
// '값을 수정'하는 의사 코드

"안녕하세요".ChangeTo("Hello"); // 실제로는 이런 메서드가 없다
Console.WriteLine("안녕하세요"); // Hello가 출력된다
```

값을 수정할 수 있다면 안심하고 값을 사용할 수 없다. 1이라는 숫자가 갑자기 0이 된다면 혼란을 일으킬 것이다. 1이라는 숫자는 항상 1이어야 한다. 값은 변하지 않기 때문에 안심하고 사용할 수 있는 것이다.



```c#
// 일반적으로 볼 수 있는 값 수정

var fullName = new FullName("masanobu", "naruse");
fullName.ChangeLastName("sato");
```

이 코드는 대부분의 개발자에게 자연스럽게 받아들여진다. 그러나 FullName 클래스를 값으로 간주할 경우 부자연스러운 부분이 생긴다. 바로 값이 수정되기 때문이다.

FullName은 시스템 특유의 값을 표현하는 값 객체다. 그리고 FullName은 값이기도 하다. 그러므로 변하지 않아야 한다. FullName 클래스에 값을 수정하는 기능을 제공하는 ChangeLastName 같은 메서드가 정의되어서는 안 된다.



>**불변하는 값의 장점 & 단점**
>
>- 장점
>  - 상태가 변화하지 않게 하는 것은 프로그램을 단순하게 만들 가능성이 있는 제약이다.
>  - 값이 변하는 상황을 고려할 필요가 없기 때문에 병렬 및 병행 처리를 비교적 쉽게 구현할 수 있다.
>  - 메모리가 부족할 때도 객체를 캐싱하는 전략을 취할 수 있다. → 리소스 절약
>- 단점
>  - 객체의 일부 값만 바꾸고 싶을 때도 객체를 아예 새로 생성해야 한다.
>
>가변 객체를 불변 객체로 바꾸는 것보다는 불변 객체를 가변 객체로 만드는 것이 노력이 적게 들기 때문에 가변 객체와 불변 객체 중 결정을 내리기가 어려울 때는 일단 불변 객체를 적용하는 것이 낫다.



### 2.2.2 교환 가능하다.

값은 불변이다. 그러나 값을 수정하지 않고서는 목적을 달성할 수 있는 소프트웨어를 만들기는 어렵다. 값은 불변일지라도 값을 수정할 필요는 있다.

**'변하지 않는' 성질을 갖는 값은 값 자체를 수정할 수 없다. 이것은 값 객체 또한 마찬가지다. 값 객체의 수정 역시 값과 마찬가지로 대입문을 통해 교환의 형식으로 표현된다.**

```c#
// 값 객체를 수정하는 방법

var fullName = new FullName("masanobu", "naruse");
fullName = new FullName("masanobu", "sato");
```



### 2.2.3 등가성 비교 가능

숫자끼리 혹은 문자끼리처럼 같은 종류의 값끼리는 비교할 수 있다. 예를 들어 표현식 0 == 0 에서 좌변의 0과 우변의 0은 인스턴스로서는 별개의 존재(최적화 등을 통해 동일한 인스턴스가 되는 경우도 있다.)지만, 값은 값으로 취급된다.

이것이 의미하는 바는 값은 값 자신이 아니라 값을 구성하는 속성을 통해 비교된다는 점이다. **시스템 고유의 값인 값 객체도 이와 마찬가지로 값 객체를 구성하는 속성(인스턴스 변수)을 통해 비교된다.**

```c#
// 값 객체 간의 비교
var nameA = new FullName("masanobu", "naruse");
var nameB = new FullName("masanobu", "naruse");
 
// 두 인스턴스를 비교
Console.WriteLine(nameA.equals(nameB)); // 인스턴스를 구성하는 속성이 같으므로 true
```

그러나 가끔 객체끼리의 비교에서 속성값을 꺼내서 직접 비교하는 경우가 있다. 그러나 객 체가 값이라는 사실을 생각하면 부자연스럽다.

**값 객체는 시스템 고유의 값이다. 결국 값이다. 따라서 값의 속성을 꺼내서 비교하는 것이 아니라, 값과 마찬가지로 직접 값끼리 비교하는 방식이 자연스럽다.**



```c#
// 속성을 직접 비교할 경우 새로운 속성 추가하기

var compareResult = nameA.FirstName == nameB.FirstName
                    && nameA.LastName == nameB.LastName
                    && nameC.MiddleName == nameB.MiddleName // 조건식이 추가됨
```

값 객체이 비교를 위한 메서드가 없어서 코드에서 속성을 직접 꺼내 비교해야할 경우, 새로 추가된 속성이 생겼을 때 비교하는 코드를 모두 수정해야 한다.

이 정도의 수정이라면 간단하게 생각할 수도 있다. 그렇지면 객체를 비교하는 곳이 한 곳이 아니라면 수정 난이도는 크게 상승한다.

**그러나, 값 객체에서 직접 비교수단을 제공한다면 단순하지만 복잡해질 수 있는 작업을 피할 수 있다.**



## 2.3 값 객체가 되기 위한 기준

사실 시스템에서 사용되는 개념 중 어디까지 값 객체로 만들어야 하는가도 어려운 문제다. 단순히 도메인 모델로 정의되는 개념은 값 객체로 정의할 수 있지만, 그렇지 않은 경우에는 혼란을 낳는다.

예를 들어 FullName의 FirstName과 LastName도 이름을 나타내는 값 객체와 성을 나타내는 값 객체로 만드는 경우, 정도가 '지나치다'고 보는 사람도 있는가 하면, '괜찮다'고 보는 사람도 있다.

**이것이 적당한지에 대한 기준은 상황에 따라 달라진다. 어느 한쪽이 항상 옳은 것도 아니고, 다른 쪽이 항상 틀린 것도 아니다.**



**필자는 개인적으로 도메인 모델로 선정되지 못한 개념을 값 객체로 정의해야 할지에 대한 기준으로 '규칙이 존재하는가'와 '낱개로 다루어야 하는가'라는 점을 중요하게 본다.**

성명을 예로 들면, '성과 이름으로 구성된다'는 규칙이 있다. 또 앞서 본문에서 봤듯이 '낱개로 다루어지는' 정보다. 필자의 판단기준에 비춰보면 성명은 값 객체로 정의해야 할 개념이 된다.



성 혹은 이름은 어떻게 될까? 현재로서는 시스템 상에 성과 이름에 대한 제한은 없다. 성만 사용하거나 이름만 사용하는 화면도 아직은 없다. 필자라면 성과 이름을 값 객체로 만들지 않을 것이다.

전제를 살짝 바꿔서 만약 성과 이름에서 사용 가능한 문자에 제약이 있다면 어떻게 될까? 물론 이 경우에도 값 객체로 정의하지 않고 인자를 전달받아 생성하는 시점에 검사를 하면 규칙을 강제할 수 있다.



중요한 것은 값 객체 정의를 피하는 것이 아니다. 값 객체로 정의할 필요가 있는지를 판단하고, 만약 그렇다면 대담하게 행동으로 옮기는 것이 중요하다.

**그리고 값 객체로 정의할 만한 가치가 있는 개념을 구현 중에 발견했다면 그 개념은 도메인 모델로 피드백해야 한다.**



## 2.4 행동이 정의된 값 객체

**값 객체에서 중요한 점 중 하나는 독자적인 행위를 정의할 수 있다는 점이다. 값 객체는 데이터만을 저장하는 컨테이너가 아니라 행동을 가질 수도 있는 객체다.**

```c#
// 금액을 더하는 처리를 구현하기

class Money
{
    private readonly decimal amount;
    private readonly string currency;
     
    ...
    ...
     
    public Money Add(Money arg) {
        if (arg == null) throw new ArgumentNullException(nameof(arg));
        if (currency != arg.currency) throw new ArgumentException($"화폐 단위가 다름(this:{currency}, arg:{arg.currency})");
     
        return new Money(amount + arg.amount, currency);
    }
}
```

화폐 단위가 일치하지 않는 경우에는 예외를 발생시키므로 잘못된 계산을 예방할 수 있다. 값 객체는 불변이기 때문에 새로운 인스턴스로 반환된다.

**값 객체는 결코 데이터를 담는 것만이 목적인 구조체가 아니다. 값 객체는 데이터와 더불어 데이터에 대한 행동을 한곳에 모아둠으로써 자신만의 규칙을 갖는 도메인 객체가 된다.**



### 2.4.1 정의되지 않았기 때문에 알 수 있는 것

객체에 정의된 행위를 통해 이 객체가 어떤 일을 할 수 있는지 알 수 있다. **이를 반대로 생각하면 객체는 자신에게 정의되지 않은 행위는 할 수 없다는 말도 된다.**



## 2.5 값 객체를 도입했을 때의 장점

당연한 일이지만 시스템 고유의값을 객체로 나타내면 그만큼 정의하는 클래스의 수도 늘어난다. 원시 타입 값을 '잘 활용하는' 방법으로 개발해왔기 때문에 많은 수의 클래스를 정의하는 것을 껄끄러워하는 개발자도 많다. 값 객체를 도입하려면 여기에 따르는 심리적 장애물을 넘어야 한다.

어떤 이유로 값 객체가 필요한지 이해한다면 좀 더 많은 사람이 값 객체의 필요성을 느끼고 프로젝트에 도입하게 될 것이다.

값 객체의 장점은 크게 다음 네 가지로 시스템을 보호하는 데 크게 도움이 된다.



### 2.5.1 표현력의 증가

공산품에는 로트 번호나 일련번호, 제품번호 등 식별을 위한 다양한 번호가 부여된다. 이들 번호는 숫자만으로 구성되기도 하고 알파벳이 섞인 문자열 형태도 있다. 제품번호를 원시 타입으로 나타낸 프로그램은 어떨가?

```c#
// 원시 타입으로 정의한 제품번호

var modelNumber = "a20421-100-1";
```

modelNumber는 원시 타입인 문자열 타입의 변수다. 그러나 코드 상에서 갑자기 modelNumber라는 변수를 맞닥뜨리게 되면 제품번호의 내용이 어떤 것인지 예측하기 어렵다. 제품번호의 내용을 알려면 modelNumber 변수가 어디서 만들어져 어떻게 전달됐는지 따라가 보는 수밖에 없다.

```c#
// 제품번호를 나타내는 값 객체

class ModelNumber
{
    private readonly string productCode;
    private readonly string branch;
    private readonly string lot;
     
    public ModelNumber(string productCode, string branch, string lot)
    {
        if (prouductCode == null) throw new ArgumentNullException(nameof(productCode));
        if (branch == null) throw new ArgumentNullException(nameof(branch));
        if (lot == null) throw new ArgumentNullException(nameof(lot));
 
        this.productCode = productCode;
        this.branch = branch;
        this.lot = lot;
    }
     
    public override string ToString()
    {
        return productCode + "-" + branch + "-" + lot;
    }
}
```

ModelNumber 클래스의 정의를 살펴보면 제품번호가 제품코드(productCode)와 지점번호(branch), 로트번호(lot)로 구성됨을 알 수 있다. 아무 정보를 제공하지 않는 문자열과 비교하면 큰 진보다.

**값 객체는 자기 정의를 통해 자신이 무엇인지에 대한 정보를 제공하는 자기 문서화를 돕는다.**



### 2.5.2 무결성의 유지

시스템에는 각 값이 준수해야 할 규칙이 있다. 예를 들면 사용자명은 3 글자 이상, 10 글자 이하, 그리고 알파벳 문자만을 포함한다.

```c#
// 존재할 수 없는 값의 예

var username = "me";
```

사용자명을 의미하는 변수 username은 길이가 2인 문자열이다. 그러므로 규칙을 위반하는 이상값이다. 그러나 프로그램상에 길이가 2인 문자열이 존재하는 데는 아무런 문제가 없다. 컴파일러의 관점에서 정상적이기 때문이다. 따라서 세 글자 이하로 유효하지 않은 사용자명임에도 이 값이 존재할 수 있다.

**유효하지 않은 값은 효과가 느젝 나타나는 독과 같이 다루기 까다로운 상대다. 유효하지 않은 값을 허용하는 경우 값을 사용할 때 항상 값이 유효한지 확인을 거쳐야 한다.**

```c#
// 값을 사용하기 전 값이 유효한지 확인하기

if (userName.Length >= 3)
{
    // 유효한 값이므로 처리를 계속한다.
}
else
{
    throw new Exception("유효하지 않은 값");
}
```

값의 유효성을 매번 확인하면 급한 불은 끌 수 있지만, 그로 인해 코드 여기저기에 유효성을 검사하는 코드가 반복될 것이다. 자칫 한곳이라도 잘못되면 시스템 오류로 이어질 수 있다.

**값 객체를 잘 이용하면 유효하지 않은 값을 처음부터 방지할 수 있다.**

```c#
// 사용자명을 나타내는 값 객체
class UserName
{
    private readonly string value;
     
    public UserName(string userName)
    {
        if (value == null) throw new ArgumentNullException(nameof(value));
        if (value.Length < 3) throw new ArgumentException("사용자명은 3글자 이사잉어야 함", nameof(value));
 
        this.value = value;
    }
}
```



### 2.5.3 잘못된 대입 방지

대입은 코드에서 매우 자주 사용되는 문법으로, 개발자라면 일상적으로 사용할 것이다.

그만큼 일상적인 행위이기 때문에 개발자도 가끔(혹은 자주) 대입문을 잘못 사용하는 경우가 있다.

```c#
// 간단한 대입문의 예

User CreateUser(string name)
{
    var user = new User();
    user.Id = name;
    return user;
}
```

**코드가 올바른지 아닌지를 판단하기 위해 관계자의 기억이나 문서를 뒤지는 것보다는 자기 문서화의 힘을 빌리는 것이 바람직하다. 코드가 올바른지 아닌지를 코드로 나태날 수 있다면 그보다 더 나은 방법은 없다.** 값 객체를 통해 이를 실현할 수 있다.

```c#
// 사용자 ID를 나타내는 값 객체

Class UserId
{
    private readonly string value;
     
    public UserId(string value)
    {
        if (value == null) throw new ArgumentNullException(nameof(value));
         
        this.value = value;
    }
}
```

```c#
// 사용자명을 나타내는 값 객체

Class UserName
{
    private readonly string value;
     
    public UserName(string value)
    {
        if (value == null) throw new ArgumentNullException(nameof(value));
         
        this.value = value;
    }
}
```

UserId와 UserName 클래스는 각각 원시타입인 문자열을 래핑한 단순한 객체다. 행동은 아직 정의되지 않았지만, 지금 다루는 문제를 해결하기에는 이 정도면 충분하다.



```c#
// 값 객체를 사용하도록 수정된 User 클래스

Class User
{
    public UserId Id { get; set; }
    public UserName Name { get; set; }
}
```

함수의 인자로 문자열이 아닌 UserName 객체를 전달받게 한다.



```c#
// 원시 타입 대신 값 객체를 사용한 예

User CreateUser(UserName name)
{
    var user = new User();
    user.Id = name; // 컴파일 에러 발생
    return user;
}
```



### 2.5.4 로직을 한곳에 모아두기

DRY 원칙에서 밝혔듯이 코드 중복을 방지하는 일은 매우 중요하다. 중복된 코드가 많아지면 코드를 수정하는 난도가 급상승한다.

```c#
// 입력값 검증이 포함된 사용자 생성 처리

void CreateUser(string name)
{
    if (name == null) throw new ArgumentNullException(nameof(name));
    if (name.Length < 3) throw new ArgumentException("사용자명은 3글자 이상이어야 함", nameof(name));
 
    var user = new User(name);
     
    ...
}
```

사용자 정보를 수정하는 부분이 이곳 하나라면 이 코드는 문제가 없다 .그러나 이 외에 사용자 정보를 수정하는 처리가 있다면 어떻게 될까

```c#
// 사용자 정보 수정 시에도 같은 검증이 필요하다

void UpdateUser(string name)
{
    if (name == null) throw new ArgumentNullException(nameof(name));
    if (name.Length < 3) throw new ArgumentException("사용자명은 3글자 이상이어야 함", nameof(name));
 
    var user = new User(name);
     
    ...
}
```

위의 코드에서도 인자를 검증하는 코드가 중복된다. **이런 중복 코드의 단점은 규칙이 수정됐을 때 드러난다.**

값 객체를 정의해 그 안에 규칙을 정리하면 이를 방지할 수 있다.

```c#
// 값 객체 안에 정리된 규칙

Class UserName
{
    private readonly string value;
     
    public UserName(string value)
    {
        if (value == null) throw new ArgumentNullException(nameof(value));
        if (value.Length < 3) throw new ArgumentException("사용자명은 3글자 이상이어야 함", nameof(value));
         
        this.value = value;
    }
}
```

이 값 객체를 사용하면 사용자 신규 생성과 사용자 정보 수정 처리를 다음과 같이 구현할 수 있다.

```c#
// 값 객체를 이용해 구현한 사용자 신규 생성과 사용자 정보 수정

void CreateUser(string name)
{
    var userName = new UserName(name);
    var user = new User(userName);
     
    ...
}
 
void UpdateUser(string id, string name)
{
    var userName = new UserName(name);
     
    ...
}
```

규칙이 UserName 클래스 안에 기술되어 있으므로 '사용자명의 최소 길이'를 변경할 때 수정할 곳은 UserName 클래스 안으로 국한된다. **규칙을 기술한 코드가 한 곳에 모여 있다면 수정할 곳도 한곳뿐이라는 의미다.**



## 2.6 정리

**값 객체의 개념은 '시스템 고유의 값을 만드는' 단순한 것이다.** 시스템에는 해당 시스템에서만 쓰이는 값이 반드시 있게 마련이다.

도메인에는 다양한 규칙이 표함된다. **값 객체를 정의하면 이러한 규칙을 값 객체 안에 기술해 코드 자체가 문서의 역할을 할 수 있다.** 시스템의 명세는 일반적으로 문서에 정리되는데, 이때 코드로 규칙을 나타낼 수 있다면 더 나을 것이다.

**값 객체는 도메인 지식을 코드로 녹여내는 도메인 주도 설계의 기본 패턴이다.**