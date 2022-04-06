# 초난감 DAO
## 엔티티 
```java
public class User {
    String id;
    String name;
    String password;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
## DAO 
```java
public class UserDao {

    public void add(User user) throws SQLException, ClassNotFoundException {
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

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
        Class.forName("org.postgresql.Driver");

        String user = "postgres";
        String password = "password";

        Connection c = DriverManager.getConnection(
                "jdbc:postgresql://localhost/toby_spring"
                , user
                , password
        );

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
    
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        UserDao dao = new UserDao();

        User user = new User();
        user.setId("1");
        user.setName("제이크");
        user.setPassword("jakejake");

        dao.add(user);

        System.out.println(user.getId() + " register succeeded");

        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassword());

        System.out.println(user2.getId() + " query succeeded");
    }
}
```

## 문제점 

일단 UserDao 클래스 코드의 문제점은 무엇일지 스스로 생각해보자.   
사실 기능적인 문제점은 존재하지 않지만 아래와 같은 질문들을 끊임없이 이어나가자.  
 
* DAO 코드를 개선했을 때의 장점은 무엇일까?      
* 장점들이 당장에, 또는 미래에 주는 유익은 무엇일까?    
* 객체지향 설계의 원칙과는 무슨 상관이 있을까?    
* 이 DAO를 개선하는 경우와 그대로 사용하는 경우, 스프링을 사용하는 개발에서 무슨 차이가 있을까?   
  
**스프링을 공부한다는 건 바로 이런 문제 제기와 의문에 대한 답을 찾아나가는 과정이다.**    
스프링은 기계적인 답변이나 성급한 결론을 주지 않는다.    
**최종 결론은 스프링을 이용해 개발자가 스스로 만들어 내는것이지, 스프링이 덥석 줄 수 있는 것은 아니다.**    
 
스프링은 단지 그 과정에서 어떤 고민을 제대로 하고 있는지 끊임없이 확인해주고    
좋은 결론을 내릴 수 있도록 객체지향 기술과 자바 개발의 선구자들이 먼저 고민하고 제안한 방법에 대한 힌트를 제공해줄 뿐이다.  
