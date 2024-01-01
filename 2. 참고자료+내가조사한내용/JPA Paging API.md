
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


## 각 메서드의 내부 코드를 살펴보자

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
2. 쿼리 성공여부를 알려주는 `boolean success` 변수를 false로 설정한다.
3. doList() 메서드를 통해 쿼리를 실행한다. 쿼리실행에 성공하면 `success = true` 를 설정한 후 실행 결과를 반환한다.