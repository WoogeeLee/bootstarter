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
