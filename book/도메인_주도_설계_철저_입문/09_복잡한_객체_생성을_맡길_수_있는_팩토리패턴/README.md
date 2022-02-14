# 9. 복잡한 객체 생성을 맡길 수 있는 '팩토리 패턴'

## 9.1 팩토리의 목적

객체 지향 프로그래밍에 쓰이는 클래스는 도구 그 자체다. 메서드의 사용 방법만 알면 클래스 내부 구조를 몰라도 누구든 사용할 수 있다.

그러나 도구는 편리함만큼이나 복잡한 구조를 갖는다.

복잡한 객체는 객체를 생성하는 처리도 그만큼 복잡하다. 복잡한 객체를 생성하기 위한 복잡한 처리는 도메인 모델을 나타낸다는 객체의 애초 취지를 불분명하게 만든다. 그렇다고 클라이언트에만 맡기는 것도 좋은 방법이 아니다. 객체 생성 과정 자체는 도메인에 큰 의미가 없을지 몰라도, 도메인을 나타내기 위한 계층의 책임임에는 변함이 없기 때문이다.



여기서 바로 객체 생성 과정을 객체로 정의할 필요가 생긴다. 이렇게 객체 생성을 책임지는 객체를 마치 도구를 만드는 공장과도 같다고 해서 '팩토리'라고 부른다. **팩토리는 객체 생성 과정과 관련 지식이 정리된 객체**다.



## 9.2 번호 매기기를 구현한 팩토리의 구현 예

지금까지는 User 클래스의 인스턴스를 생성할 때 객체의 식별자로 GUID를 사용했다.



```c#
public class User
{
  private readonly UserId id; 
  private UserName name;
  
  // 사용자를 최초 생성할 때 실행되는 생성자 메서드
  public User(UserName name)
  {
    if (name == null) throw new ArgumentNullException(nameof(name));
    // 식별자로 GUID를 사용한다
    Id = new UserId(Guid.NewGuid().ToString()); 
    this.name = name;
  }
  
  // 사용자 객체를 복원할 때 실행되는 생성자 메서드 
  public User(UserId id, UserName name)
  {
    if (id == null) throw new ArgumentNullException(nameof(id));
    if (name == null) throw new ArgumentNullException(nameof(name));
    this.id = id;
    this.name = name; 
  }
  (...생략...) 
}
```



User 클래스는 생성자 메서드를 2개 갖추고 있다. UserId 객체를 인자로 받는 생성자 메서드는 사용자 객체를 복원하기 위한 용도이며, 나머지 다른 생성자 메서드는 사용자 객체를 최초 생성할 때 사용한다. 

사용자 객체를 최초 생성할 때 함께 생성되는 GUID는 유일한 식별자로 사용할 수 있는 무작위 문자열이므로, 항상 유일 식별자임이 보장된다.



그러나 시스템에 따라 식별자가 매겨지는 과정을 통제해야 하는 경우가 있다. 이렇게 번호를 매기는 처리는 어떻게 구현해야 할까?



전통적인 방법은 시퀀스나 테이블을 이용하는 방법이 있다. 

시퀀스를 이용해 번호를 매기도록 수정해보자

```c#
public class User 
{
  private readonly UserId id; 
  private UserName name;
  
  public User(UserName name) 
  {
    string seqId;
    // 데이터베이스 접속 설정에서 커넥션을 생성 
    var connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString; 
    using (var connection = new SqlConnection(connectionString))
    using (var command = connection.CreateCommand())
    {
      connection.Open();
      // 번호 매기기용 테이블을 이용해 번호를 매김 
      command.CommandText = "SELECT seq = (NEXT VALUE FOR UserSeq)";
      
      using (var reader = command.ExecuteReader())
      {
      	if (reader.Read())
        {
          var rawSeqId = reader["seq"];
          seqId = rawSeqId.ToString(); 
				}
        else 
        {
          throw new Exception(); 
        }
      } 
    }
  	id = new UserId(seqId); 
    this.name = name;
  	(...생략...)
  }
}
```

위 코드는 그리 바람직한 코드가 아니다. 추상화 수준이 높은 개념인 User에 데이터베이스를 다루는 낮은 추상화 수준의 처리가 작성돼 있기 때문이다. 



테스트 목적으로 대충 인스턴스를 생성하고 싶을 때는 적당히 식별자를 붙여주고, 그렇지 않은 경우에는 데이터베이스에서 제대로 된 식별자를 매겨주고 싶을 때 팩토리를 사용한다.



먼저 팩토리 인터페이스를 정의한다.

```c#
public interface IUserFactory 
{
	User Create(UserName name); 
}
```



팩토리에는 UserName을 인자로 받아 User의 인스턴스를 반환하는 메서드가 정의되어 있는데, 앞으로 User 객체를 새로 만들때는 생성자 메서드 대신 이 메서드를 사용한다.

User를 생성하는 처리는 위 인터페이스를 구현한 클래스가 맡게 된다.



다음은 시퀀스를 이용해 번호를 매기는 팩토리의 구현이다.

```c#
public class UserFactory : IUserFactory 
{
  public User Create(UserName name) 
  {
    string seqId;
    var connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
   
    using (var connection = new SqlConnection(connectionString))
    using (var command = connection.CreateCommandO)
    {
      connection.Open(); 
      command.CommandText = "SELECT seq = (NEXT VALUE FOR UserSeq)"; 
      using (var reader = command.ExecuteReader())
      {
        if (reader.Read()) 
        {
          var rawSeqld = reader["seq"];
          seqId = rawSeqId.ToString();
        } 
        else 
        {
        	throw new Exception(); 
        }
      }
    }
    var id = new UserId(seqId);
    return new User(id, name); 
  }
}
```



인스턴스를 생성하는 처리를 팩토리로 옮겼으니 User 클래스를 만들려면 반드시 외부에서 Userld를 전달받아야 한다. 이로 인해Userld를 생성하던 User의 생성자 메서드가 불필요하게 됐다.

```c#
public class User
{
  private readonly UserId id; 
  private UserName name;
  
  public User(UserId id, UserName name)
  {
    if (id == null) throw new ArgumentNullException(nameof(id));
    if (name == null) throw new ArgumentNullException(nameof(name));
    
    this.id = id;
    this.name = name; 
  }
  (...생략...) 
}
```

이제 User 클래스의 생성자 메서드에서 데이터베이스에 접속하는 코드가 사라졌다.



그리고 팩토리를 사용하면 UserApplicationService의 사용자 등록 처리 역시 인스턴스 생성을 팩토리에 위임하게 된다.

```c#
public class UserApplicationService {
  private readonly IUserFactory userFactory; 
  private readonly IUserRepository userRepository; 
  private readonly UserService userService;
  
  (...생략...)
  
  public void Register(UserRegisterCommand command) 
  {
    var userName = new UserName(command.Name); 
    // 팩토리를 이용해 인스턴스를 생성
    var user = userFactory.Create(userName);
    
    if (userService.Exists(user)) 
    {
    	throw new CanNotRegisterUserException(user); 
    }
    
    userRepository.Save(user); 
  }
}
```



Register 메서드를 테스트할 때는 데이터베이스 없이 인메모리로 동작하게 하면 좋을 것 같다.

다음과 같이 팩토리를 구현한다.

```c#
class InMemoryUserFactory : IUserFactory
{
  // 마지막으로 발행된 식별자 
  private int currentId;
  public User Create(UserName name) 
  {
    // 사용자를 생성할 때마다 1씩 층가 
    currentld++;
    return new User(
    	new Userld(currentId.ToString()), 
      name
    );
  }
}
```

이 팩토리를 의존 관계 해소 설정에 포함시키면 테스트를 수행할 수 있다.

> 팩토리를 만들었으니 앞으로는 객체를 생성할 때 팩토리를 이용할거라고 생각할 것이다. 
>
> 그러나 User 클래스의 코드만 봐서는 팩토리의 존재감을 느끼기 어렵다.
>
> public class User
> {
>     // 생성자 메서드가 있다는 것만 알 수 있다 
>     public User(Userld id, UserName name); 
>     (...생략...)
> }
>
> 팩토리의 존재감을 좀 더 확실하게 느끼게 할 수 있는 방법이 있다. 
>
> 먼저 패키지를 다음과 같이 구상한다. 
>
> SnsDomain.Models.Users.User 
> SnsDomain.Models.Users.IUserFactory
>
> 나중에 참여한 개발자가 SnsDomain.Models.Users패키지를 열어보면 User와 lUserFactory클래스가 함께 있는 것을 보게 될 것이다.



### 9.2.1 자동 번호 매기기 기능 활용하기

SQL Server를 예로 들면, IDENTITY 속성을 적용한 칼럼은 레코드가 추가될 때마다 자동으로 번호가 매겨진다.

![image-20220208080758366](images/image-20220208080758366.png)

자동 번호 매기기는 데이터베이스에 객체를 저장할 때 ID가 부여된다. 그러므로 객체가 최초 생성될 때는 ID가 없는 상태로 생성된다. 

그리고 객체가 데이터베이스에 저장될 때 ID가 생성 되므로 식별자에 대한 세터가 필요하다. 이러한 요소는 객체를 불안정하게 만 든다.

```c#
public class User {
  private UserName name;
  public User(UserName name)
  {
  	this.name = name;
  }
  public UserId Id { get; set; } 
}
```



엔티티는 식별자를 통해 식별되는 객체다. 객체를 데이터베이스에 저장할 때까지 이 식별자가 부여되지 않는다는 것은 자연스럽지도 않고 범해서는 안 되는 금기사항이다. 식별자가 부여되지 않은 상태에서 객체를 잘못 다루면 의도하지 않은 동작을 보일 것이다. 



주의할 점은 또 있다. 

식별자에 대한 세터가 있다는 점이다. 위의 User 클래스의 Id 속성에 대한 세터는 리포지토리에서만 식별자를 다룬다는 것을 전제로 만들어진 것이다. 그러나 코드만으로는 이런 규칙을 알 수 없으니 앞뒤 사정을 모르는 개발자가 식별자를 바꾸는 코드를 작성할 가능성은 언제나 존재한다.



### 9.2.2 번호 매기기 메서드를 리포지토리에 두기

리포지토리에 번호 매기기 메서드를 두는 패턴도 가능하다.

```c#
public interface IUserRepository
{
	User Find(UserId id); 
  void Save(User user); 
  UserId NextIdentity();
}
```

Nextldentity 메서드는 새로운 생성자 Userld를 생성한다. 



이 메서드를 사용하면 UserApplicationService의 register 메서드는 다음과 같이 변경된다.

```c#
public class UserApplicationService {
  private readonly IUserRepository userRepository;
  (...생략...)
  
  public void Register(UserRegisterCommand command) 
  {
    var userName = new UserName(command.Name); 
    var user = new User(
      userRepository.Nextldentity(), 
      userName
    );
    (...생략...)
  }
}
```

리포지토리에 번호매기기 메서드를 두면 팩토리를 만들 정도로 수고롭지는 않으면서 식별자가 없는 불안정한 엔티티의 존재를 허용하지도 않는다.



하지만 번호 매기기와 객체를 저장하기 위한 기술이 서로 어긋나게 되는 경우 자연스럽지 않게 된다.

```c#
public class UserRepository : IUserRepository {
  private readonly NumberingApi numberingApi; 
  (...생략...)
  
  // 객체 저장에 관계형 데이터베이스를 사용하지만 
  public User Find(Userld id)
    {
    var connectionString =
     ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString; 
    using (var connection = new SqlConnection(connectionString))
    using (var command = connection.CreateCommand())
    {
      connection.Open();
      command.CommandText = "SELECT * FROM users WHERE id = @id"; 
      command.Parameters.Add(new SqlParameter("@id", id.Value)); 
      using (var reader = command.ExecuteReader())
      {
      	if(reader.Read())
        {
          var name = reader["name"] as string; 
          return new User(
            id,
            new UserName(name)
          );
        }
        else 
        {
        	return null;
        } 
      }
    } 
  }
  
  // 자동 번호 매기기에는 다른 기술을 사용함
  public Userld Nextldentity()
  {
  	var response = numberingApi.Request(); 
    return new Userld(response.NextId);
  }
}
```

한 클래스의 정의 안에 여러가지 기술 기반이 섞여 사용되고 있다.

리포지토리는 애초 객체를 저장하고 복원하기 위한 객체라는 점에서 번호매기기까지 리포지토리의 책임으로 확장하는 것은 좀 지나치다는 생각이다.



## 9.3 팩토리 역할을 하는 메서드

클래스 자체가 아닌 메서드가 팩토리 역할을 하는 경우도 있다. 이런 방법은 객체 내부의 데이터를 이용해 인스턴스를 생성할 필요가 있을 때 흔히 쓰인다.



서클 기능을 예로 들어보자. 서클은 동아리나 팀처럼 사용자가 소속되어 취미 등에 대한 의견을 나누는 모임이다. 서클에는 서클장의 역할을 맡는 사용자가 있다. 어떤 서클의 서클장이 누구인지에 대한 정보를 그 사용자의 사용자ID를 통해 나타낸다고 하자. 

```c#
var circle = new Circle(
	user.Id, // 게터를 통해 사용자 ID를 받아옴 
  new CircleName("my circle")
),
```

서클장의 사용자 ID를 Circle 객체에 전달하려면 게터를 이용해야 한다. 게터는 마냥 편하게 사용할 만한 메서드가 아니다 (12장에서)

내부 정보를 사용하지만 외부로 공개하지 않는 것은 메서드로 인스턴스를 생성하고 반환 값으로 주는 방법으로 가능하다.

```c#
public class User
{
  // 외부로 공개하지 않아도 된다
  private readonly Userld id; 
  (...생략...)
  
  // 팩토리 역할을 하는 메서드
  public Circle CreateCircle(CircleName circleName) 
  {
  	return new Circle( 
      id,
      circleName
    );
	}
}
```

이 방법이 옳은 것인지는 도메인을 어떤 관점에서 모델링 했는지에 따라 달라진다. 사용자가 서클을 생성하는 것을 도메인 객체의 행위로 정의해야 한다면 이를 정당화 할 수 있다.



## 9.4 복잡한 객체 생성 절차를 캡슐화하기

다형성 장점을 누릴 수 있게 팩토리를 만들 수 있지만, 이와 달리 단순히 생성 절차가 복잡한 인스턴스를 만드는 코드를 모아둔 팩토리를 만드는 것도 좋은 습관이다.



원래대로라면 객체의 초기화는 생성자 메서드의 역할이지만, 생성자 메서드의 단순함을 유지하기 위해 팩토리를 정의할 수 있다.



'생성자 메서드 안에서 다른 객체를 생성하는가' 라는 질문은 팩토리의 필요성을 나타내는 좋은 지표라고 할 수 있다. 만약 생성자 메서드가 다른 객체를 생성하고 있다면, 이 객체가 변경되었을 때 생성자도 같이 변경될 우려가 있다.

물론 모든 인스턴스를 팩토리에서 만들어야하는 것이 아닌, '그냥 하던대로 객체를 생성하지 말고, 팩토리가 필요하지는 않은가 검토하는 습관을 들이자’는 것이다.

> 팩토리는 도메인에서 유래한 객체가 아니다. 이 점은 리포지토리와 같다. 그렇다면 팩토리나 리포지토리는 도메인과는 무관한 존재일까? 그렇지는 않다.
>
> 객체의 생성은 도메인에서 유래한 것은 아니지만, 도메인을 표현하기 위해 필요한 요소다. 도메인을 표현하는데 도움을 주는 팩토리와 리포지토리 등의 요소는 도메인 설계를 구성하는 요소가 된다.
>
> 도메인을 모델에 녹여내고 코드로 다시 모델을 표현하는 과정인 도메인 설계를 완성하려면 도메인 모델을 표현하지 않는 요소도 필요하다는 점을 알아두자.



## 9.5 정리

팩토리는 객체의 생애 주기의 시작 단계에서 자신의 역할을 수행한다.

팩토리를 통해 생성 절차가 복잡한 객체를 생성하면 코드의 의도를 더 분명히 드러낼 수 있으며, 똑같은 객체 생성 코드가 이곳저곳 중복되는 것도 막을 수 있다.

팩토리를 이용해 객체 생성 절차를 캡슐화하는 것도 로직의 의도를 더 명확히 드러내면서 유연성을 확보할 수 있는 좋은 방법이다.