# be-airdnb
* 코드스쿼드 마스터즈 2024 BE 프로젝트 숙박앱
* 해당 [Repo](https://github.com/codesquad-members-2024/be-airdnb)에서 Team04, 그로밋으로 참여하였습니다.
## 😎팀원 소개
|<img src="https://avatars.githubusercontent.com/u/136168660?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/87357932?v=4" width="150" height="150"/>|
|:-:|:-:|
|Ahn JongHyeok<br/>[@jonghyeok-97](https://github.com/jonghyeok-97)|yeriimii<br/>[@Yeriimii](https://github.com/Yeriimii)|

## 맡았던 사안
### 1. 성능테스트를 위한 20만건 더미 데이터를 csv 파일을 사용하여 DB Connection timeout -> 7초로 줄이기
- 전문 Blog 링크(https://dkswhdgur246.tistory.com/47)
1. 문제
- 성능 테스트를 위해 20만건의 더미 데이터를 생성했지만 Stream 으로 돌면서 DB에 Insert or InsertAll 을 하면 DB Connection timeout 이 생기는 문제 발생

2. 고민
- Spring Batch 를 학습하여 일괄적 작업하기
- 테이블은 파일임을 이용해 csv파일 사용하기

- 프로젝트 남은 기간을 고려해서 러닝커브가 낮은 csv파일을 사용하는 방법을 선택
  
3. 과정
- csv파일을 만드는 외부 라이브러리 의존성 추가
- @PostConstruct 를 사용
- 만든 csv파일을 Mysql 가 있는 Docker 컨테이너로 복사
- MySql 서버에서 csv파일의 데이터를 읽어 테이블(파일)에 삽입

### 2. AWS, Docker, Github Actions 를 활용한 서버 배포 및 CI/CD 구축
#### 서버 배포
- EC2
  - OS: Ubuntu, JDK: 17, t2.micro 사용  
  - 인바운드 포트 SSH, HTTP, HTTPS
  - 아웃바운드 3306 포트를 RDS IP주소와 연결
- RDS
  - 인바운드 3306 포트를 EC2 의 IP주소만 허용
  - 모든 아웃바운드 포트 닫음
 
#### CI/CD 구축  
- release 브랜치에 push 가 되면 workflows/deploy.yml 이 실행
- IAM Role 을 활용해 EC2가 ECR에 접근할 수 있도록 권한 부여
- 사이드 프로젝트에 간단하게 적용할 수 있지만, 서버가 확장되거나 무중한 배포에 적용하기는 어렵다는 단점이 있음
#### 전체적인 흐름
![배포 drawio](https://github.com/user-attachments/assets/faacaf1e-9224-4559-bc19-a239b4d0b219)

### 3. 동시 예약 막기
- 예약을 DB에 저장하는 insert 쿼리에는 비관적 락과 낙관적 락을 사용할 수 없음을 트랜잭션 격리 수준을 통해 학습 [Blog 링크](https://dkswhdgur246.tistory.com/44)
- 트랜잭션 격리수준을 SERIALIZABLE 로 하기에는 에어비앤비 서비스 특성상 숙소 조회에 많은 DB커넥션이 사용될것이며, 트랜잭션을 순차적으로 실행하기에는 적합하지 않다고 판단
- DB 유니크 제약 조건을 사용하여 동시 예약 요청을 막았습니다. [Blog 링크](https://dkswhdgur246.tistory.com/46)
