# 제어의 역전(IOC) 
 
IoC(제어의 역전)이라는 용어가 있다.     
객체의 생성과 생명 주기 관리를 프레임워크에게 맡긴다는 개념이다.  

## 오브젝트 팩토리  

성격이 다른 책임이나 관심사는 분리해버리는 것이 지금까지 해왔던 주요 작업이다.   
이제는 객체의 생성과 관련된 책임을 떠 맡는 팩토리에 대해서 알아보자.   
  
### 팩토리   

팩토리 클래스의 역할은 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 것이다.   
팩토리는 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 깔끔하게 분리하는 목적으로 사용한다.  
(디자인 패턴에서 말하는 팩토리와는 별개의 개념이므로 혼동하지 말자)   

```java
public class SimpleConnectionMakerFactory {
    public static SimpleConnectionMaker factory(final String dbName) {
        return switch(dbName) {
          case "PostgreSql" -> new NConnectionMaker();
          case "MySql" -> new DConnectionMaker();
          default -> new DefaultConnectionMaker();
        } 
    }  
}
```

```java
pubilc class Main {
    public static void main(String[] args) {
        final UserDao userdao = new UserDao(SimpleConnectionMakerFactory.factory("MySql"));  
    } 
}
```
 
팩토리 클래스가 생성이라는 책임을 담당하게 되었고 이를 통해 객체 사용을 더욱 편라히게 하고 있음을 볼 수 있다.    

### 설계도로서의 팩토리  

![image (5)](https://user-images.githubusercontent.com/50267433/162201533-83c2e57e-2cb3-4ba4-a76c-e81780963ccc.png)

이렇게 분리된 오브젝트들의 역할과 관계를 분석해보자.(명확한 이해를 위해 예제와는 다른 클래스를 만들었다)      
  
* UserDao: 데이터 로직에 대한 책임
* SimpleConnectionMaker: DB 연결 기술에 대한 책임
* SimpleConnectionMakerFactory: 오브젝트를 구성하고 관계를 정의하는 책임
* UserDaoTest: 동작을 테스트하는 책임
  
생성과 관련된 부분만 다른 클래스로 바꾼다면 다른 관계들의 변화없이 유지하며서 기능을 바꿀 수 있다.    
Factory를 분리함으로써 얻을 수 있는 장ㅈ머은 매우 다양하다.     
그 중에서도 애플리케이션의 컴포넌트 역할을 하는 오브젝트와 애플리케이션 구조를 결정하는 오브젝트 분리했다는데 가장 큰 의미가 있다.  

## 오브젝트 팩토리 활용  

**properties**
```properties 
common.dbName=PostgreSql
```

**Main**
```java
pubilc class Main {
    
    private String dbName;
    
    public Main(@Value("common.dbName") final String dbName) {
        this.dbName = dbName;
    }
      
    public static void main(String[] args) {
        final UserDao userdao = new UserDao(SimpleConnectionMakerFactory.factory(dbName));  
    } 
}
```

위와 같은 구조로 코드를 구현한다면 


**properties**
```properties 
common.dbName=MySql
``` 

다음과 같은 형식으로 기능을 교체하기 편해진다.      
이것이 오브젝트 팩토리를 사용하는 핵심 이유이다.       
 
## 제어권의 이전을 통한 제어관계 역전 

