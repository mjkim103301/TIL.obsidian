## 실제 테스트해본 결과

```
void dataInsert() {  
    for (int i = 0; i < 1000; i++) {  
        ConfirmationType confirmationType1 = ConfirmationType  
                .builder()  
                .title("암")  
                .build();  
        confirmationTypeRepository.save(confirmationType1);  
    }  
}  
  
@Test  
void count(){  
    dataInsert();  
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    TypedQuery<Long> count = em.createQuery("SELECT count(c.id) From Confirmation c", Long.class);  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("count 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
  
@Test  
void em_createquery() {  
    dataInsert();  
    long beforeTime = System.currentTimeMillis(); // 코드 실행 전에 시간 받아오기  
    // 시간체크 시작  
    TypedQuery<Confirmation> query = em.createQuery("SELECT c From Confirmation c", Confirmation.class);  
    query.setFirstResult(100);  
    query.setMaxResults(20);  
    query.getResultList();  
  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("em.createQuery 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
  
@Test  
void page() {  
    dataInsert();  
  
  
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    Pageable pageable = PageRequest.of(5, 20);  
    Page<Confirmation> pages = confirmationRepository.findAll(pageable);  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("page findAll 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
}  
  
@Test  
void slice() {  
    dataInsert();  
  
    long beforeTime = System.currentTimeMillis();  
    // 시간체크 시작  
    Pageable pageable = PageRequest.of(5, 20);  
    Slice<Confirmation> slices = confirmationRepository.findAll(pageable);  
  
    // 시간체크 끝  
    long afterTime = System.currentTimeMillis(); // 코드 실행 후에 시간 받아오기  
    long diffTime = afterTime - beforeTime; // 두 개의 실행 시간  
    System.out.println("find findSlice 실행 시간(ms): " + diffTime); // 세컨드(초 단위 변환)  
  
    // 시간체크 시작  
    //List<Confirmation> list = confirmationRepository.findAll(pageable);  
  
    // 시간체크 끝  
}
```


1000개 데이터
100번째 데이터
20개

em.createquery
select 쿼리만 실행됨
count 쿼리는 실행되지 않음
1. 90ms
2. 89ms
3. 70ms
4. 77ms
5. 83ms
81.8

page
select 쿼리
count 쿼리 실행됨
1. 95ms
2. 81ms
3. 101ms
4. 83ms
5. 90ms
90

slice
1. 81ms
2. 79ms
3. 116ms
4. 81ms
5. 92ms
89.8


count
80ms...?