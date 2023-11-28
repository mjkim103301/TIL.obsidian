Jpa에서 setFirstResult(),. setMaxResults() 를 사용하는 방법과 Pageable을 사용하는 방법은 다른 것 같다.
```
TypedQuery<Member> query =
	em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.setFirstResult(10);
query.setMaxResults(20);
query.getResultList();
```

