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

제어의 역전은 프로그램의 제어 흐름 구조가 뒤바뀌는 것이라고 설명할 수 있다.       
기존에는 UserDao 직접 객체를 생성하고 생명주기에 맞추어 관리를 했어야 했다.    
하지만, Factory 가 도입됨으로써 생성에 대한 책임은 사라졌고, 사용에 있어 조금 더 집중 할 수 있는 환경이 되었다.    
 
제어의 역전이란 Factory를 도입하는 것처럼 제어 흐름의 개념을 거꾸로 뒤집는 것이다.           
제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않으며 생성하지도 않는다.   

어떻게 만들어지고 어디서 사용되는지 알 수 없으며, 모든 제어 권한을 자신이 아닌 다른 대상에 위임하기 때문이다.    
그리고 모든 오브젝트는 이렇게 위임 받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어진다.     
사실 이러한 개념은 JSP. EJB와 같은 컨테이너를 사용하는 기술들에 이미 적용된 요소이다.   
또한, 제어의 역전은 프레임워크 뿐만 아니라 디자인 패턴에서도 적용될 수 있디.   

프레임워크는 이러한 제어의 역전 개념이 적용된 대표적인 기술이다.  

* **프레임워크는 라이브러리의 다른 이름이 아니다.**  
* **프레임워크는 단지 미리 만들어둔 반제품이나, 확장해서 사용할 수 있도록 준비된 추상 라이브러리의 집합이 아니다.**   
* 프레임워크가 어떤 것인지 이해하려면 라이브러리와 프레임워크가 어떻게 다른지 알아야한다.     
  
라이브러리를 사용하는 애플리케이션 코드는 애플리케이션 흐름을 직접 제어한다.      
단지 동작하는 중에 필요한 기능이 있을 때 능동적으로 라이브러리를 사용할 뿐이다.    
   
반면에 프레임워크는 거꾸로 애플리케이션 코드가 프레임워크에 의해 사용된다.      
보통 프레임워크 위에 개발한 클래스를 등록해두고, 프레임워크가 흐름을 주도하는 것 중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식이다.      
**프레임워크에는 제어의 역전이라는 개념이 적용되어 있어야한다**      
  
이전 예제에서는 자연스럽게 관심을 분리하고 책임을 나누고 유연하게 확장 가능한 구조로 만들기 위해 IoC를 도입했었다.        
IoC를 적용함으로써 설계가 깔끔해지고 유연성이 증가하며 확장성이 좋아지기 때문에 필요할 때면 IoC 스타일의 설계와 코드를 만들면 좋다.  
   
제어의 역전에서는 프레임워크 또는 컨테이너와 같이      
애플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요하다.      
스프링은 IoC 를 모든 기능의 기초가 되는 기반 기술로 삼고 있으며, IoC를 극한까지 적용하고 있는 프레임워크다.     
