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

위의 코드를 분석해보면 11~30번(총 20개)의 데이터를 조회한다.

[출처: 자바 ORM 표준 JPA 프로그래밍 p364 페이징 API]