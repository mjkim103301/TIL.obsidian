## JPA 페이징 API

```
TypedQuery<Member> query =
    em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.setFirstResult(10);
query.setMaxResults(20);
query.getResultList();
```

JPA 는 페이징을 2개의 API로 추상화했다.

- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)
- setMaxResults(int maxResult): 조회할 데이터 수
- setFirstResult 의 startPosition은 0부터 시작하므로 0이 1번째 데이터다.

위의 코드를 분석해보면 11번째 데이터부터~30번째 데이터(총 20개)의 데이터를 조회한다.

### 각 메서드의 내부 코드를 살펴보자
```
// query.setFirstResult(10);
public void applyFirstResult(int startPosition) {  
    if ( startPosition < 0 ) {  
       throw new IllegalArgumentException( "first-result value cannot be negative : " + startPosition );  
    }  

    getSession().checkOpen();  
    getQueryOptions().getLimit().setFirstRow( startPosition );  
}
```

setFirstResult 메서드로 조회 시작 위치를 설정한다.  
매개변수로 들어온 값이 0보다 작으면 exception을 발생시킨다.  
현재 데이터베이스에 연결된 세션을 가져와서 트랜젝션 open변수에 true를 세팅한다.  
그리고 Limit 클래스에서 조회를 시작할 행의 번호를 설정한다.


```
// query.setMaxResults(20);
public void applyMaxResults(int maxResult) {  
    if ( maxResult < 0 ) {  
       throw new IllegalArgumentException( "max-results cannot be negative" );  
    }  
    getSession().checkOpen();  
    getQueryOptions().getLimit().setMaxRows( maxResult );  
}
```

setMaxResults 메서드로 조회할 데이터의 개수를 설정한다.  
매개변수로 들어온 값이 0보다 작으면 exception을 발생시킨다.  
현재 데이터베이스에 연결된 세션을 가져와서 트랜젝션 open변수에 true를 세팅한다.  
Limit 클래스에서 조회할 데이터 개수를 설정한다.

```
// query.getResultList()
@Override  
public List<R> list() {  
    beforeQuery();  
    boolean success = false;  
    try {  
       final List<R> result = doList();  
       success = true;  
       return result;  
    }  
    catch (IllegalQueryOperationException e) {  
       throw new IllegalStateException( e );  
    }  
    catch (TypeMismatchException e) {  
       throw new IllegalArgumentException( e );  
    }  
    catch (HibernateException he) {  
       throw getSession().getExceptionConverter().convert( he, getQueryOptions().getLockOptions() );  
    }  
    finally {  
       afterQuery( success );  
    }  
}

protected void beforeQuery() {  
    getQueryParameterBindings().validate();  

    getSession().prepareForQueryExecution(  
          requiresTxn( getQueryOptions().getLockOptions().findGreatestLockMode() )  
    );  
    prepareForExecution();  

    assert sessionFlushMode == null;  
    assert sessionCacheMode == null;  

    final FlushMode effectiveFlushMode = getHibernateFlushMode();  
    if ( effectiveFlushMode != null ) {  
       sessionFlushMode = getSession().getHibernateFlushMode();  
       getSession().setHibernateFlushMode( effectiveFlushMode );  
    }  

    final CacheMode effectiveCacheMode = getCacheMode();  
    if ( effectiveCacheMode != null ) {  
       sessionCacheMode = getSession().getCacheMode();  
       getSession().setCacheMode( effectiveCacheMode );  
    }  
}
```

마지막으로 getResultList 메서드를 통해 쿼리를 통해 조회한 결과를 가져온다.  
이 메서드 내부는 3단계로 나눠진다.

1. beforeQuery() 메서드를 통해 파라미터를 쿼리에 바인딩하고, 쿼리에 설정된 데이터베이스 락 옵션, 기타 옵션들을 설정한다.
2. 쿼리 성공여부를 알려주는 `boolean success` 변수를 false로 설정한다.

doList() 메서드를 통해 쿼리를 실행한다. 쿼리실행에 성공하면 `success = true` 를 설정한 후 실행 결과를 반환한다.

## JPA 페이징 기능

![](https://i.imgur.com/PqLdVKs.png)

JpaRepository 인터베이스의 부모인 PagingAndSortingRepository를 통해 간편하게 페이지 기능을 사용할 수 있다.

```
Page<T> findAll(Pageable pageable); 
```


### 동작과정

Pageable 인터페이스로 조회할 시작 페이지 번호와 데이터 개수를 세팅한다.

```
Pageable pageable = PageRequest.of(0, 2);

public static PageRequest of(int page, int size) {  
    return of(page, size, Sort.unsorted());  
}
```

page 변수는 0부터 시작되며, size 변수는 0보다 커야 한다.

```
public static PageRequest of(int page, int size, Sort sort) {  
    return new PageRequest(page, size, sort);  
}
```

Sort 클래스도 인자로 넘기면 정렬도 할 수 있다.

그리고 실제로 findAll 메서드가 실행되면 JdkDynamicAopProxy.java 클래스의 Invoke 메서드가 실행된다.  
자세한 내용은 모르겠다.. 어렵다..
```

@Override  
@Nullable  
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
    Object oldProxy = null;  
    boolean setProxyContext = false;  

    TargetSource targetSource = this.advised.targetSource;  
    Object target = null;  

    try {  
       if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {  
          // The target does not implement the equals(Object) method itself.  
          return equals(args[0]);  
       }  
       else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {  
          // The target does not implement the hashCode() method itself.  
          return hashCode();  
       }  
       else if (method.getDeclaringClass() == DecoratingProxy.class) {  
          // There is only getDecoratedClass() declared -> dispatch to proxy config.  
          return AopProxyUtils.ultimateTargetClass(this.advised);  
       }  
       else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&  
             method.getDeclaringClass().isAssignableFrom(Advised.class)) {  
          // Service invocations on ProxyConfig with the proxy config...  
          return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);  
       }  

       Object retVal;  

       if (this.advised.exposeProxy) {  
          // Make invocation available if necessary.  
          oldProxy = AopContext.setCurrentProxy(proxy);  
          setProxyContext = true;  
       }  

       // Get as late as possible to minimize the time we "own" the target,  
       // in case it comes from a pool.       target = targetSource.getTarget();  
       Class<?> targetClass = (target != null ? target.getClass() : null);  

       // Get the interception chain for this method.  
       List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);  

       // Check whether we have any advice. If we don't, we can fall back on direct  
       // reflective invocation of the target, and avoid creating a MethodInvocation.       if (chain.isEmpty()) {  
          // We can skip creating a MethodInvocation: just invoke the target directly  
          // Note that the final invoker must be an InvokerInterceptor so we know it does          // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.          Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);  
          retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);  
       }  
       else {  
          // We need to create a method invocation...  
          MethodInvocation invocation =  
                new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);  
          // Proceed to the joinpoint through the interceptor chain.  
          retVal = invocation.proceed();  // 여기서 실제 쿼리가 실행된다. select content, select count 쿼리가 순차적으로 실행된다.
       }  

       // Massage return value if necessary.  
       Class<?> returnType = method.getReturnType();  
       if (retVal != null && retVal == target &&  
             returnType != Object.class && returnType.isInstance(proxy) &&  
             !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {  
          // Special case: it returned "this" and the return type of the method  
          // is type-compatible. Note that we can't help if the target sets          // a reference to itself in another returned object.          retVal = proxy;  
       }  
       else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {  
          throw new AopInvocationException(  
                "Null return value from advice does not match primitive return type for: " + method);  
       }  
       return retVal;  
    }  
    finally {  
       if (target != null && !targetSource.isStatic()) {  
          // Must have come from TargetSource.  
          targetSource.releaseTarget(target);  
       }  
       if (setProxyContext) {  
          // Restore old proxy.  
          AopContext.setCurrentProxy(oldProxy);  
       }  
    }  
}
```

## JPA Paging 최적화

### JPA 페이징 처리에서 발생할 수 있는 성능 이슈

JPA 의 페이징 기능을 사용하게 되면 다음 메서드를 실행하게 된다.

```
Page<T> findAll(Pageable pageable);
```

그러면 select, count 쿼리가 각각 날라가게 된다.

```
Hibernate: select userentity0_.id as id1_0_, userentity0_.created_at as created_2_0_, userentity0_.email as email3_0_, userentity0_.password as password4_0_, userentity0_.updated_at as updated_5_0_ from user userentity0_ limit ?, ? Hibernate: select count(userentity0_.id) as col_0_0_ from user userentity0_
```

성능 이슈는 count 쿼리를 실행할 때 발생한다.  
count 쿼리는 테이블 full-scan을 필요로 하기 때문에 테이블에 데이터가 많을수록 성능 저하가 발생한다.

### 성능 이슈 해결 방안

만약 조회한 데이터에서 Total Count 데이터가 필요하지 않다면 다른 반환 타입을 사용할 수 있다. (count 쿼리를 실행하지 않기 위한 방법)

1. List

```
class OrderCustomRepositoryImpl : QuerydslRepositorySupport(Order::class.java), OrderCustomRepository {  
override fun findSliceBy(pageable: Pageable, address: String): Slice<Order> {  
  val query: JPAQuery<Order> = from(order).select(order).where(order.address.eq(address))  
  val content: List<Order> = querydsl.applyPagination(pageable, query).fetch()  
  val hasNext: Boolean = content.size >= pageable.pageSize  
  return SliceImpl(content, pageable, hasNext)  
}  
}
```

2. Slice

3. QueryDSL : 여러 테이블을 조인해서 조회할 때 count 쿼리는 최대한 조인하지 않고 사용한다.

  

```
-- Content 조회 쿼리  
select o.*,  
       u.*,  
       c.*  
from orders o  
         left join coupon c on o.coupon_id = c.id  
         inner join user u on o.user_id = u.id  
where o.address = ? limit ?, ?  
;  
  
-- Content 조회 쿼리를 그대로 사용하는 경우  
select count(o.id)  
from orders o  
         left join coupon c on o.coupon_id = c.id  
         inner join user u on o.user_id = u.id  
where o.address = ?  
;  
  
-- Content 쿼리를 사용하지 않고 별도의 Count 조회 쿼리  
select count(o.id) ascount  
from orders o  
where o.address = ?  
;
```

  

4. Content 조회 쿼리, Count 조회 쿼리 병렬처리하기  
    *  Content 조회쿼리는 500ms 소요되고, Count 조회 쿼리는 1000ms 소요될 때 전체적인 페이징 시간은 1500ms 소요되게 된다.  
    * 위의 2개의 조회 작업은 의존성이 없기 때문에 병렬로 처리할 수 있다.

```
class OrderCustomRepositoryImpl : QuerydslRepositorySupport(Order::class.java), OrderCustomRepository {  
    override fun findPagingBy(pageable: Pageable, address: String): Page<Order> = runBlocking {  
        log.info("findPagingBy thread : ${Thread.currentThread()}")  
        val content: Deferred<List<Order>> = async {  
            log.info("content thread : ${Thread.currentThread()}")  
            from(order)  
                .select(order)  
                .innerJoin(user).on(order.userId.eq(user.id))  
                .leftJoin(coupon).on(order.couponId.eq(coupon.id))  
                .where(order.address.eq(address))  
                .run {  
                    querydsl.applyPagination(pageable, this).fetch()  
                }  
        }  
        val totalCount: Deferred<Long> = async {  
            log.info("count thread : ${Thread.currentThread()}")  
            from(order)  
                .select(order.count())  
                .where(order.address.eq(address))  
                .fetchFirst()  
        }  
  
        PageImpl(content.await(), pageable, totalCount.await())  
    }  
}
```

  

### 실제 테스트해본 결과

```
void dataInsert() {  
    for (int i = 0; i < 1000; i++) {  
        Member member = Member  
                .builder()  
                .title("안녕")  
                .build();  
        memberRepository.save(member);  
    }  
}  
```
  
```
@Test  
void count(){  
    dataInsert();
    
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    TypedQuery<Long> count = em.createQuery("SELECT count(m.id) From Member m", Long.class);  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("count 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
```
  
```
@Test  
void em_createquery() {  
    dataInsert();
    
    long beforeTime = System.currentTimeMillis(); // 코드 실행 전에 시간 받아오기  
    // 시간체크 시작  
    TypedQuery<Member> query = em.createQuery("SELECT m From Member m", Member.class);  
    query.setFirstResult(100);  
    query.setMaxResults(20);  
    query.getResultList();  
  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("em.createQuery 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
```
  
```
@Test  
void page() {  
    dataInsert();  
  
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    Pageable pageable = PageRequest.of(5, 20);  
    Page<Member> pages = memberRepository.findAll(pageable);  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("page findAll 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
```
  
```
@Test  
void slice() {  
    dataInsert();  
  
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    Pageable pageable = PageRequest.of(5, 20);  
    Slice<Member> slices = memberRepository.findAll(pageable);  
  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("find findSlice 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}
```

#### 테스트 조건

- DB저장 데이터: 1000개
- 검색 데이터: 100번 ~120번 (20개)
- 5번씩 수행

#### 테스트 결과

|   |   |   |   |
|---|---|---|---|
||페이징 API (em.createQuery)|Pageable|Slice|
|최저(ms)|70|81|79|
|최고(ms)|89|101|116|
|평균(ms)|81.8|90|89.8|

  

#### 느낀점

페이징 API > Slice > Pageable 순서로 빠른 것을 알 수 있다.

  

생각보다 시간차이가 많이 나지는 않는다.

그리고 count 쿼리만 실행했을 때 속도는 80ms 다.

분명 JPA page기능을 사용하면 `select content + select count = 최종 조회 시간` 이 되는데 결과값이 맞지 않다.

  

테스트를 잘못한걸까...? 확인을 해봐야겠다.

![](https://t1.daumcdn.net/keditor/emoticon/face/large/070.png)

  

---

참고자료

- [Spring, JPA 페이징 처리와 성능 최적화 알아보기](https://blog.leaphop.co.kr/blogs/36)  
- [JPA 페이징 Performance 향상 방법](https://cheese10yun.github.io/page-performance/ "JPA 페이징 Performance 향상 방법")

- [[배워보자 Spring Data JPA] JPA 에서 Pageable 을 이용한 페이징과 정렬](https://wonit.tistory.com/483)

- 도서: 자바 ORM 표준 JPA 프로그래밍 p364 페이징 API