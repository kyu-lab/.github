# [WriteHere](https://www.writehere.kro.kr)
![Image.png](..%2Fimage%2FImage.png)
## 간단 소개
MSA의 기본적인 배경으로 설계된 게시판형 웹사이트입니다.  
인증/인가는 JWT를 이용하며 각 서비스간의 통신은 카프카와 REST API를 이용합니다.

### 인증/인가
1. UsersService에 로그인 성공시 액세스 토큰과 리프레쉬 토큰 발급
   1. 리프레쉬 토큰은 재인증을 위해 레디스에 저장
   2. 액세스 토큰(body)과 쿠키(HttpOnly)에 리프레쉬 토큰을 담아 클라이언트에게 전달
2. 클라이언트에서 액세스 토큰을 파싱하여 사용
3. 인증이 필요할경우 헤더에 Authorization에 액세스 토큰을 싣는다.
4. API-Gateway에서 토큰을 검증한다.
   1. 허용된 액세스 토큰은 통과
   2. 만료된 액세스 토큰은 쿠키의 리프레쉬 토큰과 레디스의 토큰을 비교하여 확인
      1. 쿠키와 레디스가 일치하다면 액세스 토큰을 재발급 url을 사용자에게 전달 후 클라이언트에서 재요청
      2. 일치하지 않을 경우 요청을 취소하고 클라이언트에 있는 인증값을 초기화 후 재로그인 요청
5. 사용자 ID와 같은 기본정보는 각 서비스에서 토큰을 파싱하여 사용한다.

### 카프카를 사용한 이유
MSA 구조에서는 서비스 간의 결합도를 낮추면서도 데이터 연동이 필요합니다.
이때 Kafka와 같은 메시지 브로커를 활용하면 서비스 간 통신을 비동기, 내결함성, 확장성 있게 처리할 수 있습니다.
Kafka는 높은 처리량과 내구성을 제공하며, Spring 진영과의 호환성도 우수하여 메시지 브로커로 채택하였습니다.

## 배포 과정
![Process.png](..%2Fimage%2FProcess.png)
1. 개발자가 포크한 개인 저장소에 소스를 반영합니다.
2. 깃허브의 PR을 통해 프로젝트 저장소에 반영합니다.
3. 프로젝트 저장소에 정상적으로 반영되면 깃허브의 webhooks를 이용해 젠킨스에 요청을 보냅니다.
4. 젠킨스가 요청을 받고 지정된 서버에 배포를 합니다.
5. 모든 애플리케이션들은 로컬과 비슷한 환경을 위해 도커를 이용하여 실행됩니다.

# 서비스
- 언어 : 자바스크립트(es6+), Java(17), SQL
- 서버 : AWS
- 빌드 : Git, Jenkins

아래부터는 각 서비스들에 대한 설명입니다.
### [client](https://github.com/kyu-lab/front)
React + vite로 구성되어 있으며 이유는 다음과 같습니다.  
React : SPA를 만들기 위해 커뮤니티 규모가 거대하여 개발에 용이하였다.   
vite : CRA에 비해 개발 환경이 훨씬 가볍고 빠르다.

### [Config-Server](https://github.com/kyu-lab/config-server)
Spring-Cloud-Config를 이용한 애플리케이션 설정 서버입니다.
현재는 실제 파일을 하나의 운영서버에 두고 운영하고 있습니다.
분산된 서비스에서 설정을 한 곳에서 관리하기 위해서 채택하였습니다.

### [Discovery-Server](https://github.com/kyu-lab/discovery-server)
netflix-eureka-server를 이용한 서비스 디스커버리 서버입니다.  
분산된 서비스들을 파악하기 쉽게 사용하는 용도로 자바진영에서 가장 유명한 서버를 채택

### [Gateway-Service](https://github.com/kyu-lab/gateway-service)
Spring Cloud Gateway 기반으로 사용자의 모든 요청을 한곳에서 제어하기 위해 사용합니다.    
기본적인 인증(토큰)을 담당하고 있으며 디스커버리 서버와 연동하여 라우팅합니다.  
Spring Cloud Gateway는 기본적으로 netty 서버를 사용하며 WebFlux 기반입니다.

### [Users-Service](https://github.com/kyu-lab/users-service)
Spring Mvc + Spring Data JPA 기반으로 사용자 인증/인가와 관리를 담당합니다.  
검색과 사용자 이미지 데이터는 카프카를 이용해 전달합니다.  
Postgresql를 채택한 이유는 오픈소스이면서 SQL 표준을 잘지키며, 다양한 데이터 타입(JSON, ARRAY 등)을 지원하여 채택하였습니다.  

### [Post-Service](https://github.com/kyu-lab/post-service)
Spring Mvc + Spring Data JPA 기반으로 게시글, 댓글, 그룹 기능을 담당합니다.   
게시글의 이미지와 알림 정보는 카프카를 이용해 각 서비스에게 전달합니다.
데이터는 Postgresql에 저장됩니다.

### [Notices-Service](https://github.com/kyu-lab/notices-service)
Spring WebFlux 기반으로 사용자 알림을 담당합니다.    

WebFlux 채택 이유  
논블로킹 I/O와 비동기 프로그래밍을 지원해 대용량 데이터와 동시 요청 처리에 적합하독 판단

알림 서비스는 다음과 같이 동작합니다.
1. 로그인된 사용자는 EventSource를 이용해 알림 서버와 연결됩니다.
2. 게시글 서비스, 사용자 서비스 등에게서 알림 메시지를 카프카로 전달받습니다.  
3. 사용자는 메시지를 SSE로 전달받습니다.  

데이터는 mongodb로 저장합니다.

### [File-Service](https://github.com/kyu-lab/file-service)
Spring WebFlux 기반으로 파일 저장/읽기를 담당합니다.  
현재는 파일을 보기위한 url(예: "/api/file/image/{imageId}")  
url을 통해 게시글, 사용자 아이콘 등의 이미지를 불러옵니다.  
데이터는 mongodb로 저장합니다.

### [Search-Service](https://github.com/kyu-lab/serach-service)
Spring WebFlux + elasticsearch를 이용하여 검색 기능을 사용합니다.  
사용자 데이터, 게시글 데이터는 카프카를 이용해 전달받습니다.