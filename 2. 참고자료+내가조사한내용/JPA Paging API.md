
```
TypedQuery<Member> query =
	em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.setFirstResult(10);
query.setMaxResults(20);
query.getResultList();
```

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