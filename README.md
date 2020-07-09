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

nohup java -jar \ -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \ $REPOSITORY/$JAR_NAME 2>&1 &

