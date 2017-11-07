# Dockerfile

## Dockerfile을 사용하여 이미지 빌드하기

Dockerfile은 도커 이미지를 빌드 시 필요한 명령어로 구성된 기본 DSL을 사용한다.

Dockerfile은 인수와 함께 여러 명령어들을 포함한다. 각 명령어들은 대문자가 되어야 하고 그 다음에 인수가 있어야 한다.

```dockerfile
FROM ubuntu:trusty
MAINTAINER Kim Joong Hyeon "karis0222@gmail.com"
RUN apt-get update
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo "root:passsword" | chpasswd
EXPOSE 22
```



## Dockerfile을 통한 이미지 빌드 워크플로우

- 도커는 이미지로부터 컨테이너를 실행한다.
- 명령어는 컨테이너에서 실행되고 변경 사항을 생성한다.
- 도커는 새로운 레이어를 커밋하기 위해 docker commit와 같은 기능을 실행한다.
- 그러고 나서 도커는 이 새로운 이미지로부터 새로운 컨테이너를 실행한다.
- 파일에 있는 다음 명령어가 실행되고 그 과정은 모든 명령어가 실행될 때까지 반복된다.

Dockerfile이 어떤 이유에서든지 중지된다면(예를 들어, 명령어가 실행에 실패했다면), 중지된 상태를 나중에 사용할 수 있도록 이미지로 남겨 둔다. 이 이미지는 디버깅할 때 매우 유용하다.

Dockerfile에 있는 첫 번째 명령어는 항상 `FROM`이 되어야 한다. FROM 명령어는 기존 이미지를 설정하여 그 다음에 나오는 명령어들이 동작할 수 있도록 한다. 이 이미지를 베이스 이미지라고 한다.

다음은 `MAINTAINER` 명령어를 설정한다. 이것은 이미지의 저작자가 누구인지 그리고 이메일 주소가 무엇인지 도커에게 알려준다. 이는 이미지의 소유권자와 컨택할 사람을 설정하는 데 유용하다.



## RUN

 명령어는 현재 이미지의 명령어를 실행한다. 이 명령어 각각은 새로운 레이어를 생성하고 성공한다면 그 레이어를 커밋하고 다음 명령어를 실행한다.
기본적으로 RUN 명령어는 명령어 래퍼인 `\bin\sh -c` 를 사용하여 쉘 안에서 실행된다. 쉘이 없는 플랫폼에서 명령어를 실행하거나 쉘 없이 실행하고 싶다면 exec 포맷에서 명령어를 사용하도록 설정할 수 있다.

exec 형태의 RUN 명령어 :
```dockerfile
RUN ["apt-get", "install", "-y", "openssh-server"]
```



## EXPOSE

 명령어는 컨테이너에 있는 애플리케이션이 컨테이너에 있는 특정 포트를 사용하도록 한다. 이것은 컨테이너에 있는 해당 포트에 실행하는 어떤 서비스라도 자동으로 액세스할 수 있도록 한다는 의미는 아니다. 보안상의 이유로 인해 도커는 포트를 자동으로 오픈할 수 없으며 docker run 커맨드를 사용하는 컨테이너를 실행할 때 오픈하도록 대기한다.



## Dockerfile로부터 이미지 빌드하기

docker build 커맨드를 실행할 때 모든 명령어들이 실행되고 커밋되며 결국 새로운 이미지가 리턴된다.
```bash
docker build -t="jamtur01/sshd:v1"
```
-t 플래그를 사용하여 도커허브의 리포지토리와 이미지 이름, 태그를 설정할 수 있다.

특정 스텝에서 빌드에 실패해도 각 스텝별로 RUN 명령어가 실행된 레이어가 커밋된 이미지가 생성되기 때문에 실패한 원인을 디버깅하는 용도로 사용할 수 있다.



## Dockerfile과 빌드 캐시

각 RUN 명령어에 대한 레이어를 빌드 캐시로 저장해두는데, 이를 통해 변경 사항이 없는 스텝은 빌드하지 않고 변경 사항이 있는 스텝부터 빌드 프로세스를 시작한다. 이렇게 하면 빌드할 때 많은 시간을 절약할 수 있다.
빌드 캐시를 사용하고 싶지 않다면(예를 들어 apt-get update와 같은 스텝) `docker build` 커맨드 사용 시 `--no-cache` 플래그를 사용하면 된다.



## 템플릿을 위한 빌드 캐시 사용하기

RUN 명령어를 사용하기 전, MAINTAINER 명령어 사용 후에 ENV 명령어를 사용하면 이미지에서 환경 변수를 설정할 수 있다. 이 경우 `REFRESHED_AT` 환경 변수를 설정하면 템플릿이 마지막으로 업데이트된 시점을 보여준다.
apt-get-qq update 명령어를 RUN 명령어에서 설정한다. 이 명령어는 실행될 때 APT 패키지 캐시를 리프레시하고, 최신 패키지가 설치될 준비가 됐다는 것을 보장한다.
템플릿을 사용하여, 빌드를 리프레시하고 싶을 때 ENV 명령어에 있는 날짜를 변경한다. 그러고 나면, 도커는 ENV 명령어를 히트했을 때 캐시를 리셋하고 캐시에 의존하지 않고 모든 그 이후의 명령어들을 실행한다.



## 새로운 이미지 보기

docker history 커맨드를 사용하여 이미지가 어떻게 생성됐는지 자세히 확인할 수 있다.



## 새로운 이미지에서 컨테이너 런칭하기

```bash
docker run -d -p 22 --name sshd 이미지명:태그
```
-p 플래그는 도커가 실행 중에 어떤 네트워크 포트를 오픈할 지를 관리한다. 컨테이너를 실행할 때 도커는 도커의 호스트에 포트를 할당하는 두 개의 메소드를 갖고 있다

도커는 컨테이너에서 포트 22로 매핑되어 있는 도커 호스트에 랜덤으로 49000~49900까지 높은 포트를 할당한다

컨테이너에서 포트 22로 매핑되어 있는 도커 호스트에 특정 포트를 할당할 수 있다.
위의 방법으로 사용하면 컨테이너의 포트 22에 연결될 도커 호스트에 랜덤 포트를 오픈한다. docker ps 커맨드를 사용해서 어떤 포트가 할당되었는지 확인할 수 있다.
```bash
docker port [컨테이너명] [포트번호]
# 해당 컨테이너의 해당 포트 정보 조회 명령어.

docker run -d -p 2222:22 --name sshd [이미지명:태그]
# 컨테이너 22 포트를 호스트의 2222 포트에 바인딩.
# 이와 같이 호스트의 특정 포트에 바인딩하는 것도 가능하다.

docker run -d -p 127.0.0.1:2222:22 --name sshd [이미지명:태그]
# 특정 인터페이스에 바인딩할 수도 있다. 위와 같이 하면 호스트의 127.0.0.1:2222 에 바인딩한다.
# 랜덤 포트에 바인딩하고자 하면 127.0.0.1::처럼 표기하면 되며,
# UDP 포트를 바인딩하고자 하면 127.0.0.1:2222/UDP 처럼 표기한다.

docker run -d -p -P --name sshd [이미지명:태그]
# 도커는 또한 숏컷을 갖고 있으며
# -P는 Dockerfile에서 EXPOSE 명령어를 통해 설정한 모든 포트를 오픈하도록 해준다.
```



# Dockerfile 명령어



## CMD

RUN 명령어와 비슷하지만, 컨테이너가 빌드될 때 명령어를 실행하기보다는 오히려 컨테이너를 런칭하는 docker run 커맨드를 통해 명령어를 설정하는 것과 비슷하다.

```dockerfile
CMD ["/bin/bash", "-l"]
```

명령어는 배열 안에 포함되어 있다. CMD 명령어를 배열 없이도 설정할 수 있는데, 이 경우 도커는 /bin/sh -c 명령어를 실행한 것과 동일하게 동작한다. 배열 신택스를 사용할 것을 권장한다.

docker run 커맨드를 사용하여 CMD 명령어를 오버라이드할 수 있다는 것을 이해하는 것이 중요. Dockerfile에서 CMD를 설정하고 docker run 커맨드 라인을 설정하면, 커맨드 라인은 Dockerfile의 CMD 명령어를 오버라이드하게 된다.

Dockerfile에서는 하나의 CMD 명령어만 설정할 수 있다. 두 개 이상의 CMD 명령어를 설정하면 마지막 CMD 명령어가 사용된다.



## ENTRYPOINT

CMD 명령어와 밀접한 관련이 있고 종종 혼동되기도 한다. docker run 커맨드 라인에 설정하는 인수는 ENTRYPOINT에서 설정한 커맨드에게 넘겨준다.
```dockerfile
ENTRYPOINT ["/usr/sbin/sshd", "-d"]
```
```bash
docker run -d [이미지명] -D
```
위와 같은 docker run 명령어를 실행하면 이미지명 뒤의 커맨드 라인 -D는 ENTRYPOINT 명령어에 설정된 커맨드로 넘겨지며 결국 /usr/sbin/sshd -D가 된다.

ENTRYPOINT와 CMD를 조합하여 사용할 수도 있다.
```dockerfile
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-T"]
```
런타임에 필요하면, ENTRYPOINT를 docker run 커맨드와 -entrypoint 플래그를 사용하여 오버라이드할 수 있다.



## WORKDIR

컨테이너를 위한 작업 디렉토리를 설정하는 방법을 제공. ENTRYPOINT나 CMD가 컨테이너가 이미지로부터 런칭될 때 실행되는 작업 디렉토리를 설정하는 방법 제공.

여러 개의 명령어나 혹은 최종 컨테이너를 위한 작업 디렉토리를 설정하는 데 사용하며, 특정 명령어를 위한 작업 디렉토리를 설정하기 위해서는 다음과 같다.
```dockerfile
WORKDIR /opt/webapp/db
RUN bundle install
WORKDIR /opt/webapp
ENTRYPOINT ["rackup"]
```
런타임 시 -w 플래그를 사용하여 작업 디렉토리를 오버라이드할 수 있다.
```bash
docker run -ti -w /var/log ubuntu pwd
```



## ENV

이미지를 빌드하는 과정 중 환경 변수를 설정하는 데 사용.
```dockerfile
ENV RVM_PATH /home/rvm/
RUN gem install unicorn
```
환경 변수들을 런타임 시 -e 플래그를 사용해서 docker run 커맨드로 전달할 수도 있다.
```bash
docker run -ti -e "WEB_PORT=8080" ubuntu env
```



## USER

이미지를 실행시키는 사용자를 설정.
```dockerfile
USER nginx
```
사용자 이름 혹은 UID를 설정한다. USER 명령어를 설정하지 않는다면 디폴트 유저는 기본적으로 root다.
docker run 커맨드와 함께 -u 플래그를 설정해서 오버라이드할 수 있다.



## VOLUME

이미지로부터 생성된 컨테이너에 볼륨을 추가한다. 볼륨은 유니온 파일 시스템(UFS)을 바이패스하는 하나 이상의 컨테이너 안에서 특별히 설계된 디렉토리다. 이 유니온 파일 시스템은 데이터를 유지하거나 공유하기 위해 몇 가지 유용한 기능을 제공한다.

- 볼륨은 컨테이너 사이에서 공유되고 재사용된다.
- 컨테이너는 볼륨을 공유하기 위해 실행될 필요는 없다.
- 볼륨에 대한 변경은 직접 적용된다
- 볼륨에 대한 변경은 이미지를 업데이트할 때는 포함되지 않는다.
- 볼륨은 컨테이너가 사용하지 않을 때까지 계속 유지된다.

데이터(소스 코드 등), 데이터베이스, 혹은 다른 컨텐츠를 이미지에 커밋하지 않고 이미지로 추가하게 해주며, 컨테이너 간에 데이터를 공유할 수 있게 해준다. 이는 컨테이너와 애플리케이션 코드, 관리 로그, 컨테이너 안의 데이터베이스 처리 등을 테스트하는 데 사용.
```dockerfile
VOLUME ["/opt/project"]
```



## ADD

빌드 환경에서 이미지로 파일과 디렉토리를 추가. 예를 들어 애플리케이션을 설치하는 경우. ADD 명령어는 파일에 대한 소스와 목적지를 설정한다.
```dockerfile
ADD sofrware.lic /opt/application/software.lic
```
파일이나 디렉토리는 빌드 디렉토리에 상대적 경로로 설정해야 한다. 소스는 파일이 될 수도 있고 URL이 될 수도 있다.

ADD 명령어는 로컬 tar 압축 파일을 처리할 수 있다. tar 압축 파일(gzip, bzip, xz를 포함한 압축 타입)이 소스 파일로 설정되며, 도커는 자동으로 이 압축 파일을 해제한다.
```dockerfile
ADD latest.tar.gz /var/www/wordpress/
```
목적 디렉토리 내에 같은 이름을 갖는 파일이나 디렉토리가 존재하면 덮어쓰지는 않는다.
목적지가 존재하지 않으면 도커는 디렉토리를 포함한 전체 경로를 생성한다. 새로운 파일과 디렉토리는 0755 모드로 생성되며 UID, GID는 0으로 생성된다.

빌드 캐시가 ADD 명령어에 의해 무효화될 수 있다는 것을 알고 있을 것. ADD 명령어의 변경에 의해 추가된 파일이나 디렉토리는 도커 파일에 있는 모든 명령어를 위한 캐시를 무효화한다.



## COPY

COPY 명령어는 ADD 명령어와 밀접한 관련이 있음. 가장 큰 차이점은 COPY 명령어는 빌드 컨텍스트에서 로컬 파일을 복사하는 작업에 초점을 맞추며 추출이나 압축 해제와 같은 기능은 갖고 있지 않다는 점.
```dockerfile
COPY conf.d/ /etc/apache2
```
파일의 소스는 빌드 컨텍스트에 상대적인 디렉토리나 파일의 경로가 되어야 한다. 또한 빌드 컨텍스트는 Dockerfile이 존재하는 위치의 로컬 소스 디렉토리. 이 디렉토리 밖에 있는 것은 복사할 수 없다. 목적지는 컨테이너 내부에 있는 절대 경로가 되어야 한다.

복사를 해서 생성한 파일이나 디렉토리는 UID, GID를 0으로 갖게 되며 소스가 디렉토리면 전체 디렉토리가 복사되고 이 디렉토리는 파일 시스템의 메타 데이터도 포함된다. 소스가 파일 종류라면 그 파일의 메타 데이터와 함께 개별적으로 복사된다.

목적지가 존재하지 않으면 경로에 있는 존재하지 않는 새로운 디렉토리를 생성한다.



## ONBUILD

이미지에 트리거를 추가. 트리거는 이미지가 다른 이미지의 베이스로 사용될 때 실행된다.
예를 들면 사용할 수 없는 특정 위치에 추가된 소스 코드가 필요한 이미지를 갖고 있을 때 혹은 이미지가 빌드된 환경의 빌드 스크립트를 실행할 필요가 있는 경우 등.

트리거는 새로운 명령어가 FROM 명령어 다음에 위치하도록 설정하는 것처럼 새로운 명령어를 빌드 과정에 삽입한다. 트리거는 어떤 빌드 명령어도 될 수 있다.
```dockerfile
ONBUILD ADD . /app/src
ONBUILD RUN cd /app/src && make
```
ONBUILD 트리거는 부모 이미지에 설정된 순서에 따라 실행되며 한 번은 상속할 수 있다(예를 들어 자식은 되지만 손자는 될 수 없다).

ONBUILD에서 사용할 수 없는 몇 가지 명령어 : FROM, MAINTAINER, ONBUILD. Dockerfile 빌드에서는 자신을 호출하는 재귀 호출을 사용할 수 없기 때문이다.