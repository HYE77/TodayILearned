## **한 줄 정의**

> **Spring Bean은 Spring 컨테이너가 생성하고 관리하는 객체(Object)입니다.**
일반 Java 객체와의 차이점은 객체를 개발자가 직접 생성하는 것이 아니라, Spring이 생성하고 생명주기를 관리한다는 점입니다.
> 

---

# **왜 필요한가?**

일반 Java에서는 객체를 직접 생성합니다.

```java
MemberService memberService = new MemberService();
```

객체가 많아질수록

- 생성 책임
- 의존성 연결
- 객체 관리

를 모두 개발자가 해야 합니다.

---

Spring에서는

```java
@Service
public class MemberService {
}
```

Spring이 객체를 생성합니다.

```java
@Autowired
private MemberService memberService;
```

필요한 곳에 자동으로 주입해 줍니다.

---

# **Bean 등록 과정**

예를 들어

```java
@Service
public class MemberService {
}
```

애플리케이션 시작 시

```
Spring Application 시작
        ↓
@Component Scan 수행
        ↓
@Service 발견
        ↓
MemberService 객체 생성
        ↓
Spring Container 저장
        ↓
Bean 등록 완료
```

---

# **Spring Container란?**

Bean을 저장하고 관리하는 공간입니다.

```
Spring Container
 ├─ MemberService Bean
 ├─ OrderService Bean
 ├─ UserRepository Bean
 └─ ...
```

Bean은 모두 Spring Container 안에서 관리됩니다.

---

# **Bean 등록 방법**

## **1. 컴포넌트 스캔 방식 (가장 많이 사용)**

```java
@Component
public class TestService {
}
```

또는

```java
@Service
@Repository
@Controller
@RestController
```

모두 내부적으로 `@Component`를 포함합니다.

```
@Component
 ├─ @Service
 ├─ @Repository
 ├─ @Controller
 └─ @RestController
```

---

## **2. @Bean 사용**

직접 설정 클래스에 등록할 수도 있습니다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService();
    }
}
```

---

## **언제 @Bean을 사용할까?**

외부 라이브러리처럼 수정할 수 없는 객체를 등록할 때 사용합니다.

예)

```java
ObjectMapper
PasswordEncoder
RestTemplate
```

---

# **Bean의 생명주기**

```
1. 객체 생성
      ↓
2. 의존성 주입
      ↓
3. 초기화
      ↓
4. 사용
      ↓
5. 종료
```

Spring이 이 과정을 모두 관리합니다.

---

# **Bean Scope**

## **Singleton (기본)**

```java
@Service
public class MemberService {
}
```

하나의 객체만 생성

```
Spring Container
      ↓
MemberService (1개)
      ↓
모든 요청이 공유
```

가장 많이 사용됩니다.

---

## **Prototype**

```java
@Scope("prototype")
```

요청할 때마다 새 객체 생성

```
요청1 → 객체1

요청2 → 객체2

요청3 → 객체3
```

---

# **Bean과 DI 관계**

Bean이 존재해야 DI가 가능합니다.

예)

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

동작 과정

```
Spring Container

PaymentService Bean 생성
        ↓
OrderService Bean 생성
        ↓
PaymentService 주입
```

Bean으로 등록되지 않은 객체는 주입할 수 없습니다.

---

# **일반 객체와 Bean의 차이**

| **일반 객체** | **Spring Bean** |
| --- | --- |
| 개발자가 생성 | Spring이 생성 |
| 개발자가 관리 | Spring이 관리 |
| DI 불가능 | DI 가능 |
| 생명주기 직접 관리 | Spring이 관리 |
| 단순 객체 | 컨테이너 관리 객체 |

---

# **면접 답변 예시**

Spring Bean은 Spring 컨테이너가 생성하고 관리하는 객체입니다. 일반 객체와 달리 객체의 생성, 의존성 주입, 소멸 등의 생명주기를 Spring이 관리합니다. Bean은 `@Component`, `@Service`, `@Repository` 같은 어노테이션이나 `@Bean` 설정을 통해 등록할 수 있으며, Spring의 DI 기능은 Bean을 기반으로 동작합니다. 또한 기본적으로 Singleton Scope로 관리되어 하나의 객체를 여러 곳에서 공유하여 사용합니다.

---

# **꼬리 질문 대비**

### **Q. Bean과 객체(Object)의 차이는?**

- 객체는 단순히 생성된 인스턴스
- Bean은 Spring 컨테이너가 관리하는 객체

즉,

```
모든 Bean은 객체이다.
하지만 모든 객체가 Bean은 아니다.
```

---

### **Q. Bean은 기본적으로 어떤 Scope를 가지나요?**

**Singleton Scope**

애플리케이션 전체에서 하나의 객체만 생성하여 사용합니다.

---

### **Q. @Bean과 @Component의 차이는?**

| **@Component** | **@Bean** |
| --- | --- |
| 클래스에 붙임 | 메서드에 붙임 |
| 자동 등록 | 수동 등록 |
| 직접 작성한 클래스 | 외부 라이브러리 등록 시 자주 사용 |

---

### **Q. Bean이 없으면 DI가 가능한가요?**

불가능합니다.

Spring은 컨테이너에 등록된 Bean을 찾아 의존성을 주입하기 때문입니다.
