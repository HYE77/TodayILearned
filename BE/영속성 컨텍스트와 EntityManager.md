# 1. 영속성 컨텍스트 Persistence Context란?

<aside>
💡

**엔티티를 저장, 관리하는 JPA의 논리적 저장 공간**

</aside>

- 엔티티의 생명주기 상태를 관리
- 트랜잭션 범위 내에서 유지됨
- DB가 아니라 메모리 상의 컨텍스트

→ 엔티티 인스턴스를 관리하고 동일성 (identity)을 보장하는 컨텍스트

## 1-1. 핵심 기능

### (1) 1차 캐시

```java
Member m1 = em.find(Member.class, 1L);
Member m2 = em.find(Member.class, 1L);
```

- DB 조회는 1회
- 두 객체는 같은 인스턴스

```java
m1 == m2 // true
```

### (2) 엔티티 동일성 보장

- 같은 ID → 같은 객체
- Java 레벨에서 동일성 보장
- *DB 트랜잭션과는 다른 개념*

### (3) 트랜잭션 쓰기 지연

```java
em.persist(member);
em.persist(order);
// 아직 INSERT 안 나감
```

- SQL은 즉시 실행 X
- `flush` 시점에 모아서 실행\

### (4) 변경 감지 (Dirty Checking)

```java
member.setName("newName");
```

- 영속성 컨테이너가 관리하는 엔티티의 상태를 감지해서 변경된 부분이 있다면 자동으로 트랜잭션이 끝나는 시점에 데이터베이스에 반영.
- `update()` 호출 불필요, 관련 쿼리 작성 불필요.
- 트랜잭션 커밋 시 자동 UPDATE
- setter 기반 도메인 설계가 가능한 이유

### (5) 지연 로딩(Proxy) 관리

```java
@ManyToOne(fetch = FetchType.LAZY)
private Member member;
```

- `LAZY` 면 `Order.getMember()`가 처음부터 Member 실체를 들고 있는 게 아니라 껍데기 (대리 객체)를 들고 있음
    - memberId 같은 식별자 값 정도는 알고 있음
    - Member의 다른 필드는 아직 모름
    - 그 필드는 진짜로 필요한 순간 DB를 조회해서 채움
- 프록시는  Persistence context 안에서만 초기화 가능
    - 프록시가 초기화( DB를 조회해서 실제로 인티티를 채움)되려면 DB 조회를 수행할 수 있는 실행 환경 필요
        - 열려있는 세션/EntityManager
        - 그에 연결된 Persistence Context
        - 스프링에서는 보통 @Transactional 범위와 묶여 있음
- 트랜잭션 밖 → 예외 발생

---

# 2. Entity Manager

<aside>
💡

**Persistence Context를 조작하는 인터페이스 (API)**

- Persistence Context = 상태를 보관하는 공간
- Entity Manager = 그 공간을 조작하는 도구
</aside>

```java
@PersistenceContext
private EntityManager em;
```

## 2-1. 엔티티 생명 주기 Entity Lifecycle

<img width="445" height="337" alt="image" src="https://github.com/user-attachments/assets/2ccf0b09-3d59-4048-b4f7-55402c966dd3" />


### (1) 비영속 Transient

```java
Member member = new Member();
```

- 단순한 자바 객체
- DB와 전혀 무관
- Persistence Context가 알지 못함
- 1차 캐시에도 없음
- **SQL 발생 ❌**

### (2) 영속 Managed

```java
em.persist(member);
```

또는

```java
Member member = em.find(Member.class, 1L);
```

- **비영속 → 영속**
- Persistence Context가 관리
- 1차 캐시에 저장
- 변경 감지(Dirty Checking) 대상
- 트랜잭션 커밋 시 자동 SQL 실행
    - persist 시 바로 나가는 것 ❌
    - 보통 flush/commit 시점에 INSERT

### (3) 준영속 Detached

```java
em.detach(member);
```

또는

```java
em.clear();
```

- 트랜잭션 종료
- **영속 → 준영속**
- 더 이상 관리하지 않음
- 변경 감지 ❌
- 1차 캐시에서 제거
- 프록시 초기화 불가
- **SQL 발생** ❌ (관리만 끊김)

### **⚠️ 준영속의 위험**

```java
member.setName("변경");
```

- 준영속 시점에서는 UPDATE 나가지 않음
- `why?`  Persistence Context가 이 객체를 추적하지 않기 때문

### (4) 삭제 (Removed)

```java
em.remove(member);
```

- **영속 → 삭제**
- 삭제 예약 상태
- flush/commit 시 DELETE SQL 실행

### 서비스 메서드 안에서는 대부분 영속 상태

```java
@Transactional
public void update(Long id) {
		Member m = repository.findById(id).get();
		m.setName("변경");
}
```

- 트랜잭션 안에서 조회하면 **영속**
- 트랜잭션이 끝날 때 자동 UPDATE

### 트랜잭션 밖에서는 전부 준영속

- Controller에서 받은 엔티티는 보통 준영속
- 따라서 `save()`를 다시 호출해야 반영

## 2-2. EntityManager의 주요 메서드와 의미

<aside>
💡

**JPA의 모든 강력한 기능은 영속 상태에서만 작동한다**

1차 캐시, 변경 감지, 지연 로딩, 자동 UPDATE 등

</aside>

### persist()  - 비영속 → 영속 (등록)

- **DB에 저장하는 게 아니라 영속성 컨텍스트에 등록하는 것**

```java
@Transactional
public void saveMember() {
		Member member = new Member("kim"); // 비영속
		
		em.persist(member); // 영속 상태 전환
}
```

- 1차 캐시에 저장됨
- 변경 감지 대상이 됨
- INSERT SQL은 아직 안 나갈 수 있음 ( 쓰기 지연)
- 커밋 시점에 INSERT 실행

### find() - 즉시 조회 (영속 상태 반환)

```java
@Transactional
public void findMember() {
		Member member = em.find(Member.class, 1L);
}
```

- 1차 캐시 먼저 확인 → 없으면 SELECT 실행 → 영속 상태로 반환
- 동일성 보장 (1차 캐시 때문)

### getReference() - 프록시 반환

```java
@Transactional
public void setRelation(Long memberId) {
		Member member = em.getReference(Member.class, memberId);
		System.out.println(member.getId());     // SELECT 안 나감
    System.out.println(member.getName());   // 여기서 SELECT 발생
}
```

- DB 조회 없이 프록시 생성
- 실제 필드 접근 시 SELECT

```java
@Transactional
public void createOrder(Long memberId) {
    Member member = em.getReference(Member.class, memberId);

    Order order = new Order(member);
    em.persist(order);
}
```

### remove() - 삭제 예약

```java
@Transactional
public void deleteMember(Long id) {
		Member member = em.find(Member.class, id);
		em.remove(member);
}
```

- 영속 → 삭제 상태
- 즉시 DELETE ❌
- flush/commit 시 DELETE 실행

⚠️ **주의**

```java
Member m = new Member("kim");
em.remove(m); // 예외 발생 (비영속은 삭제 불가) 반드시 영속 상태여야
```

### flush() - 강제 DB 동기화

```java
@Transactional
public void updateAndFlush(Long id) {
		Member member = em.find(Member.class, id);
		member.changeName("lee");
		
		em.flush(); // 여기서 UPDATE 실행
}
```

- 변경 감지 결과를 DB에 즉시 반영
- 트랜잭션은 아직 종료되지 않음
- **제약조건 위반을 빨리 확인하고 싶을 때**
- **다음 쿼리가 변경 데이터를 기준으로 동작해야 할 때 이용**

### clear() - 전체 준영속화

```java
@Transactional
public void clearExample(Long id) {
		Member member = em.find(Member.class, id);
		member.changeName("park");
		
		em.clear(); // 전체 영속성 컨텍스트 초기화
		
		member.changeName("choi"); // UPDATE 안 나감
}
```

- member는 이제 **준영속**
- 변경 감지 작동 ❌
- 대량 배치 처리 시 메모리 관리를 위해 이용

### detach() - 특정 entity만 준영속

```java
@Transactional
public void detachExample(Long id) {
		Member member = em.find(Member.class, id);
		
		em.detach(member);
		
		member.changeName("han"); // UPDATE 안나감
}
```

- 특정 객체만 관리 중지
- `clear`는 전체, `detach`는 개별

### 상태 변화 흐름 요약

```java
new -> persist() -> 영속
영속 -> detach()/clear() -> 준영속
영속 -> remove() -> 삭제
```

| 메서드 | 상태 변화 |
| --- | --- |
| new | 없음 → 비영속 |
| persist | 비영속 → 영속 |
| find | DB → 영속 |
| detach | 영속 → 준영속 |
| clear | 전체 영속 → 준영속 |
| remove | 영속 → 삭제 |
| commit | 삭제 → DB 반영 |

### 서비스 레이어 관점에서 이해하기

| 상황 | 상태 |
| --- | --- |
| @Transactional 내부 조회 | 영속 |
| 트랜잭션 종료 후 | 준영속 |
| Controller에서 받은 엔티티 | 보통 준영속 |
| save() 호출 | 다시 영속 |
