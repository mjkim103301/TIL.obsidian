JPA 부트스트림 모드는 기본,  Lazy, Deferred 로 나뉜다.

###  Default
기본 설정은 앱 실행 전에 Repository들을 모두 초기화시킨다.
Repository 등록 
-> Repository 초기화
-> Repository 인스턴스 생성
-> 코드 실행

### Lazy
Repository 등록
-> 테스트코드 실행
-> Repository 프록시 생성
-> Repository 초기화

### Deferred
Repository 등록
-> Repository 초기화 리스너 등록
-> Repository 초기화
-> 테스트코드 실행

Deferred 모드는 비동기로 Repository를 실행하려고 할 때 많이 사용하며, 어플리케이션 실행시작 시점이 빨라진다.


[BootstrapMode for JPA Repositories](https://www.baeldung.com/jpa-bootstrap-mode)
[@EnableJpaRepositories bootstrapMode 정리](https://effortguy.tistory.com/281)