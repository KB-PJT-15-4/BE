# 🔧 MOA 프로젝트 설정 파일 가이드

## ⚠️ 중요: Git 설정 관리

### application.properties 파일 관리
- **application.properties**는 `.gitignore`에 추가되어 Git에 추적되지 않습니다
- 로컬 설정은 각자 환경에 맞게 설정해야 합니다
- **application.properties.example** 파일을 복사하여 사용하세요:
  ```bash
  cp src/main/resources/application.properties.example src/main/resources/application.properties
  # 이후 본인 환경에 맞게 수정
  ```

---

## 📋 환경별 설정 구조

### 1. 로컬 개발 환경 (Docker 사용 X)
- **설정 파일**: `application.properties`
- **데이터베이스**: localhost:3306 (로컬 MySQL)
- **Redis**: localhost:6379 (로컬 Redis)
- **프로필**: 기본 (default)

### 2. AWS 배포 환경 (Docker 사용)
- **설정 파일**: `application-docker.properties`
- **데이터베이스**: mysql:3306 (Docker 서비스명)
- **Redis**: redis:6379 (Docker 서비스명)
- **프로필**: docker

---

## 📁 주요 설정 파일들

### 1. **application.properties** (로컬용)
```properties
# 로컬 MySQL
jdbc.url=jdbc:mysql://localhost:3306/moa?useSSL=false&serverTimezone=Asia/Seoul
jdbc.username=root
jdbc.password=1118

# 로컬 Redis
redis.host=localhost
redis.port=6379
```

### 2. **application-docker.properties** (Docker용)
```properties
# Docker MySQL (서비스명 사용)
jdbc.url=jdbc:mysql://mysql:3306/moa?useSSL=false&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
jdbc.username=root
jdbc.password=1118

# Docker Redis (서비스명 사용)
redis.host=redis
redis.port=6379
```

### 3. **RootConfig.java**
- 프로필에 따라 다른 설정 파일 사용
- `@Profile("docker")`: Docker 환경
- `@Profile("!docker")`: 로컬 환경

### 4. **docker-compose.yml**
```yaml
services:
  mysql:
    container_name: moa-mysql
    environment:
      MYSQL_ROOT_PASSWORD: 1118
      MYSQL_DATABASE: moa
  
  redis:
    container_name: moa-redis
  
  tomcat:
    environment:
      SPRING_PROFILES_ACTIVE: docker  # Docker 프로필 활성화
```

---

## 🚀 사용 방법

### 로컬 개발
```bash
# IntelliJ에서 실행 (기본 프로필 사용)
# application.properties 자동 사용
```

### Docker 배포
```bash
# Docker Compose 실행시 자동으로 docker 프로필 활성화
docker-compose up -d

# Tomcat 컨테이너에서 SPRING_PROFILES_ACTIVE=docker 설정됨
# application-docker.properties 자동 사용
```

---

## ✅ 체크리스트

### 로컬 환경
- [ ] MySQL 로컬 설치 (3306 포트)
- [ ] Redis 로컬 설치 (6379 포트) - 선택사항
- [ ] application.properties 파일 확인
- [ ] localhost로 접속 설정

### Docker 환경
- [ ] docker-compose.yml 파일 확인
- [ ] application-docker.properties 파일 확인
- [ ] SPRING_PROFILES_ACTIVE=docker 환경변수
- [ ] 서비스명(mysql, redis)으로 접속 설정

---

## 🔍 문제 해결

### 1. 로컬에서 MySQL 연결 실패
```bash
# MySQL 실행 확인
mysql -uroot -p1118 -e "SHOW DATABASES;"

# 포트 확인
lsof -i :3306
```

### 2. Docker에서 MySQL 연결 실패
```bash
# 네트워크 확인
docker network inspect be_moa-network

# 컨테이너 간 통신 테스트
docker exec moa-tomcat ping mysql
```

### 3. 프로필 확인
```bash
# Tomcat 로그에서 확인
docker-compose logs tomcat | grep "Active Profiles"
```

---

## 📝 주의사항

1. **비밀번호 통일**: 모든 환경에서 `1118` 사용 (개인 프로젝트)
2. **Firebase 키**: 서버에서 직접 생성 필요
3. **프로필 자동 설정**: Docker Compose가 자동으로 처리
4. **포트 충돌**: 로컬과 Docker 동시 실행시 포트 충돌 주의

---

## 🎯 요약

| 환경 | 설정 파일 | MySQL 호스트 | Redis 호스트 | 프로필 |
|------|----------|-------------|-------------|--------|
| 로컬 | application.properties | localhost | localhost | default |
| Docker | application-docker.properties | mysql | redis | docker |

설정이 자동으로 전환되므로 코드 수정 없이 환경별 배포 가능!
