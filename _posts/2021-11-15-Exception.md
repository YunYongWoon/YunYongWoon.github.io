---
layout: post
title: "예외(Exception)?"
subtitle: "예외란 무엇이고, 어떻게 해결하는 것인가"
date: 2021-11-15 22:40:00 +0900
background: '/img/posts/04.jpg'
categories: ['java']
---

## 예외(Exception) 란?
- 프로그램 실행 시 의도하지 않게 발생하는 문제점
- 개발자가 어느정도 대처할 수 있는 문제들이다. 대처할 수 없는 문제들은 에러(Error) 라고 부른다.
- JAVA에서는 java.lang.Exception클래스와 그 서브클래스들로 정의된다.
![](https://images.velog.io/images/yuyun0124/post/00b6047b-dbb7-40fd-a4c2-c0608653da60/%E1%84%8B%E1%85%A8%E1%84%8B%E1%85%AC.png)

## 체크예외와 언체크예외
- Exception클래스는 체크예외와 언체크예외로 나뉘어진다.

### 체크예외
- Exception을 상속하는 예외들 중, RuntimeException을 상속하지 않는 예외들.
- 체크 예외가 발생할 수 있는 메소드를 사용하는 경우, 반드시 예외를 처리하는 코드를 함께 작성해야 한다.
	- try~catch로 잡는 방법
	- throws 정의하여 메소드 밖으로 던지는 방법
    
### 언체크/런타임 예외
- java.lang.RuntimeException 클래스를 상속한 예외들은 명시적인 예외처리를 강제하지 않기 때문에 언체크 예외(Unchecked Exception)라고 부른다.
- try~catch, throws로 선언해줘도, 혹은 선언해주지 않아도 된다.
- 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들.
	- 오브젝트를 할당하지 않은 레퍼런스 변수를 사용하려고 시도했을 때 발생하는 NullPointerException
   	- 허용되지 않는 값을 사용해서 메소드를 호출할 때 발생하는 IllegalArgumentException
  	- 기타등등.....
    
## 예외 처리 방법
### 예외 복구
- 예외상황을 파악하고, 문제를 해결하여 정상 상태로 돌려놓는 것.
- 코드 
```JAVA
int maxretry = MAX_RETRY
while(maxretry-- > 0){
    try{
        ...		// 예외가 발생할 가능성이 있는 시도
        return	// 작업 성공
    } catch(SomeException e) {
        // 로그 출력, 정해진 시간만큼 대기
    } finally {
        // 리소스 반납. 정리 작업
    }    
}
throw new RetryFailedException(); // 최대 시도 횟수를 넘기면 직접예외 발생
```
- 예외처리 코드를 강제하는 체크 예외들은 이렇게 예외를 어떤 식으로든 복구할 가능성이 있는 경우 사용.

### 예외 회피
- 예외처리를 자신이 담당하지 않고, 자신을 호출한 쪾으로 던져버리는 것.
- throws문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나, catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 방법을 사용한다.
```JAVA
// 회피방법 1
public void add1() throws SQLException{
    // JDBC API
}
// 회피방법 2
public void add2() throws SQLException{
    try{
        // JDBC API
    } catch(SQLException e) {
        // 로그 출력
        throw e;
    }
}
```

### 예외 전환
- 예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로 만들 수 없기 때문에 예외를 메소드를 밖으로 던지는 것이다.
- 하지만 예외 회피와는 다르게, 발생한 예외를 적절한 예외로 전환해서 던진다.
- 내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우
  - 의미를 분명하게 해줄 수 있는 예외로 바꿔줄 수 있다.
  - 다음 코드에서는, 아이디가 중복 시 단순 SQLException이 아닌 DuplicateUserIdException을 던져줄 수 있다.
```JAVA
    public void  add(User user) throws DuplicateUserIdException,SQLException{
        try{
            // JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
            // 그런 기능을 가진 다른 SQLException을 던지는메소드를 호출하는 코드
        } catch(SQLException e){
            // ErrorCode가 MySQL의 "Duplicate Entry(1062)" 이면예외 전환
            if(e.getErrorCode() == MysqlErrorNumbersER_DUP_ENTRY)
                throw DuplicateUserIdException();
            else
                throw e; // 그 이외의 경우는 SQLException 그대로
        }
    }
```
- 보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외로 만드는 것이 좋다.
  - 중첩 예외는 getCause() 메소드를 사용하여 처음 발생한 예외가 무엇인지 확인 할 수 있다. 
 
```JAVA
    // 중첩 예외 1
    catch(SQLException e){
        ...
        throw DuplicateUserIdException(e);
    }
    // 중첩 예외 2
    catch(SQLException e){
        ...
        throw DuplicateUserIdException().initCause(e); // 근본원인이 되는 메소드를 넣어준다.
    }
```
- 예외를 처리하기 쉽고 단순하게 만들기 위해 포장
	- 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우 사용한다.
```JAVA
    try{
        OrederHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
        Order order = orderHome.findByPrimaryKey(Integer id);
    } catch (NamingException ne) {
        throw new EJBException(ne)
    } catch (SQLException se) {
        throw new EJBException(se)
    } catch (RemoteException re) {
        throw new EJBException(re)
    } 
```
- 체크예외를 계속 throws를 사용하여 넘기는 것은 메소드 선언이 지저분해지고, 아무런 장점이 없다. 복구 불가능한 예외라면, 가능한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws 선언이 들어가지 않게 해 줘야 한다.


   	