# kyu-lab 🚀
이곳은 백엔드 기술들 마스터하기 위한 개인 작업실입니다.  
Spring Boot, Kafka, WebFlux로 분산 시스템을 구축하며, 학습과 도전을 코드로 구현합니다.  

## 소개
kyu-lab은 REST API 설계부터 이벤트 기반 시스템 통합까지, 이곳의 모든 프로젝트는 복잡한 문제를 해결하고 다양한 기술들을 실험하고 있습니다.  
**"행동으로 배우는"** 과정을 통해 성장하는 것을 즐깁니다. 

## 기술 스택
- 언어: Java(17), JavaScript, SQL, NoSQL
- 백엔드: Spring Boot, WebFlux, Spring Cloud (Config, Eureka, Gateway), JPA
- 프론트엔드: React, react-router, axios
- 도구: Kafka, Redis, PostgreSQL, MongoDB, Elasticsearch, Docker, Jenkins, Git
- 아키텍처: Microservices (MSA), REST API, Event-Driven, Reactive Programming

## 주요 프로젝트

### 📝 Post Service

설명: 게시글, 댓글, 그룹 등 게시판 기능을 다루는 마이크로서비스입니다.

기술 스택: Spring Boot, PostgreSQL, JPA, Kafka

주요 성과:
- 게시글 작성 알림을 위해 Kafka를 이용
- 

링크: [post-service](https://github.com/kyu-lab/post-service)

🔐 Users Service

설명: 사용자 인증 및 권한 관리를 처리하는 마이크로서비스로, 시스템 전반의 안전한 접근을 보장합니다.

기술 스택: Spring Boot, PostgreSQL, JPA, Redis

주요 성과:
- JWT 기반 인증으로 stateless API 보안 구현
- 토큰 관리를 위해 Redis를 이용

링크: [users-service](https://github.com/kyu-lab/users-service)

🌐 Gateway Service

설명: 모든 API 요청의 진입점으로, Spring Cloud Gateway를 사용하여 마이크로서비스 간 트래픽을 라우팅합니다.

기술 스택: Spring Boot, Spring Cloud Gateway

주요 성과:
- 동적 라우팅으로 서비스 간 원활한 통신 보장
- eureka를 이용한 로드밸런스 

링크: [gateway-service](https://github.com/kyu-lab/gateway-service)


### 연락처

📧 이메일: slgmgm@naver.com

더 깊이 알고 싶으시다면 리포지토리를 확인하거나 메시지를 남겨 주세요!
