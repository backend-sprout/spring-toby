# DAO의 확장 

모든 오브젝트들은 변하기 마련이지만, 관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화의 특징이 있다.    
즉, 앞선 관심사 2개 `데이터 액세스 로직`, DB 연결`의 변화의 성격이 다르다는 의미다.  

* 변화의 성격이 다르다 == 변화의 이유와 시기, 주기 등이 다르다.     

```
데이터 액세스 로직을 JDBC/JPA/MyBatis 와 같은 요소들을 이용해 바꿀수 있다.     
하지만, DataSource 라는 DB 연결은 변하지 않는다.      
반대로, DataSource 외에 다른 라이브러리로 DB 연결을 바꾸더라도 데이터 액세스 로직은 바뀌지 않는다.  
```
   
추상 클래스를 만들고 이를 상속한 서브클래스에서 변화가 필요한 부분을 바꿔서 쓸 수 있게 만든 이유는       
**변화의 성격이 다른 것을 분리해서 서로 영향을 주지 않은채로 각각 필요한 시점에 독립적으로 변경하기 위해서이다.**   

## 클래스의 분리 

관심사가 다르고 변화의 성격이 다른 요소를 화끈하게 분리해볼 생각이다.     
관심사를 본격적으로 독립시키면서 동시에 손쉽게 확장하는 방법에 대해서 이야기하고자 한다.    

**상속 관계가 아닌, 완전히 독립적인 클래스로 만들어보자**   

![image (2)](https://user-images.githubusercontent.com/50267433/162185789-2964f391-86e9-4ae4-a748-de3d482ade81.png)
    
우선은 인터페이스를 생각하지 않고 별도의 하드한 클래스로만 만든 구조이다.    
개인적으로 변화의 폭이 적으면, 인터페이스와 추상 클래스와 같이 상위 추상 컴포넌트를 굳이 만들지 않아도 된다고 생각한다.     
반대로, **변화의 폭이 많고 유지보수, 확장의 가능성이 있는 관심사의 경우 상위 추상 컴포넌트를 만드는 것을 추천한다.**   

```java
public class UserDao {

    private final SimpleConnectionMaker simpleConnectionMaker;

    public UserDao() {
        this.simpleConnectionMaker = new SimpleConnectionMaker();
    }

    public void add(User user) throws SQLException, ClassNotFoundException {
        Connection c = simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values (?, ?, ?)");
        
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
    
    ...
}
```
 
위와 같은 방법이 컴포지션 방법이다.        
상속구조를 없애고 조합하는 형식이기에 상속의 단점을 해소할 수 있다.        
단, 위와 같은 구조에도 몇가지 문제점이 있는데 차근 차근 알아가보자.   

## 인터페이스 도입 

![image (3)](https://user-images.githubusercontent.com/50267433/162186723-9fb4734b-e373-4764-9abc-b4479351c834.png)

**ConnectionMaker.interface**
```java
@Functional
public interface SimpleConnectionMaker {
    Connection makeNewConnection() throws ClassNotFoundException, SQLException;
}
```
* 상위 추상 컴포넌트인 인터페이스를 정의한다.   

**NConnectionMaker.class**
```java
public class NConnectionMaker implements SimpleConnectionMaker {

    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        return DriverManager.getConnection(
            "jdbc:postgresql://localhost/toby_spring", user, password
        );
    }
}
```
* 상위 추상 컴포넌트를 구현한 하위 구현체 컴포넌트를 정의한다.        

```java
public class UserDao {

    private final SimpleConnectionMaker connectMakger;

    public UserDao() {
        this.connectMakger = new NConnectionMaker();
    }

    public void add(User user) throws SQLException, ClassNotFoundException {
        Connection c = simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values (?, ?, ?)");
        
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
    
    ...
}
```

* UserDao 는 동일한 영역의 컴포넌트를 바라본다.     
* SimpleConnectionMaker 은 상위 컴포넌트를 바라본다.      
* 위와 같이 의존성이 상->하 가 아닌, 하->상의 구조를 DIP(의존 관계 역전의 원칙)라 부른다.  

**단, 현재 구조에 문제점이 있는데 바로 다른 하위 구현체를 바꾸고자 한다면 UserDAO를 수정해야한다.**      
언뜻 누군가는 문제 없다고 생각할 수 있지만, 앞서 언급했듯이 서로 다른 관심사는 영향을 받으면 안된다.          
  
**DConnectionMaker.class**
```java
public class DConnectionMaker implements SimpleConnectionMaker {

    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("org.mysql.Driver");

        String user = "mysql";
        String password = "password";

        return DriverManager.getConnection(
            "jdbc:postgresql://localhost/toby_spring", user, password
        );
    }
}
```
만약, 위 코드와 마찬가지로 다른 하위 구현체 클래스를 만들었다고 가정하자.   
그러면 UserDao 는 이를 반영하기 위해 다음과 같이 코드를 수정해야한다.  

```java
    public UserDao() {
        this.connectMakger = new DConnectionMaker();
    }

```

**다른 기능을 사용하기 위해서 UserDao 의 코드를 수정하는건 말이 안 되는 이야기이다.**         
이를 해결할 수 있는 방법이 여러가지가 있는데 다음 단락에서 확인하자.      
     
## 관계 설정 책임의 분리 
 
객체 생성을 담당하는 것은 하나의 책임을 가지는 것이다.    
구체적인 클래스 타입을 UserDAO가 알게 되는 것은 물론, 해당 타입 구현체를 생성하고 사용하는 2가지 책임을 한번에 가지는 것이다.   
**만약, 다른 하위 클래스 타입을 사용하고자 한다면 UserDAO의 코드가 변동되어야 한다는 뜻이다.**        

![image (4)](https://user-images.githubusercontent.com/50267433/162191143-957be917-ff31-4a7b-bf80-9665674d6247.png)

이를 해결하는 방법은 아주 간단한다.       
**객체 생성을 담당하는 책임을 다른 클래스에 넘기면 된다.**     

```java

public class Main {
   public static void main(String[] args) {
      final UserDaoOne = new UserDao(new NConnectionMaker()); 
      final UserDaoTwo = new UserDao(new DConnectionMaker());    
   } 
}
```

**UseDao**
```java
public class UserDao {

    private final SimpleConnectionMaker connectMakger;

    public UserDao(final ConnectionMaker connectMakger) {
        this.connectMakger = connectMakger;
    }

    public void add(User user) throws SQLException, ClassNotFoundException {
        Connection c = simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values (?, ?, ?)");
        
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
    
    ...
}
```

위와 같이 구조로 표현하면, 주입받는 클래스의 구체적인 타입을 UserDao 가 알지 못하도록 할 수 있다.     
즉, 구현체가 무엇이든 UserDAO는 관리할 책임이 없으며 단지 메서드를 호출하는 형태로 사용하기만 하면 된다.  
   
그런데, 이렇게 다른 클래스에 책임을 넘겨도 되는 것일까?         
엄밀히 말하면 우후죽순으로 다른 클래스에 책임을 넘기는 것은 별로 좋지 않다.    
    
[클린 코드](https://github.com/kwj1270/TIL_CleanCode/blob/master/11%20%EC%8B%9C%EC%8A%A4%ED%85%9C.md#%ED%8C%A9%ED%86%A0%EB%A6%AC)에서 언급하기를,   
객체 생성을 담당하는 별도의 요소에게 책임을 맡기고, 해당 객체를 주입받아 사용하는 것을 추천한다.          
이와 같은 구조로 가져갈 경우, 객체 사용에 대한 책임만 지게 되어 변경의 여지를 줄일 수 있고 테스트에 용이해진다는 장점이 있다.        

## 원칙과 패턴 
 
앞선 예제에서 적용한 원칙과 패턴에 대해서 정리하고자 한다.   

### OCP(개발 폐쇄 원칙)
 
OCP(개발 폐쇄 원칙) 은 깔끔한 설계를 위해 적용 가능한 객체지향 설계 원칙 중의 하나다.(SOLID).       
`클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.`           
즉, 핵심 기능을 구현한 코드는 그런 변화에 영향을 받지 않고 유지할 수 있어야 한다는 의미이다.       

잘 설계된 객체지향 클래스의 구조를 살펴보면 바로 이 개방 폐쇄 원칙을 아주 잘 지키고 있다.    
인터페이스를 사용해 확장 기능을 정의한 대부분의 API는 바로 이 개발 폐쇄 원칙을 지키고 있다고 보면 된다.   
  
### 높은 응집도와 낮은 결합도  
      
소프트웨어의 고전적인 원리로도 설명이 가능하다.       
응집도가 높다는 것은 **하나의 모듈, 클래스가 하나의 책임 또는 관심사에 집중되어 있다는 뜻이다.**         
불필요하거나 직접 관련이 없는 외부의 관심과 책임이 얽혀 있지 않으며, 하나의 공통 관심사는 한 클래스에 모여 있다.     
**높은 응집도는 클래스 레벨뿐 아니라, 패키지, 컴포넌트, 모듈에 이르기가지 그 대상의 크기가 달라져도 동일한 원리로 적용될 수 있다.**     

#### 높은 응집도  

* 응집도가 높다 === 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다.   
    
변경이 일어날때 모듈의 많은 뿐이 함께 바꾸니다면 응집도가 높다고 말할 수 있다.          
만약 모듈의 일부분에만 변경이 일어나도 된다면,     
모듈 전체에서 어떤 부분이 바뀌어야 하는지 파악해야 하고,       
또 그 변경으로 인해 바뀌지 않는 부분에는 다른 영향을 미치지 않는지 확인해야하는 이중 부담이 생긴다.   

#### 낮은 결합도 
  
낮은 결합도는 높은 응집도보다 더 민감한 원칙이다.    
책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태(낮은 결합도)를 유지하는 것이 바람직하다.     
   
느슨한 연결은    
**관계를 유지하는데 필요한 최소한의 방법만 `간접적인 형태`로 제공하고 나머지는 서로 독립적이고 알 필요도 없게 만들어주는 것이다.**       
결합도가 낮아지면 변화에 대응하는 속도가 높아지고, 구성이 깔끔해지고 확장하기에도 매우 편리하다.   

### 전략 패턴  

```java
public class UserDao {

    private final SimpleConnectionMaker connectMakger;

    public UserDao(final ConnectionMaker connectMakger) {
        this.connectMakger = connectMakger;
    }

    public void add(User user) throws SQLException, ClassNotFoundException {
        Connection c = simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values (?, ?, ?)");
        
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
    
    ...
}
```

위 구조는 디자인패턴의 시각으로 보자면 `전략 패턴`에 해당한다고 볼 수 있다.    
   
전략 패턴은 자신의 기능 맥락에서   
필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고      
이를 구현한 구체적인 구현체의 알고리즘에 따라 바꿔서 사용할 수 있는 디자인 패턴을 의미한다.    
(알고리즘이란 것은 독립적으로 분리가 가능한 기능을 의미한다.)    
 
NConnectionMaker 는 postgreSql 방식이지만, DConnectionMaker 는 mysql 방식이다.       
이렇듯 각각의 전략에 맞춰서 기능을 달리 실행하고자 할 때 주로 사용한다.   
