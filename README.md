# bootstarter

# Build, Execution, Deployment > Build Tools > Gradle
# Run test using : Gradle > Intellij IDEA 로 변경

# 그래들 버전 다운그레이드
# alt + f12
# > gradlew wrapper --gradle-version 4.10.2
# https://console.cloud.google.com/
# https://developer.naver.com/apps/#/register?api=nvlogin

# EC2 세팅
용량 30G로 증설 후 다음- 30G까지 프리
보안 설정 - 22,443,8080 추가 후 인스턴트 업
탄력적 IP 추가 후 저장
(단, 추가 후 바로 EC2에 연결해야 한다.)
22는 ppk 파일 깃헙에 업로드시 바로 채굴에 악용될 수 있으니 조심!
puttygen, putty 다운로드
puttygen으로 pem -> ppk
후 putty로 접속
Host Name에 ec2-user@탄력적IP 입력 후
Connection -> SSH -> Auth 에서 ppk파일 세팅하고 접속 
(접속이 잘 안되서 ssh 보안 설정 IP 풀고 인스턴스 재부팅했음)

$ sudo yum install -y java-1.8.0-openjdk-devel.x86_64
$ sudo /usr/sbin/alternatives --config java
$ sudo yum remove java-1.7.0-openjdk
$ java -version

$ sudo rm /etc/localtime
$ sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
$ date

$ sudo vim /etc/sysconfig/network
HOSTNAME=woogee-springboot2-webservice
변경 후 저장
$ sudo reboot

$ sudo vim /etc/hosts
127.0.0.1   woogee-springboot2-webservice
추가 후 저장
$ curl woogee-springboot2-webservice
curl: (7) Failed to connect to woogee-springboot2-webservice port 80: Connection refused
나오는지 확인

# AWS RDB

할당 스토리지 20G
퍼블릭 액세스 가능 : 예
대시 안되는부분만 언더바로 변경하고 쭉 생성

세팅
타임존
캐릭터셋
Max Connection

좌측 - 파라미터 그룹
파라미터 그룹 생성
내가 생성한 DB 버전으로 woogee-springboot2-webservice 그룹 생성
파라미터 그룹 들어가서 파라미터 편집
time_zone : Asia/Seoul
character_set_client
character_set_connection
character_set_database
character_set_filesystem
character_set_results
character_set_server
collation_connection
collation_server
character는 utf8mb4 / collation는 utf8mb4_general_ci로 변경
utf8mb4는 이모지가 가능
max_connections 150
마지막으로 DB인스턴스 수정으로 파라미터 그룹 변경 - 즉시 적용
반영이 제대로 안되면 작업 -> 재부팅

데이터베이스 보안 그룹 선택
EC2에 사용된 보안그룹의 그룹ID 복사
복사된 보안 그룹의 ID와 본인의 IP를 RDS 보안 그룹의 인바운드로 추가

인텔리제이 Database에서 엔트포인트로 접속
캐릭터셋 등 잘 적용되었는지 확인

show variables like 'c%';

ALTER DATABASE woogee_springboot2_webservice
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';

select @@time_zone, now();

EC2에서 RDS로 접근 확인

$ sudo yum install mysql
$ mysql -u gogojet86 -p -h RDB호스트
비밀번호 입력하고 접속

# EC2 프로젝트 클론 받기
$ sudo yum install git

$ git --version

$ mkdir ~/app && mkdir ~/app/step1

$ cd ~/app/step1

$ git clone 내저장소https주소

$ cd 내프로젝트명

$ chmod +x ./gradlew

$ ./gradlew test

# 배포 스크립트 만들기

$ vim ~/app/step1/deploy.sh

#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1

PROJECT_NAME=bootstarter

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "> 현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &

=========================

$ vim /home/ec2-user/app/application-oauth.properties

로컬의 설정 옮겨서 저장 후

$ vim deply.sh

nohup java -jar -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties $REPOSITORY/$JAR_NAME 2>&1 &

(ㅅㅂ 역슬래시 필요없네..)

# RDS 테이블 생성

create table posts (id bigint not null auto_increment, create_date datetime, modify_date datetime, author varchar(255), content TEXT not null, title varchar(500) not null, primary key (id)) engine=InnoDB;

create table user (id bigint not null auto_increment, create_date datetime, modify_date datetime, email varchar(255) not null, name varchar(255) not null, picture varchar(255), role varchar(255) not null, primary key (id)) engine=InnoDB;

CREATE TABLE SPRING_SESSION (
                                PRIMARY_ID CHAR(36) NOT NULL,
                                SESSION_ID CHAR(36) NOT NULL,
                                CREATION_TIME BIGINT NOT NULL,
                                LAST_ACCESS_TIME BIGINT NOT NULL,
                                MAX_INACTIVE_INTERVAL INT NOT NULL,
                                EXPIRY_TIME BIGINT NOT NULL,
                                PRINCIPAL_NAME VARCHAR(100),
                                CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
                                           SESSION_PRIMARY_ID CHAR(36) NOT NULL,
                                           ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
                                           ATTRIBUTE_BYTES BLOB NOT NULL,
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
                                           CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

# Travis CI 배포 자동화

https://travis-ci.org/

깃헙으로 로그인 후, Settings 

올릴 프로젝트를 활성화

야믈파일 설정 - 그래들 파일과 같은 위치에

language: java
jdk:
  - openjdk8

branches:
  only:
    - master

#Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

#CI 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - jojoldu@gmail.com
      
아놔 실행권한이 또 없댄다.

야믈파일에 아래를 추가하거나

before_install:
  - chmod +x gradlew

git에서 아래와 같이 실행한다.

git update-index --chmod=+x gradlew

git ls-tree HEAD

git commit -m "permission access for travis"

# AWS Key 발급

IAM 사용자 추가

woogee-travis-deploy

프로그래밍 방식 액세스

기존정책 직접연결

s3full 검색, CodeDeployFull 검색해서 2개 추가 - 실제 서비스에서는 둘을 별도로 관리하기도 함

태그는 Name 정도 추가 / woogee-travis-deploy

확인 후 사용자 생성

그후 생성된 액세스 키 ID / 비밀 액세스 키를 travis에 등록

(More options > Settings > Environment Variables)

AWS_ACCESS_KEY , AWS_SECRET_KEY 를 Name으로 등록

# AWS S3 생성

버킷 이름 woogee-springboot2-build	

별다른 변경 없이 쭉 생성 - 퍼블릭 액세스 차단 (어차피 IAM으로 접근하므로)

그 후 travis 야믈파일에 아래와 같이 추가

before_deploy:
  - zip -r woogee-springboot2-webservice *
  - mkdir -p deploy
  - mv woogee-springboot2-webservice.zip deploy/woogee-springboot2-webservice.zip

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # travis settings 설정
    secret_access_key: $AWS_SECRET_KEY # travis settings 설정
    bucket: woogee-springboot2-build # S3 버킷
    region: ap-northeast-2 # 버킷 위치
    skip_cleanup: true
    acl: private # zip 파일 접근을 private로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deployed: true이 설정 추가

# travis ci와 aws s3, codedeploy 연동하기

AWS에서 IAM 검색하고 역할 -> 역할만들기(사용자는 외부서비스 / 역할은 내부서비스)

AWS 서비스 > EC2 선택

EC2RoleForA 검색 AmazonEC2RoleforAWSCodeDeploy 선택

Name / ec2-codedeploy-role

이렇게 만든 역할을 EC2 서비스에 등록

인스턴스 창에서 우클릭, 인스턴스 설정 > IAM 역할 연결/바꾸기

방금 생성한 역할로 바꾼 후 인스턴스 재부팅

# CodeDeploy 에이전트 설치

aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2

(성공시 : download: s3://aws-codedeploy-ap-northeast-2/latest/install to ./install)

install 파일에 실행권한 추가

chmod +x ./install

설치를 실행

sudo ./install auto

상태 검사

sudo service codedeploy-agent status

(상태 정상이면 : The AWS CodeDeploy agent is running as PID XXXX)

# IAM 에서 역할만들기

역할만들기 > AWS 서비스 > CodeDeploy 선택

권한이 하나뿐이니(AWSCodeDeployRole) 선택 후

Name : codedeploy-role

로 생성

# CodeDeploy 생성

이제 CodeDeploy로 넘어가서

시작하기 -> 애플리케이션 생성

이름 woogee-springboot2-webservice 입력 후 EC2/온프레미스 선택

배포 그룹 생성

woogee-springboot2-webservice-group

역할은 방금 생성한 역할

여러 대면 블루 그린이지만 단일 서버이므로 현재 위치

EC2 인스턴스 선택 후 Name : woogee-springboot2-webservice

에이전트 구성 안함 처리하고 (X) -> 한번만 구성하도록 변경

배포구성 CodeDeployDefault.AllAtOnce

로드밸런싱 비활성화

이대로 등록

# 각 설정 연동

먼저 EC2에서

$ mkdir ~/app/step2 && mkdir ~/app/step2/zip

travis는 .travis.yml , CodeDeploy는 appspec.yml 에 설정

결과적으로 travis의 빌드가 끝나면 S3에 파일이 전송되고, 이 zip파일은 ~/app/step2/zip 로 복사되어 압축을 풀 예정

appspec.yml 추가

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes

travis.yml deploy s3 아래 추가

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: woogee-springboot-build # S3 버킷
    key: woogee-springboot2-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip
    application: woogee-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 어플리케이션
    deployment_group: woogee-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true
    
그리고 커밋/ 푸시하니..

Version 2 of the Ruby SDK will enter maintenance mode as of November 20, 2020. To continue receiving service updates and new features, please upgrade to Version 3. More information can be found here: https://aws.amazon.com/blogs/developer/deprecation-schedule-for-aws-sdk-for-ruby-v2/

에러.. 루비를 업데이트하라고 한다.

위에 codedeploy에서 어플리케이션이랑 배포그룹 다시 생성하면서

에이전트를 설치하도록 변경해도 안된다..

끄어우.. 버켓명이 다르다..

bucket: woogee-springboot2-build

성공.

# 프로젝트에 배포 scripts 추가

/script 생성 후 deploy.sh 파일 생성

#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2

PROJECT_NAME=woogee-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl ${PROJECT_NAME} | grep jar | awk '{print $1}')

echo "> 현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties -Dspring.profiles.active=real $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &

.travis.yml 수정

before_deploy:
  - mkdir -p before-deploy
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy *
  - cd ../ && mkdir -p deploy
  - mv before-deploy/before-deploy.zip deploy/woogee-springboot2-webservice.zip

appspec.yml 수정

아래에 

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
      
커밋 & 푸시

또 안된다..






