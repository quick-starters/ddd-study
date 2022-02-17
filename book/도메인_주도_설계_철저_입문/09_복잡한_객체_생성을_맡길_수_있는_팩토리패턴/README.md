# 13 복잡한 조건을 나타내기 위한 명세

객체를 평가하기 위해 복잡한 절차가 필요한 경우가 있는데, 이러한 절차를 해당 객체의 메서드로 정의했을 때 취지가 잘 드러나지 않는 경우가 있다. 이럴 때 평가 절차를 별도의 명세라는 객체로 분리할 수 있다. 



## 13.1 명세란?

객체를 평가하는 절차가 단순하다면 평가해야하는 해당 객체의 메서드에 포함해도 되겠지만, 복잡한 평가 절차가 필요한 경우 대상 객체에 메서드로 두는게 자연스럽지 못한 경우가 있다.

이런 절차는 애플리케이션 서비스에 구현되기 마련인데, 이 평가 조건은 도메인 규칙 중에서도 중요도가 높으므로 서비스에 구현하기에 걸맞은 내용이 아니다. 이를 위해 쓰는 것이 명세인데, 어떤 객체가 그 객체의 평가 기준을 만족하는지 판정하기 위한 객체다. 



### 객체의 복잡한 평가 절차

***특정한 조건 충족 여부를 평가하는 메서드***

```c#
public class Circle
{
  private List<UserId> members;
  
  (...생략...)
  
  public bool IsFull()
  
  {
  	return CountMembers() >= 30;
	} 
}
```

어떤 객체가 특정 조건을 만족하는지 간단한 조건이라면 위와 같이 해당 객체의 메서드로 정의한다. 이 정도의 조건이라면 큰 문제가 되지 않는다. 

그럼 더 복잡해진다면? 다음과 같이 규칙이 변경된다고 가정해보자.

1. 사용자 중에는 프리미엄 사용자라는 유형이 존재한다.
2. 서클의 최대 인원은 서클장과 소속 사용자를 포함해 30명이다.
3. 프리미엄 사용자가 10명 이상 소속된 서클은 최대 인원이 50명으로 늘어난다.

서클 객체는 자신에게 소속된 사용자 목록을 저장하고 있지만, `UserId`의 컬렉션을 포함하는 것을 넘어 프리미엄 여부를 알기 위해 사용자 리포지토리를 통해 확인해야한다. 그러나 Circle 객체는 사용자 리포지토리를 가지고 있지 않다. 그러므로 이 리포지토리를 가진 애플리케이션에서 처리해야한다.



***서클의 최대 인원이 조건에 따라 변화하는 경우***

```c#
public class CircleApplicationService 
{
  private readonly ICircieRepository circleRepository; 
  private readonly lUserRepository userRepository;

  (...생략...)
  
  public void Join(CircleJoinCommand command)
  {
    var circleld = new Circleld(command.Circleld); 
    var circle = circleRepository.Find(circleld);
    
    var users = userRepository.Find(circle.Members);
    // 서클에 소속된 프리미엄 사용자의 수에 따라 최대 인원이 결정됨 
    var premiumUserNumber = users.Count(user => user.IsPremium); 
    var circleUpperLimit = premiumUserNumber < 10 ? 30 : 50;
	  if (circle.CountMembers() >= circleUpperLimit)
  	{
  		throw new CircleFullException(circleld); 
  	}
    
	  (...생략...)
	}
}
```

서클의 최대 인원 규칙은 도메인 규칙이다. **서비스는 도메인 규칙에 근거한 로직을 포함하면 안된다.** 이리저리 도메인 주요 규칙이 중복되기 때문이다. 도메인 규칙은 도메인 객체에 정의해야한다. 

`Circle` 클래스의 `isFull` 메서드에 정의할 방법을 찾아보자. 식별자만으로 사용자 정보를 얻으려면 `isFull` 메서드가 리포지토리를 전달받아야 한다.



***리포지토리를 갖게 된 엔티티***

```c#
public class Circle
{
  // 소속된 사용자 중 프리미엄 사용자의 수를 확인해야 하는데
  // 가진 정보는 사용자의 식별자 뿐
  private List<UserId> members;

  (...생략...)

  // 엔티티가 사용자 리포지토리를 갖는다?
  public bool IsFull(IUserRepository userRepository)
  {
    var users = userRepository.Find(Members);
    var premiumUserNumber = users.Count(user => user.IsPremium);
    var circleUpperLimit = premiumUserNumber < 10 ? 30 : 50;
    return CountMembers() >= circleUpperLimit;
  }
}
```

이 방법은 좋지 않다. 리포지토리는 도메인 설계에 포함된다는 점에서 도메인 객체라고 할 수 있지만, 도메인 개념에서 유래한 객체는 아니다. `Circle` 클래스가 사용자 리포지토리를 갖게 되면 도메인 모델을 나타내는데 전념하지 못하게 된다. 엔티티나 값 객체가 도메인 모델을 나타내는 데 전념할 수 있으려면 리포지토리를 다루는 것은 가능한 피해야 한다.

> ⚠️ **의도가 잘 드러나지 않는 객체**
>
> 객체를 평가하는 코드를 곧이 곧대로 해당 객체에 구현하면 객체의 원래 의도가 잘 드러나지 않는다. 다시 말해 이 객체가 무엇이며 어떤 역할을 하는지 잘 알 수 없다. 다음과 같은 코드는 아래에서 다룰명세 같은 외부 객체로 분리하는 선택지도 있다.
>
> ```c#
> public class Circle 
> {
>   public bool IsFull();
>   public bool IsAnniversary(DateTime today);
>   public bool IsRecruiting();
>   public bool IsLocked();
>   public bool IsPrivate();
>   public bool IsJoin(User user);
> }
> ```



### 이 문제의 해결책 - 명세

명세를 이용하면 위와 같은 문제를 해결할 수 있다. 평가 코드를 다음과 같이 명세로 분리해보자.

***서클의 최대 인원에 여유가 있는지 확인하는 명세***

```c#
public class CircleFullSpecification
{
  private readonly IUserRepository userRepository;

  public CircleFullSpecification(IUserRepository userRepository)
  {
    this.userRepository = userRepository;
  }

  public bool IsSatisfiedBy(Circle circle)
  {
    var users = userRepository.Find(circle.Members);
    var premiumUserNumber = users.Count(user => user.IsPremium);
    var circleUpperLimit = premiumUserNumber < 10 ? 30 : 50;
    return circle.CountMembers() >= circleUpperLimit;
  }
}
```

명세는 객체가 조건을 만족하는지 확인하는 역할만 수행한다. 평가 대상 객체가 복잡한 평가 절차 코드에 파묻히는 일 없이 원래 의도를 드러내도록 돕는다.

***명세를 이용한 예***

```c#
public class CircleApplicationService
{
  private readonly ICircleRepository circleRepository;
  private readonly IUserRepository userRepository;
  
  (...생략...)


  public void Join(CircleJoinCommand command)
  {
    var circleId = new CircleId(command.CircleId);
    var circle = circleRepository.Find(circleId);

    var circleFullSpecification = new CircleFullSpecification(userRepository);
    if (circleFullSpecification.IsSatisfiedBy(circle))
    {
      throw new CircleFullException(circleId);
    }

    (...생략...)
  }
}
```

복잡한 객체 평가 코드를 캡슐화해 원래 객체 의도를 잘 드러내게 됐다.



### 리포지토리를 되도록 사용하지 않기

명세도 도메인 객체이므로 내부에서 일어나는 입출력(리포지토리 사용)을 최대한 억제해야 한다는 의견도 있다. 이런 경우 일급 컬렉션(first-class collection)을 이용하는 방법이 있다. 

> 🔍 **[일급 컬렉션](https://jojoldu.tistory.com/412)**
>
> List 등의 제네릭 컬렉션 객체 대신 특화된 컬렉션 객체를 사용하는 패턴이다.



서클의 소속 사용자를 일급 컬렉션으로 나타내보자.

***서클의 소속 사용자를 나타내는 일급 컬렉션***

```c#
public class CircleMembers
{
  private readonly User owner;
  private readonly List<User> members;

  public CircleMembers(CircleId id, User owner, List<User> members)
  {
    Id = id;
    this.members = members;
  }

  public CircleId Id { get; }

  public int CountMembers()
  {
    return members.Count() + 1;
  }

  public int CountPremiumMembers(bool containsOwner = true)
  {
    var premiumUserNumber = members.Count(member => member.IsPremium);
    if (containsOwner)
    {
      return premiumUserNumber + (owner.IsPremium ? 1 : 0);
    }
    else
    {
      return premiumUserNumber;
    }
  }
}
```

일반적으로 사용되는 List 등과 달리 서클의 식별자와 이에 소속된 사용자 정보를 모두 저장한다. 그리고 독자적인 계산 처리를 메서드로 정의할 수 있다. 



***CircleMember 클래스를 사용한 명세***

```c#
public class CircleMembersFullSpecification
{
  public bool IsSatisfiedBy(CircleMembers members)
  {
    var premiumUserNumber = members.CountPremiumMembers(false);
    var circleUpperLimit = premiumUserNumber < 10 ? 30 : 50;
    return members.CountMembers() >= circleUpperLimit;
  }
}
```

CircleMember를 사용한 명세는 위와 같다. 



***일급 컬렉션 객체에 정보 주입하기***

```c#
var owner = userRepository.Find(circle.Owner);
var members = userRepository.Find(circle.Members);
var circieMembers = new CircleMembers(circle.Id, owner, members); 
var circieFullSpec = new CircleMembersFullSpecification();
if (circieFullSpec.IsSatisfiedBy(circleMenibers)) {
	(...생략...) 
}
```

이렇게 정의한 일급 컬렉션을 적용하기로 했다면 애플리케이션 서비스에서 일급 컬렉션 객체에 정보를 주입하는 절차를 추가로 해야한다. 도메인 객체에서 입출력을 가능한 한 배제해야 한다. 일급 컬렉션을 통해 이를 관철하는데 도움을 받을 수 있다.



## 13.2 명세와 리포지토리의 조합

명세는 리포지토리와도 사용될 수 있다. 리포지토리가 명세를 받아 명세에 전달된 조건과 합치하는 객체를 검색하는 것이다. 

리포지토리에 검색에 필요한 규칙을 작성할 수도 있지만, 중요한 도메인 규칙이 리포지토리로 빠지는 것은 지양해야한다. 이럴 때는 중요 규칙을 명세로 정의한 후 리포지토리에 이를 전달해 중요 코드가 리포지토리로 빠져나가는 것을 방지할 수 있다.



### 추천 서클 검색 기능으로 본 복잡한 검색

사용자가 서클에 가입할 때 사용자에 맞는 서클을 찾을 수 있도록 '활동이 활발한 서클'과 '최근에 결성된 클럽'을 검색할 수 있게 추천 서클 기능을 제공한다고 하자. 다음의 두 조건을 만족하는 서클을 추천 서클로 삼는다.

- 최근 1개월 이내에 결성된 서클
- 소속된 사용자 수가 10명 이상



***리포지토리에 추천 서클 검색 메서드 추가하기***

```c#
public interface ICircleRepository 
{
  (...생략...)
  public List<Circle> FindRecommended(DateTime now);
}
```

추천 서클의 정의가 결정됐으니 추천 서클 검색 기능을 지금까지처럼 리포지토리에 맡기도록해보자. FindRecommended 메서드는 인자를 받은 날로부터 조건에 가장 부합한 서클을 골라주는 메서드다. 



***애플리케이션 서비스에서 추천 서클을 검색하는 코드***

```c#
public class CircleApplicationService
{
  private readonly DateTime now;

  (...생략...)

  public CircleGetRecommendResult GetRecommend(CircleGetRecommendRequest request)
  {
    // 리포지토리에 모두 맡기면 된다
    var recommendCircles = circleRepository.FindRecommended(now);
    return new CircleGetRecommendResult(recommendCircles);
  }
}
```

애플리케이션에서 리포지토리의 메서드를 사용하면된다. 하지만 문제가 있다. 추천 서클을 가려내는 조건이 리포지토리의 구현체에 의존한다는 것이다. 이 조건은 중요도 높은 도메인 규칙이다. 이 규칙이 인프라스트럭처 객체인 리포지토리 구현체에 좌지우지되는 것은 바람직하지 않다.



### 명세를 이용한 해결책

***추천 서클 조건 만족 여부를 판단하는 명세 객체***

```c#
public class CircleRecommendSpecification
{
  private readonly DateTime executeDateTime;
  public CircleRecommendSpecification(DateTime executeDateTime)
  {
    this.executeDateTime = executeDateTime;
  }

  public bool IsSatisfiedBy(Circle circle)
  {
    if (circle.CountMembers() < 10)
    {
      return false;
    }
    return circle.Created > executeDateTime.AddMonths(-1);
  }
}
```

도메인 중요 지식은 도메인 객체로 표현해야한다. 추천 서클 여부 판단 처리는 말 그대로 객체에 대한 평가이므로 명세로 정의할 수 있다.



***명세를 통해 추천 서클 검색하기***

```c#
public class CircleApplicationService
{
  private readonly IUserRepository userRepository;
  private readonly DateTime now;

  (...생략...)
  
  public CircleGetRecommendResult GetRecommend(CircleGetRecommendRequest request)
  {
    var recommendCircleSpec = new CircleRecommendSpecification(now);
    var circles = circleRepository.FindAll();
    var recommendCircles = circles
      .Where(recommendCircleSpec.IsSatisfiedBy)
      .Take(10)
      .ToList();
    
    return new CircleGetRecommendResult(recommendCircles);
  }
}
```

CircleRecommendSpecification 은 추천 서클 조건 만족 여부를 판단하는 객체로 이 명세를 사용해 추천 서클을 검색하는 코드는 위와 같다. 이것으로 추천 서클의 선정 조건을 리포지토리에 구현할 필요가 없게 됐다. 



***명세의 인터페이스 및 구현 클래스***

```c#
public interface ISpecification<T>
{
  public bool IsSatisfiedBy(T value);
}

public class CircleRecommendSpecification : ISpecification<Circle>
{
  (...생략...)
}
```

명세의 메서드를 직접 호출하지 않고 리포지토리에 명세를 전달해 메서드를 호출할 수도 있다(더블 디스패치). 이런 방법을 사용하려면 미리 명세의 인터페이스를 정의한다.



***명세 인터페이스를 사용해 추천 서클을 추려내는 리포지토리***

```c#
public interface ICircleRepository
{
  (...생략...)
  
  public List<Circle> Find(ISpecification<Circle> specification);
}
```

리포지토리는 명세의 인터페이슬ㄹ 사용해 추천 서클을 필터링해 반환한다. 

명세를 인터페이스로 정의하면 리포지토리가 모든 명세 타입을 메서드에 추가로 정의할 필요가 없어진다.  `ISpecification<Circle>` 인터페이스를 구현해 새로운 명세를 정의하고 그대로 인자로 전달하기만 하면 해당 명세에 따른 검색이 가능하다.



***명세를 이용해 구현한 추천 서클 검색***

```c#
public class CircleApplicationService
{
  private readonly IUserRepository userRepository;
  private readonly DateTime now;

	(...생략...)
  
  public CircleGetRecommendResult GetRecommend(CircleGetRecommendRequest request)
  {
    var circleRecommendSpecification = new CircleRecommendSpecification(now);
    
    // 리포지토리에 명세를 전달해 추천 서클을 추려냄(필터링)
    var recommendCircles = circleRepository.Find(circleRecommendSpecification)
      .Take(10)
      .ToList();

    return new CircleGetRecommendResult(recommendCircles);
  }
}
```

명세를 이용해 구현한 추천 서클 검색은 위와 같다. 이 방법으로 추천 서클을 선정하는 조건을 서비스 대신 도메인 객체에 구현할 수 있게됐다.



### 명세와 리포지토리를 함께 사용할 때 생기는 성능 문제

리포지토리에 명세를 전달하는 방법은 도메인 규칙을 도메인 객체에서 유출되지 않게 하고 확장성을 높일 수 있는 방법이지만 단점도 존재한다.



***명세 객체를 이용하는 리포지토리 구현체***

```c#
public class CircleRepository : ICircleRepository
{
  private readonly SqlConnection connection;

  (...생략...)
  
  public List<Circle> Find(ISpecification<Circle> specification)
  {
    using (var command = connection.CreateCommand())
    {
      // 모든 정보를 가져오는 쿼리
      command.CommandText = "SELECT * FROM circles";
      using (var reader = command.ExecuteReader())
      {
        var circles = new List<Circle>();
        while (reader.Read())
        {
          // 인스턴스를 생성해 조건에 부합하는지 확인(조건을 만족하지 않으면 버림)
          var circle = CreateInstance(reader);
          if (specification.IsSatisfiedBy(circle))
          {
            circles.Add(circle);
          }
        }
        return circles;
      }
    }
  }
}
```

`ICircleRepository` 구현 클래스를 보자. 명세에 정의된 조건과 부합 여부는 객체 생성 후 명세의 메서드를 통해 확인해야 알 수 있다. 결과적으로 모든 서클의 정보를 받아온 후 하나하나 조건 평가를 해야한다. 데이터 건수가 많아지면 매우 느린 작업이 될 수 있다.

**리포지토리에서 명세를 필터로 이용할 때는 항상 성능을 염두에 두자.**



### 복잡한 쿼리는 리드모델로

추천 서클 검색과 같은 특정 조건 만족 객체 검색 기능은 소프트웨어라면 반드시 포함되는 기능이다. 이러한 기능은 대부분 사용자 편의성을 위한 기능으로 성능 면에서도 요구사항이 높은 경우가 많다.

이런 상황이라면 명세와 리포지토리를 결합해 사용하는 패턴을 사용하지 않는 것도 고려해야 한다. 



***서클 목록을 받아오는 코드***

```c#
public class CircleApplicationService
{
  (...생략...)
  
  public CircleGetSummariesResult GetSummaries(CircleGetSummariesCommand command)
  {
    // 모든 서클의 목록을 받아옴
    var all = circleRepository.FindAll();
    
    // 페이징 처리
    var circles = all
      .Skip((command.Page - 1) * command.Size)
      .Take(command.Size);
    var summaries = new List<CircleSummaryData>();
    foreach(var circle in circles)
    {
      // 각 서클의 서클장에 해당하는 사용자 정보 검색
      var owner = userRepository.Find(circle.Owner);
      summaries.Add(new CircleSummaryData(circle.Id.Value, owner.Name.Value));
    }
    
    return new CircleGetSummariesResult(summaries);
  }
}
```

위 코드는 서클의 목록을 받아온 후 각 서클의 서클장이 되는 사용자의 정보도 함께 받아오는 코드다. 

이 코드에는 두 가지 문제가 있다.

1. 코드 초반에 모든 서클의 목록을 가져온다. 페이징 처리가 포함돼 있으므로 모든 서클의 목록이 필요하지 않다. 시스템 자원을 들여 복원한 대부분의 인스턴스는 한번의 참조 없이 버려진다.

2. 서클에 소속된 사용자 정보를 받아오는 검색이 반복문을 통해 여러 번 실행된다. 일반적인 SQL을 상정하면 대량의 쿼리가 쓰이는 셈이다. 원래라면 JOIN 문 등을 이용해 하나의 쿼리로 만들 수도 있다.

위의 코드는 정상적으로 동작하지만 최적화와는 거리가 멀다. 도메인 지식을 모아둔다는 목적으로는 일리가 있겠지만, 이 이유만으로 사용자 편의성을 위한 최적화 요청을 외면할 수 있을까?

시스템의 애초 존재 의의는 사용자의 문제를 해결하는 것이다. 도메인의 보호를 이유로 사용자에게 불편을 강요하는 것은 결코 옳은 일이 아니다.

복잡한 읽기 작업에서 성능 문제가 우려된다면 이러한 부분에 한해 도메인 객체의 제약에서 벗어나는 방법도 가능하다.



***최적화를 위해 직접 쿼리를 실행하는 코드***

```c#
public class CircleQueryService
{
  private readonly DatabaseConnectionProvider provider;

  public CircleQueryService(DatabaseConnectionProvider provider)
  {
    this.provider = provider;
  }

  public CircleGetSummariesResult GetSummaries(CircleGetSummariesCommand command)
  {
    var connection = provider.Connection;
    using (var sqlCommand = connection.CreateCommand())
    {
      sqlCommand.CommandText = @"
 SELECT
   circles.id as circleId,
   users.name as ownerName
 FROM circles
 LEFT OUTER JOIN users
 ON circles.ownerId = users.id
 ORDER BY circles.id
 OFFSET @skip ROWS
 FETCH NEXT @size ROWS ONLY
";
      
      var page = command.Page;
      var size = command.Size;
      sqlCommand.Parameters.Add(new SqlParameter("@skip", (page - 1) * size));
      sqlCommand.Parameters.Add(new SqlParameter("@size", size));
      
      using (var reader = sqlCommand.ExecuteReader())
      {
        var summaries = new List<CircleSummaryData>();
        while (reader.Read())
        {
          var circleId = (string) reader["circleId"];
          var ownerName = (string) reader["ownerName"];
          var summary = new CircleSummaryData(circleId, ownerName);
          summaries.Add(summary);
        }

        return new CircleGetSummariesResult(summaries);
      }
    }
  }
}
```

위의 코드는 페이징을 위한 쿼리를 직접 실행하는 예이다.



***ORM(EntityFramework)가 적용된 경우***

```c#
public class EFCircleQueryService
{
  private readonly MyDbContext context;

  public EFCircleQueryService(MyDbContext context)
  {
    this.context = context;
  }

  public CircleGetSummariesResult GetSummaries(CircleGetSummariesCommand command)
  {
    var all =
      from circle in context.Circles
      join owner in context.Users
      on circle.OwnerId equals owner.Id
      select new { circle, owner };

    var page = command.Page;
    var size = command.Size;

    var chunk = all
      .Skip((page - 1) * size)
      .Take(size);

    var summaries = chunk
      .Select(x => new CircleSummaryData(x.circle.Id, x.owner.Name))
      .ToList();

    return new CircleGetSummariesResult(summaries);
  }
}
```

이러한 방식은 ORM에서도 적용 가능한데, 적용된 모델의 예는 위와 같다.



### CQRS 패턴

프레젠테이션 계층의 요구에 특화된 유스케이스는 편의성에 신경 쓴 시스템이라면 거의 필수다. 읽기 대상 내용은 복잡하지만, 동작 내용 자체는 도메인 로직이라 할 만한 것이 거의 없다. 반대로 쓰기 작업에는 도메인에 의한 제약이 있는 경우가 많다.

이 때문에 쓰기 작업에서는 도메인과 결합을 느슨하게 하기 위해 도메인 객체 등을 적극적으로 활용하지만, 읽기 작업은 그렇지 않다. 이런 아이디어는 `CQS(Command-query separation)` 또는 `CQRS(command-query responsibility sgregation)`라는 개념에서 온 것이다. 

이들 개념은 객체의 메서드를 성격에 따라 커맨드와 쿼리로 크게 나눠 다르게 다루는 것이 요점이다. 프레젠테이션 계층의 성능적 요구를 만족하면서도 시스템의 통제를 늦추지 않는 효과를 얻을 수 있다.

> 🔍 **지연 실행(Lazy Processing)**
>
> 리포지토리를 그대로 쓰면서도 성능 문제를 해결하는 방법이 있다. 
>
> ***지연 실행을 고려한 리포지토리 인터페이스***
>
> ```c#
> public interface ICircleRepository
> {
>   IEnumerable<Circle> FindAll();
> }
> ```
>
> 지연 실행을 사용하려면 인터페이스의 검색 메서드의 반환 타입을 IEnumerable로 바꿔야한다. IEnumerable 은 컬렉션 타입이지만, 실제 요소에 접근하기 전까지 컬렉션이 확정되지 않는다.
>
> 
>
> ***지연 실행 리포지토리를 적용한 예***
>
> ```c#
> public class CircleApplicationService
> {
>   private readonly ICircleRepository circleRepository;
> 
>   public CircleGetSummariesResult GetSummaries(CircleGetSummariesCommand command)
>   {
>     // 아직은 데이터를 받아오지 않았다
>     var all = circleRepository.FindAll();
>     // 야기서는 페이징 처리 조건만 부여한 것으로 데이터를 받지는 않았다
>     var chunk = all
>       .Skip((command.Page - 1) * command.Size)
>       .Take(command.Size);
>     // 이 시점에서 처음으로 컬렉션의 요소에 접근했으므로 조건에 따라 데이터를 받아온다
>     var summaries = chunk
>       .Select(x =>
>               {
>                 var owner = userRepository.Find(x.Owner);
>                 return new CircleSummaryData(x.Id.Value, owner.Name.Value);
>               })
>       .ToList();
>     return new CircleGetSummariesResult(summaries);
>   }
> }
> ```
>
> 해당 리포지토리를 쓰는 코드는 위와 같다. `FindAll` 메서드를 호출해 모든 서클 객체 컬렉션을 만든다. 이 시점에는 컬렉션 요소에 직접 접근하지 않으므로 실제 데이터를 받아오지 않는다. 그 뒤 페이징에서도 마찬가지다. 실제 받아오는 시점은  `ToList` 메서드를 실행하면서 컬렉션 내용이 확정되는 순간이다. 이 시점에서야 앞서 부여된 페이징 조건에 따라 데이터를 필터링해 받아온다.
>
> 이런식으로 실제 데이터가 필요한 시점까지 처리를 미루는 것을 지연실행이라고 한다. C#에서 쓰이는 EntityFramework도 이 기능을 지원한다. 하지만 이를 사용한다는 것은 특정 기술에 코드가 종속된다는 것이기도 하다.



## 13.3 정리

객체의 평가는 그 내용만으로도 지식이다. 명세는 객체를 평가하는 조건과 절차를 모델링한 객체이다. 객체의 평가를 객체 자체에 맡기는 방법도 있지만 이는 항상 바람직한 방법은 아니다. 

이번 장에서 리포지토리에 명세 객체를 전달해 필터링 하는 방법도 다뤘지만 항상 만능은 아니며 성능 문제도 생길 수 있다.

읽기 작업은 단순화하지만 최적화를 필요로 하는 경우가 많다. 도메인 제약을 잠시 잊고 클라이언트 편의성에 집중할 줄도 알아야한다.
