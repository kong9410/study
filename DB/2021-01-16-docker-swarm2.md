---
title: 도커 스웜(Docker Swarm) #2
tags: docker
---

## 스웜 모드 서비스

`docker run` 명령어는 컨테이너를 생성하는 것처럼 도커 클라이언트에서 사용하는 명령어가 제어하는 것은 컨테이너이다. 스웜 모드에서 제어하는 단위는 컨테이너가 아닌 서비스다.

서비스는 같은 이미지에서 생성된 컨테이너의 집합이며, 서비스를 제어하면 해당 서비스 내의 컨테이너에 같은 명령이 수행된다. 서비스 내에 컨테이너는 1개 이상 존재하며 컨테이너는 워커 노드, 매니저 노드에 할당된다. 이러한 컨테이너들을 테스크(Task)라고 한다.

예를 들어 ubuntu 이미지로 서비스를 생성하고 컨테이너 수를 3개로 설정했다면, 스웜 스케줄러가 서비스의 정의에 따라 컨테이너를 할당할 적합한 노드를 선정하고, 해당 노드에 컨테이너를 분산해서 할당한다. 각 노드에 컨테이너가 할당되지 않을 수도 있고 이처럼 함께 생성된 컨테이너를 레플리카라고 한다. 서비스에 설정된 레플리카의 수만큼의 컨테이너가 스웜 클러스터내에 존재해야한다.

스웜은 서비스의 컨테이너들에 대한 상태를 계속 확인하고 있다가 서비스 내에 정의된 레플리카의 수만큼 컨테이너가 스웜 클러스터에 존재하지 않으면 새로운 컨테이너 레플리카를 생성한다. 노드 서버가 다운된다면 다운되지 않은 다른 노드에 부족한 컨테이너 수만큼 새로 할당한다. 컨테이너가 일부 작동하지 않은 상태면 이것도 레플리카의 수를 충족하지 못해 스웜 매니저가 새로운 컨테이너를 클러스터에 새롭게 생성한다.

서비스는 롤링 업데이트 기능도 제공한다. 서비스 내 컨테이너들의 이미지를 일괄적으로 업데이트해야 할 때 컨테이너들의 이미지를 순서대로 변경해 서비스 자체가 다운되는 시간 없이 컨테이너의 업데이트를 진행할 수 있습니다.

### 서비스 생성

서비스를 제어하는 도커 명령어는 전부 매니저 노드에서만 사용할 수 있다.

#### step 1 - 서비스 생성해보기

서비스를 생성하려면 `docker service create` 명령어를 사용한다. 컨테이너가 시작할때 hello world를 출력하는 셀 명령어를 설정했다.

```shell
docker service create \
ubuntu:14.04 \
/bin/sh -c "while true; do echo hello world; sleep 1; done"
```

> 서비스 컨테이너는 detached 모드로 사용할 수 있는 이미지를 선택해야한다.
>
> ```shell
> docker service create ubuntu:14.04
> ```
>
> 이와 같이 생성한다면 컨테이너 내부를 차지하고 있는 프로세스가 없어 컨테이너가 정지될 것이고, 스웜 매니저는 서비스의 컨테이너에 장애가 생긴 것으로 판단해 컨테이너를 반복 생성할 것이다.

서비스가 정상적으로 작동하는지 확인하기 위해 `docker service ls`로 확인한다. 서비스이름은 따로 지정하지 않았기 때문에 무작위로 생성된다. 더 자세한 정보를 아려면 `docker service ps [서비스 이름]`으로 입력한다.

```shell
docker service ls
ID             NAME                    MODE         REPLICAS   IMAGE          PORTS
yljr77ir9kqg   compassionate_swanson   replicated   1/1        ubuntu:14.04

docker service ps compassionate_swanson
ID             NAME                      IMAGE          NODE            DESIRED STATE   CURRENT STATE           ERR
OR     PORTS
huycj6r46zle   compassionate_swanson.1   ubuntu:14.04   swarm-manager   Running         Running 5 minutes ago 
```

생성된 서비스를 삭제하려면 `docker service rm` 명령어를 사용한다. 서비스의 상태에 관계 없이 바로 삭제가 된다.

### step 2 - 레플리카 설정해보기

step1은 레플리카를 따로 설정하지 않았기 때문에 1개의 컨테이너만 생성되었다. 이번엔 `--replica` 옵션을 추가해 서비스를 외부에 노출해본다.

```shell
docker service create --name myweb --replicas 2 -p 80:80 nginx
```

myweb이라는 서비스의 이름으로 2개의 레플리카를 가진 80포트로 이어진 nginx 이미지 컨테이너를 생성하였다.

```shell
docker service ps myweb
ID             NAME      IMAGE          NODE            DESIRED STATE   CURRENT STATE            ERROR     PORTS
opcn01c2zy7g   myweb.1   nginx:latest   swarm-worker1   Running         Running 11 seconds ago             
doamk3kilqm0   myweb.2   nginx:latest   swarm-manager   Running         Running 13 seconds ago
```

ps 명령어로 확인해보면 myweb 서비스의 node가 각 매니저와 워커에 할당된 것을 확인할 수 있다. 반드시 nginx 웹 서버에 접근하려면 해당 노드의 IP로 접근해야 하는것은 아니다. docker swarm의 80포트를 개방함으로 스웜 클러스터내의 어떤 노드로 접속해도 웹 서버에 접근할 수 있다. 여기서 할당되지 않은 swarm-worekr2의 ip로 접근해보면 서비스에 접근할 수 있다는 것을 확인할 수 있다.

`scale` 명령어를 사용하면 레플리카의 수를 늘리거나 줄일 수가 있다

```shell
docker service scale myweb=4
myweb scaled to 4

docker service ps myweb
ID             NAME      IMAGE          NODE            DESIRED STATE   CURRENT STATE           ERROR     PORTS
opcn01c2zy7g   myweb.1   nginx:latest   swarm-worker1   Running         Running 8 minutes ago             
doamk3kilqm0   myweb.2   nginx:latest   swarm-manager   Running         Running 8 minutes ago             
eoow1k2q6c2d   myweb.3   nginx:latest   swarm-worker2   Running         Running 9 seconds ago             
nnc4r8mxng7z   myweb.4   nginx:latest   swarm-worker2   Running         Running 9 seconds ago 
```

worker2에 2개의 컨테이너가 새로 할당된 것을 확인할 수 있다. 컨테이너가 각 호스트의 80포트에 연결된 것은 아니고, 실제로는 각 노드의 80번 포트로 들어온 요청을 위 4개 컨테이너 중 1개로 리다이렉트 시킨다. 라운드 로빈 방식으로 컨테이너를 결정하는데, 만일 각 노드의 트래픽이나 자원 사용량을 고려하면 적절한 방식은 아닐것이다.

### step 3 - global 서비스 생성해보기

서비스의 모드는 두가지가 있다. 하나는 위 step 처럼 복제모드로 실제 서비스를 제공하기 위해 일반적으로 쓰이는 모드다. 두번째는 글로벌 모드이다. 글로벌 서비스는 클러스터 내의 사용할 수 있는 모든 노드에 컨테이너를 반드시 한개를 할당한다. 그렇기 때문에 별도의 레플리카 수를 지정하지 않는다. 글로벌 서비스는 클러스터를 모니터링하기 위해 에이전트 컨테이너 등을 생성해야 할 때 유용하다.

```shell
docker service create --name global_web --mode global nginx

docker service ps global_web
ID             NAME                                   IMAGE          NODE            DESIRED STATE   CURRENT STATE 
           ERROR     PORTS
s3dl3xjio9r6   global_web.1rqva6n2xizgjgt7bs2358znq   nginx:latest   swarm-worker2   Running         Running 13 sec
onds ago             
tgu6dxfg4i17   global_web.ewkgh1qli9wvpayxaeqbikidh   nginx:latest   swarm-manager   Running         Running 12 sec
onds ago             
vtllp87dck5l   global_web.lxqz76lzffifj25um47ebgx19   nginx:latest   swarm-worker1   Running         Running 12 sec
onds ago    
```

`--mode global` 옵션을 추가하여 생성할 수 있다.

## 서비스 장애 복구

서비스의 특정 컨테이너가 정지하거나 특정 노드가 다운되면 스웜 매니저는 새로운 컨테이너를 생성해 자동으로 복구한다. myweb 서비스의 컨테이너 하나를 삭제해본다.

```shell
docker rm -f myweb.2.doamk3kilqm0i9k9eqabbf5rq

docker service ps myweb
ID             NAME          IMAGE          NODE            DESIRED STATE   CURRENT STATE            ERROR                         PORTS
opcn01c2zy7g   myweb.1       nginx:latest   swarm-worker1   Running         Running 26 minutes ago                                 
5sydw50zn2re   myweb.2       nginx:latest   swarm-manager   Running         Running 6 minutes ago                                  
doamk3kilqm0    \_ myweb.2   nginx:latest   swarm-manager   Shutdown        Failed 6 minutes ago     "task: non-zero exit (137)"   
eoow1k2q6c2d   myweb.3       nginx:latest   swarm-worker2   Running         Running 17 minutes ago                                 
nnc4r8mxng7z   myweb.4       nginx:latest   swarm-worker2   Running         Running 17 minutes ago
```

> docker ps 명령어로 생성된 컨테이너를 확인할 수 있으며 `--filter is-task=true`를 추가하면 스웜 모드의 서비스에서 생성된 컨테이너만 출력할 수 있다.

삭제 후 컨테이너 목록을 확인해 보면 새로운 컨테이너가 생성됐음을 확인할 수 있다. 특정 노드가 다운되었을 때도 확인할 수 있다.

worker1에서 노드의 도커 데몬 프로세스를 종료해 임의로 노드 장애 상태를 만들어 본다.

```shell
service docker stop
docker stop/waiting
```

그리고 manager에서 다시 노드의 상태를 확인해본다

```shell
docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Down      Active                          20.10.2
1rqva6n2xizgjgt7bs2358znq     swarm-worker2   Ready     Active                          20.10.2
```

worker1이 Down 상태인 것을 확인할 수 있다. 서비스의 컨테이너 목록을 다시 확인해본다.

```shell
docker service ps myweb
ID             NAME          IMAGE          NODE            DESIRED STATE   CURRENT STATE            ERROR                         PORTS
2m561zeospe1   myweb.1       nginx:latest   swarm-manager   Running         Running 2 minutes ago                                  
opcn01c2zy7g    \_ myweb.1   nginx:latest   swarm-worker1   Shutdown        Running 36 minutes ago                                 
5sydw50zn2re   myweb.2       nginx:latest   swarm-manager   Running         Running 16 minutes ago                                 
doamk3kilqm0    \_ myweb.2   nginx:latest   swarm-manager   Shutdown        Failed 16 minutes ago    "task: non-zero exit (137)"   
eoow1k2q6c2d   myweb.3       nginx:latest   swarm-worker2   Running         Running 27 minutes ago                                 
nnc4r8mxng7z   myweb.4       nginx:latest   swarm-worker2   Running         Running 27 minutes ago 
```

worker 1의 컨테이너들이 Shutdown 되었고 복구하기 위해 manager에 새 컨테이너가 할당된 것을 확인할 수 있다. 다운 되었던 노드를 다시 정상구동해도 다른 노드로 옮겨진 컨테이너가 해당 노드에 자동으로 할다오디지 않는다. 다시 복구했을 때는 scale 명령어를 이용해 컨테이너의 수를 줄이고 다시 늘려야한다.

```shell
docker service scale myweb=1
docker service scale myweb=4
```

## 롤링 업데이트

롤링 업데이트를 테스트하기 위한 서비스를 먼저 생성한다.

```shell
docker service create --name myweb2 --replicas 3 nginx:1.10
```

서비스가 정상적으로 생성되면 `docker service update` 명령어로 서비스의 각종 설정을 변경할  수있다. `--image` 옵션을 사용하면 이미지 업데이트를 할 수 있다. 다음은 myweb2 서비스의 이미지를 nginx:1.1로 업데이트한다.

```shell
docker service update --image nginx:1.11 myweb2
```

이후 서비스의 컨테이너 목록을 확인하면 각 컨테이너의 이미지가 변경된 것을 알 수 있다. 업데이트 대상이 되어 삭제된 컨테이너는 name 앞에 `\_`가 붙게된다. 그렇지 않은 컨테이너는 롤링 업데이트로 새롭게 생성된 컨테이너다.

서비스를 생성할 때 롤링 업데이트의 주기, 컨테이너 수, 실패했을 때 대처 등을 설정할 수 있다.

```shell
docker service create \
--replicas 4 \
--name myweb3 \
--update-delay 10s \
--update-parallelism 2 \
nginx:1.10
```

위 설정은 레플리카를 10초 단위로 업데이트 하며 업데이트 작업을 한번에 2개 컨테이너에 수행한다는 것을 의미한다. 이를 설정하지 않으면 주기 없이 차례대로 컨테이너를 한 개씩 업데이트한다.

위와 같은 서비스의 롤링 업데이트 설정은 `docker service inspect` 또는 `docker inspect --type service` 명령어로 확인할 수 있다. 업데이트 실패에 대해 아무런 설정도 하지 않으면 On failure 항목은 pause로 설정되지만 서비스를 생성할 때 `--update-failure-action` 인자 값을 continue로 설정해 업데이트 오류가 발생해도 계속 롤링 업데이트를 진행하게 할 수 있다.

```shell
docker service create --name myweb4 \
--replicas 4 \
--update-failure-action continue \
nginx:1.10
```

롤링 업데이트는 서비스 자체에 설정돼 있지만, docker service update 명령어의 옵션 값을 다르게 설정함으로써 변경할 수 있다. 서비스 롤링 업데이트 후, 서비스를 롤링 업데이트 전으로 되돌리는 롤백(rollback) 또한 가능하다.

```shell
docker service rollback myweb3
```

### 서비스 컨테이너에 설정 정보 전달하기

애플리케이션을 외부에 서비스하려면 환경에 맞춘 설정 파일이나 값들이 컨테이너 내부에 미리 준비돼 있어야 한다. 정적으로 사용할 수도 있지만 클러스터에서 파일 공유를 위해 설정 파일을 호스트마다 마련해두는 것은 비효율 적인 일이다. 이를 위해 스웜 모드는 secret와 config 기능을 제공한다. secret은 비밀번호나 SSH 키, 인증서 키와 같이 보안에 민감한 데이터를 전송하기 위해, config는 nginx나 레지스트리 설정 파일 등의 설정값에 대해 쓰일 수 있다.

#### secret

secret을 사용하려면 `docker secret create` 명령어를 사용한다.

```shell
echo q12w3e4r | docker secret create my_mysql_password -
```

이 명령어는 my_mysql_password라는 이름의 secret에 1q2w3e4r 값을 저장한다. 그리고 `docker secret ls`와 `docker secret inspect my_mysql_password` 로 생성된 secret을 확인할 수 있다. 그러나 조회해도 실제 값은 확인할 수 없다. secret 값은 매니저 노드 간에 암호화된 상태로 저장된다.

secret을 통해 MySQL 컨테이너를 생성해본다.

```shell
docker service create \
--name mysql \
--replicas 1 \
--secret source=my_mysql_password,target=mysql_root_password \
--secret source=my_mysql_password,target=mysql_password \
-e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
-e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
-e MYSQL_DATABASE="wordpress" \
mysql:5.7
```

--secret 옵션을 통해서 MYSQL 사용자 비밀번호를 컨테이너 내부에 마운트하고 source에 secret 이름을 입력하고 target에는 컨테이너 내부에서 보여질 secret이름을 입력한다.

secret 옵션으로 컨테이너로 공유된 값은 컨테이너 내부의 /run/secrets/ 디렉터리에 마운트된다. target에 각각 mysql_root_password, mysql_password로 설정되었기 때문에 /run/secret 디렉터리에 해당 이름의 파일이 각각 존재할 것이다.

이러한 방식은 컨테이너 내부의 특정 경로에서 참조를 할수있도록 설계를 해야한다는 단점이 있다. 각종 설정 변수를 파일로부터 동적으로 읽어올 수 있도록 설계하면 secret, config의 장점을 활용할 수 있다.

#### config

secret과 사용방법이 거의 동일하다. 

우선 사용할 config파일을 먼저 작성한다

```shell
vi config.yml

version:0.1
log:
  level: info
storage:
  filesystem:
    rootdirectory: /registry_data
  delete:
    enabled: true
http:
  addr: 0.0.0.0:5000
```

그리고 도커로 registry-config라는 이름의 config로 저장한다.

```shell
docker config create registry-config.yml
```

```shell
docker config ls
```

이 config를 가지고 사설 레지스트리를 생성해본다. 명령어에 `--config`를 추가하되, 나머지는 secret과 동일하다.

```shell
docker service create --name yml_registry -p 5000:5000 \
--config source=registry-config,target=/etc/docker/registry/config.yml
registry:2.6
```

만일 이후에 새로운 값을 사용해야 한다면 `docker service update` 명령어의 `--config-rm` `--config-add` `--secret-rm` `--secret-add` 옵션을 사용해 추가하거나 삭제할 수 있다.

# 출처

시작하세요! 도커/쿠버네티스(개정판), 용찬호 지음