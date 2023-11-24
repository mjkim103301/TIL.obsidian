Jpa에서 setFirstResult(),. setMaxResultss() 를 사용하는 방법과는 다른 것 같다.
```
TypedQuery<Member> query =
	em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.setFirstResult(10);
query.setMaxResults(20);
query.getResultList();
```