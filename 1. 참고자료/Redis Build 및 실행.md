valkey 프로젝트 실행하기

빌드
% make
makefile.dep 파일에 있는 모든 명령어들을 순차적으로 실행한다.
![](https://i.imgur.com/78BrgBa.png)


실행
% cd src
% ./valkey-server

![](https://i.imgur.com/pU2XJ95.png)



in-memory data source 실행

![](https://i.imgur.com/H1ek5DI.png)

간단한 get/set test
![](https://i.imgur.com/9Hxzi1S.png)

v2요청
![](https://i.imgur.com/0lrVO87.png)
레디스의 hashes 명령어다.

저장하기 문법: HSET key field vlaue
읽기 문법: HGET key field

![](https://i.imgur.com/N8DXn9U.png)


counter 증가하는 방법
incr a
-> increment a
-> 변수 a 값
![](https://i.imgur.com/ZvjGZO8.png)


telnet 에 연결해서 v2명령어로 실행했다.
telnet 127.0.0.1 6379
![](https://i.imgur.com/AxRgue5.png)
