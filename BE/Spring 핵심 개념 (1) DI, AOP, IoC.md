<aside>

> Spring의 핵심 개념을 이야기할 때 가장 많이 등장하는 개념이 **IoC(Inversion of Control)**, **DI(Dependency Injection)**, **AOP(Aspect-Oriented Programming)** 입니다.

</aside>

---

# **1. IoC (Inversion of Control, 제어의 역전)**

## **개념**

객체의 생성과 관리에 대한 제어권을 개발자가 직접 가지는 것이 아니라 **Spring 컨테이너가 대신 관리하는 것**을 의미합니다.

기존 Java에서는 개발자가 직접 객체를 생성합니다.

```java
MemberService service = new MemberService();
```

하지만 Spring에서는 객체 생성과 생명주기를 컨테이너가 담당합니다.

```java
@Autowired
private MemberService service;
```

개발자는 객체를 생성하지 않고 사용만 합니다.

---

## **왜 필요한가?**

기존 방식

```
개발자
 ├─ 객체 생성
 ├─ 객체 연결
 └─ 객체 관리
```

IoC 적용

```
개발자
 └─ 사용만

Spring Container
 ├─ 객체 생성
 ├─ 객체 연결
 └─ 객체 관리
```

객체 관리 책임을 프레임워크에 위임하여 코드 결합도를 낮출 수 있습니다.

---

## **장점**

- 객체 간 결합도 감소
- 유지보수 용이
- 테스트 편리
- 객체 생명주기 관리 가능

---

# **2. DI (Dependency Injection, 의존성 주입)**

## **개념**

DI는 IoC를 구현하는 대표적인 방법입니다.

객체가 필요한 의존 객체를 직접 생성하는 것이 아니라 외부(Spring)가 주입해주는 방식입니다.

---

## **의존성이란?**

```java
public class OrderService {
    private PaymentService paymentService
        = new PaymentService();
}
```

OrderService가 PaymentService를 사용하므로

```
OrderService → PaymentService
```

의존 관계가 존재합니다.

---

## **문제점**

```java
private PaymentService paymentService
        = new PaymentService();
```

객체를 직접 생성하면

- 강한 결합 발생
- 구현체 변경 어려움
- 테스트 어려움

---

## **DI 적용**

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring이 객체를 생성한 후 주입합니다.

```
Spring Container
      ↓
PaymentService
      ↓
OrderService
```

---

## **DI 종류**

### **1. 생성자 주입 (권장)**

```java
@RequiredArgsConstructor
@Service
public class OrderService {

    private final PaymentService paymentService;
}
```

특징

- 불변성 보장
- 순환 참조 방지
- 테스트 용이

실무에서 가장 많이 사용

---

### **2. Setter 주입**

```java
@Autowired
public void setPaymentService(
    PaymentService paymentService
) {
    this.paymentService = paymentService;
}
```

---

### **3. 필드 주입**

```java
@Autowired
private PaymentService paymentService;
```

간단하지만 테스트가 어려워 현재는 권장되지 않습니다.

---

## **IoC와 DI 관계**

```
IoC
 └─ 객체 제어권을 Spring이 가짐

DI
 └─ IoC를 구현하는 방법
```

즉,

IoC는 큰 개념이고, DI는 IoC를 구현하는 기술이다.

---

# **3. AOP (Aspect-Oriented Programming, 관점 지향 프로그래밍)**

## **개념**

핵심 비즈니스 로직과 공통 기능을 분리하는 프로그래밍 기법입니다.

---

예를 들어

```java
회원가입
로그인
주문
결제
```

모든 기능에 로그를 남겨야 한다고 가정해봅시다.

---

AOP가 없으면

```java
public void login() {
    log();
    ...
}

public void order() {
    log();
    ...
}

public void payment() {
    log();
    ...
}
```

중복 코드가 계속 발생합니다.

---

AOP를 사용하면

```java
@Aspect
public class LogAspect {

    @Before("execution(* com.example..*(..))")
    public void log() {
        System.out.println("로그 기록");
    }
}
```

Spring이 자동으로 실행합니다.

---

## **대표적인 공통 관심사(Cross Cutting Concern)**

### **로그**

```
로그인
주문
결제
```

모두 로그 필요

---

### **트랜잭션**

```java
@Transactional
public void order() {
    ...
}
```

---

### **보안**

```java
@PreAuthorize(...)
```

---

### **성능 측정**

```java
실행 시간 측정
```

---

## **핵심 관심사와 공통 관심사**

```
핵심 관심사
 ├─ 회원가입
 ├─ 로그인
 └─ 주문

공통 관심사
 ├─ 로그
 ├─ 트랜잭션
 ├─ 보안
 └─ 예외 처리
```

AOP는 공통 관심사를 분리하여 관리합니다.

---

## **Spring AOP 동작 방식**

Spring은 주로 **프록시(Proxy) 패턴**을 사용합니다.

```
사용자
   ↓
Proxy
   ↓
실제 객체
```

예를 들어

```java
orderService.order();
```

실제로는

```
orderService Proxy
      ↓
로그 수행
      ↓
트랜잭션 시작
      ↓
실제 order()
      ↓
트랜잭션 종료
```

순서로 동작합니다.

---

# **한눈에 비교**

| **구분** | **설명** |
| --- | --- |
| IoC | 객체 생성 및 관리 권한을 Spring이 가짐 |
| DI | Spring이 객체를 주입해주는 방식 |
| AOP | 공통 기능을 분리하여 관리하는 방식 |

---

# **면접 답변 예시**

IoC는 객체 생성과 생명주기에 대한 제어권을 개발자가 아닌 Spring 컨테이너가 가지는 개념입니다. DI는 IoC를 구현하는 방법으로, 필요한 객체를 직접 생성하지 않고 외부에서 주입받아 객체 간 결합도를 낮춥니다. AOP는 로그, 트랜잭션, 보안과 같은 공통 관심사를 핵심 비즈니스 로직과 분리하여 관리하는 기법이며, Spring에서는 프록시 기반으로 동작합니다. 이를 통해 코드의 재사용성과 유지보수성을 높일 수 있습니다.
