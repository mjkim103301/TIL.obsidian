![](https://i.imgur.com/PqLdVKs.png)

JpaRepository 인터베이스의 부모인 PagingAndSortingRepository를 통해 간편하게 페이지 기능을 사용할 수 있다. 
```
Page<T> findAll(Pageable pageable); 
```

## 동작과정

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
