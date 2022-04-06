# DAO의 분리 
## 관심사의 분리 
### 변화에 어떻게 대응할 것인가 
세상에는 변하는 것과 변하지 않는 것이 있다.    
하지만, 객체지향의 세계에서는 모든 것이 변한다.(오브젝트에 대한 설계와 이를 구현한 코드)    
 
소프트웨어 개발에서 끝이라는 개념은 없다.    
사용자의 비즈니스 프로세스와 그에 따른 요구사항은 끊임없이 바뀌고 발전한다.   
**그래서 개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 바로 미래의 변화를 어떻게 대비할 것인가이다.**     

### 변화에서의 관심사

지혜로운 개발자는 오늘 이 시간에 미래를 위해 설계하고 개발한다.     
그리고 그 덕분에 미래에 닥칠지도 모르는 거대한 작업에 대한 부담과     
변경에 따른 엄청난 스트레스, 그로인해 발생하는 고객과의 사이에서 또 개발팀 내에서의 갈등을 최소화할 수 있다. 
  
객체지향 설계와 프로그래밍이 이전의 절차적 프로그래밍 패러다임에 비해 초기에 좀 더 많은, 번거로운 작업을 요구하는 이유는       
객체지향 기술 자체가 가지는, 변화에 효과적으로 대체할 수 있다는 기술적인 특징 때문이다.     
  
객체지향은 흔히 실세계를 최대한 가깝게 모델링 한다고 하지만  
가상의 추상세계 자체를 효과적으로 구성할 수 있고, 이를 자유롭고 편리하게 변경, 발전, 확장시킬 수 있는데 의미가 있다.   
 
미래를 준비하는데 가장 중요헌 과제는 변화에 어떻게 대비할 것인가이다.    
그러면 어떻게 변경이 일어날 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않게 할 수 있었을까?     
그것은 바로 **분리와 확장**을 고려한 설계가 있었기 때문이다.  

**모든 변경과 발전은 한번에 한가지 관심사항에 집중해서 일어난다**   
그러나 문제는, **변화는 대체로 집중된 한가지 관심에 대해 일어나지만 그에 따른 작업은 한 곳에 집중되지 않는 경우가 많다는 점이다.**   
변화가 한번에 한가지 관심에 집중되서 일어난다면, **우리가 준비해야할 일은 한가지 관심이 한 군데에 집중되게 하는 것이다.**     
**즉, 관심이 같은 것끼리는 모으고, 관심이 다른 것은 따로 떨어져 있게 하는 것이다.**   

### 관심사 분리의 핵심  

프로그래밍의 기초 개념중에 **관심사의 분리**라는 게 있다.      
**관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고       
관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것이라 생각할 수 있다.**     
   
모든 것을 뭉뚱 그려서 한데 모으는 편이 처음엔 쉽고 편리하다.     
그 뭉쳐 있는 여러 종류의 관심사를 적절하게 구분하고 따로 분리하는 작업을 해줘야만 할 때가 온다.      
관심사가 같은 것끼리 모으고 다른 것은 분리해줌으로써 같은 관심에 효과적으로 집중할 수 있게 만들어주는 것이다.   

## 커넥션 만들기의 추출 

```java
    public void add(User user) throws SQLException, ClassNotFoundException {
    	   // 1. 커넥션 가져오기
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );


        // 2. SQL 문장을 담을 Statement를 만들고 실행하기
        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values (?, ?, ?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();


        // 3. 리소스 반납하기
        ps.close();
        c.close();
    }
```   
총 3가지의 관심사가 나온다.

* DB와 연결하기 위한 커넥션을 가져오는 것
* DB에 보낼 SQL 문장을 담을 Statement를 만들고 실행하는 것
* 공유 리소스를 시스템에 돌려주는 것

### 중복 코드의 메서드 추출 
     
`DB와 연결하기 위한 커넥션을 가져오는 것`과 관련된 코드들은 모든 메서드에 존재할 것이다.         
**만약 이런 메서드가 100개, 1000개가 넘게 있으며 이들을 유지보수 관리해야 할 일이 생긴다면?**        
즉, 이렇게 중복된 관심사 코드가 발생하면, **변경이 일어날 때 엄청난 고통을 일으킬 수 있으므로 중복된 코드를 메소드로 추출하자.**  

```java
    public void add(User user) throws SQLException, ClassNotFoundException {
        // 1.2.2 중복 코드의 메소드 추출
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values (?, ?, ?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
    
    public User get(String id) throws SQLException, ClassNotFoundException {
        // 1.2.2 중복 코드의 메소드 추출
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?"
        );
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
    
    // 커넥션 가져오기 관심사
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );
    }
```
    
이제 로그인 정보가 변경되더라도 `getConnection()` 이라는 한 메서드의 코드만 수정하면 된다.          
관심의 종류에 따라 코드를 구부해놓았기 때문에       
**한 가지 관심에 대한 변경이 일어날 경우 그 관심이 집중되는 부분의 코드만 수정하면 된다.**      
관심이 다른 코드가 있는 메서드에는 영향을 주지 않을뿐더러, 관심 내용이 독립적으로 존재하므로 수정도 간단해졌다.   

### 변경 사항에 대한 검증: 리팩토링과 테스트 
 
중요한 변화가 있었다.      
앞선 작업은 중복돼서 특정 관심사항이 담긴 코드를 별도의 메서드로 분리해낸 것이다.      
기능이 추가되거나 바뀐것은 없지만 이전보다 훨씬 깔끔해졌고 변화에 대응할 수 있는 코드가 되었다.      
이런 작업을 **리팩토링**이라고 부른다.    
   
그리고 위의 `getConnection()`과 같이 공통의 기능을      
메서드로 중복된 코드를 뽑아내는 것을 리팩토링에서는 **메서드 추출**이라고 한다.      

## DB 커넥션 만들기의 독립 

아주 초보적인 관심사의 분리 작업이지만, 메서드 추출만으로도 변화에 좀 더 유연하게 대처할 수 있는 코드를 만들었다.  
이번에 좀 더 나아가서 변화에 대응하는 수준이 아니라, 아에 변화를 반기는 DAO를 만들어 보자.     
  
### 상속을 통한 확장 

![image](https://user-images.githubusercontent.com/50267433/161970186-7057201d-0d07-4b10-adf8-ec5268146f3b.png)

**UserDao - abstract**
```java
public abstract class UserDao {
    public void add(User user) throws SQLException, ClassNotFoundException {
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values (?, ?, ?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws SQLException, ClassNotFoundException {
        Connection c = getConnection();

        PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?"
        );
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }

    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```

UserDao 를 추상 클래스로 두었다.    
   
**NUserDao - Concrete**
```java
public class NUserDao extends UserDao{
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

        return c;
    }
}
```

**DUserDao - Concrete**

```java
public class DUserDao extends UserDao {
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

        return c;
    }
}
```

관시사를 분리한 영역만 별도의 클래스로 빼서 이를 구현하도록 했다.       
물론 위 같은 코드는 인터페이스로 분리하고 컴포지션을 사용할 수도 있다.     
우리는 상속과 구현을 통해 각각의 요청에 맞는 Dao를 불러다 사용만 하면 된다.      
더불어, 새로운 요구사항이 들어오면 이에 맞는 Dao를 구현해도 된다.     
  
이렇게 슈퍼 클래스에서 기본적인 로직의 흐름을 만들고,    
그 기능의 일부를 추상 메서드나 오버라이딩이 가능한 protected 메서드등으로 만든 뒤     
서브 클래스에서 이런 메서드를 필요에 맞게 구현해서 사용하도록 하는 방법을 디자인 패턴에서 **템플릿 메서드 패턴**이라고 한다.  









