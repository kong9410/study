---
title: 도커 스웜(Docker Swarm) #1
tags: docker
---

# 도커 스웜

하나의 호스트 머신에서 도커엔진을 구동하다가 cpu나 메모리 등이 부족할 수도 있다. 이를 해결하는 방법 중 제일 간단한 것은 더 좋은 성능의 서버를 새로 사는 것이다. 그러나 확장성 측면이나 비용 측면에서는 그렇게 좋은 방법은 아니다. 가장 좋은 방법은 여러 대의 서버를 클러스터로 만들어 자원을 병렬로 확장하는 것이다.

그러나 여러 대의 서버를 하나의 자원 풀로 만드는 것은 쉬운 일이 아니다. 새로운 서버나 컨테이너가 추가되었을 때 이를 발견하는 작업부터 어떤 서버에 컨테이너를 할당 할 것인가에 대한 스케줄러와 로드밸런서 문제, 클러스터 내의 서버가 다운 됐을 때 고 가용성을 어떻게 보장할지 등이 문제로 남아있다. 이러한 문제를 해결하는 방법 중 대표적인것이 도커에서 공식으로 제공하는 도커 스웜이다.

## 스웜 클래식과 도커 스웜 모드

스웜 클래식과 스웜 모드는 여러 대의 도커 서버를 하나의 클러스터로 만들어 컨테이너를 생성하는 여러 기능을 제공한다. 다양한 전략을 세워 컨테이너를 특정 도커 서버에 할당할 수 있고 유동적으로 서버를 확장할 수 있다. 스웜 클러스터에 등록된 서버의 컨테이너를 쉽게 관리할 수 있다.

도커 스웜에는 두가지 종류가 있다. 하나는 도커 버전 1.6 이후부터 사용할 수 있는 컨테이너로서의 스웜(스웜 클래식)이고, 다른 하나는 도커 버전 1.12 이후부터 사용할 수 있는 도커 스웜 모드이다.

### 스웜 클래식과 스웜 모드의 차이

|          | 스웜 클래식                                                  | 스웜 모드                                                    |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 목적     | 여러 도커 서버를 하나의 접근점에서 사용                      | 마이크로서비스 아키텍처를 다룸                               |
| 기능     | 일반적인 도커 명령어와 도커 API로 클러스터의 서버를 제어하고 관리 | 같은 컨테이너를 동시에 여러 개 생성해 필요에 따라 유동적으로 컨테이너의 수를 조절<br />컨테이너로의 연결을 분산하는 로드밸런싱 기능 사용 |
| 클러스터 | 분산 코디네이터, 에이전트 등이 별도 실행                     | 클러스터링을 위한 모든 도구가 엔진 자체에 내장되어 있음      |

> 분산코디네이터 : 클러스터에 영입할 새로운 서버의 발견, 각종 설정 저장, 데이터 동기화 등에 주로 이용된다. etcd, zookeeper, consul 등이 대표적이다.

대규모 클러스터에는 스웜 모드를 사용하는 것이 좋다. 스웜 모드는 마이크로 아키텍쳐 뿐만 아니라 서비스 장애에 대한 고가용성과 부하 분산을 위한 로드밸런싱 기능 또한 제공하고 있다.

## 스웜 모드

스웜 모드는 별도의 설치과정이 필요하지 않고 도커 엔진 자체에 내장되어 있다. docker info 명령어를 통해 도커 엔진의 스웜 모드 클러스터 정보를 확인할 수 있다.

```shell
docker info | grep Swarm
Swarm: inactive
```

### 도커 스웜 모드의 구조

스웜 모드는 매니저 노드와 워커로 구성되어 있다.

- 워커 노드 : 실제로 컨테이너가 생성되고 관리되는 도커 서버이다.
- 매니저 노드 : 워커 노드를 관리하기 위한 도커 서버이다. 매니저 노드 또한 컨테이너가 생성될 수 있다.

매니저 노드는 1개 이상이어야 하지만 워커 노드는 없을 수도 있다. 매니저 노드는 다중화하는 것을 권장한다. 매니저의 부하를 분산하고 특정 매니저 노드가 다운됐을 때 정상적으로 스웜 클러스터를 유지할 수 있기 때문이다.

스웜 모드는 매니저 노드의 절반 이상이 문제가 생겼을 때 매니저 노드가 복구될 때까지 클러스터의 운영을 중단한다. 매니저 노드 사이에 네트워크 파티셔닝 같은 현상이 발생했을 경우 짝수 개의 클러스터는 운영이 중단될 수 있다. 홀수 개로 구성했을 경우에는 과반수 이상 유지되는 쿼럼 매니저에서 운영을 계속할 수 있기 때문에 스웜 매니저는 홀수 개 이상으로 구성하는 것이 권장된다.

## 도커 스웜 모드 클러스터 구축

매니저 역할을 할 서버에서 스웜 클러스터를 시작한다. --advertise-addr에 다른 도커 서버가 매니저 노드에 접근하기 위한 IP 주소를 입력한다.

```shell
docker swarm init --advertise-addr 35.184.69.63
```

두개 이상의 네트워크 인터페이스가 있을 경우 어느 IP 주소로 매니저에 접근해야 할지 다른 노드에 알려주어야한다. 출력 결과 중 `docker swarm join` 명령어는 새로운 워커 노드를 스웜 클러스터에 추가할 때 사용된다. `--token` 옵션에 사용된 토큰 값은 새로운 노드를 해당 스웜 클러스터에 추가하기 위한 비밀 클러스터이다

워커로 사용할 서버에서 join 명령어를 실행한다

```shell
docker swarm join --token SWMTKN-1-23h01ug2x2gr333i9dn2h21ln5zezjg80hse4yarkdhozdi4lt-c96vr7g76vtbqadgftwjunn9z 35.184.69.63:2377 
```

정상적으로 스웜 클러스터에 추가되었는지 확인하려면 매니저 노드에서 docker node ls 명령어를 수행한다

```shell
docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Ready     Active                          20.10.2
yx2h31i8fz57j0rbwry5gl9u0     swarm-worker2   Ready     Active                          20.10.2
```

새로운 매니저 노드를 사용하려면 매니저 노드를 위한 토큰을 사용해 `docker swarm join` 명령어를 사용한다. 매니저 노드를 추가하기 위한 토큰은 `docker swarm join-token manager` 명령어로 확인할 수 있다. 워커 역시 `docker swarm join-token worker`로 확인할 수 있다. 토큰은 외부에 노출되면 누구나가 쉽게 클러스터에 추가할 수가 있기 때문에 외부 노출에 조심해야한다.

토큰을 갱신하려면 `swarm join` 명령어에 `--rotate` 옵션을 추가하고 변경할 토큰의 대상을 입력하면 된다. 이 작업은 매니저 노드에서만 수행할 수 있다.

```shell
docker swarm join-token --rotate manager
```



추가된 워커 노드를 삭제하고 싶으면 해당 워커 노드에서 `docker swarm leave` 명령어를 입력한다.

```shell
docker swarm leave
Node left the swarm.
```

그러나 도커 leave 명령어로 스웜 모드를 삭제하면 매니저 노드는 해당 워커 노드의 상태를 down으로 인지할 뿐 자동으로 워커 노드를 삭제하지 않는다. 따라서 매니저 노드에서 docker node rm 명령어를 사용해 해당 워커 노드를 삭제해야한다.

```shell
docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Ready     Active                          20.10.2
yx2h31i8fz57j0rbwry5gl9u0     swarm-worker2   Down      Active                          20.10.2

docker node rm swarm-warker2
swarm-worker2

docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Ready     Active                          20.10.2
```

> 컨테이너의 이름이 아닌 ID의 일부를 사용하여 제어할 수 있다. 위의 docker node rm의 경우 다음 처럼 쓸수가 있다.
>
> ```shell
> docker node rm lxqz
> ```

매니저 노드는 `docker swarm leave --force`를 사용해야 삭제할 수 있다. 매니저 노드를 스웜 클러스터에서 삭제하면 해당 매니저 노드에 저장돼 있던 클러스터의 정보도 삭제된다.

워커 노드를 매니저 노드로 변경하려면 `docker node promote` 명령어를 사용한다. 반대로 매니저 노드를 워커 노드로 변경하려면 `docker node demote` 명령어를 사용하면 된다. 매니저 노드가 1개일 때 매니저 노드에 대한 `demote` 명령어는 사용할 수 없다.

 ```shell
docker node promote swarm-worker1
Node swarm-worker1 promoted to a manager in the swarm.

docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Ready     Active         Reachable        20.10.2
 ```

```shell
docker node demote swarm-worker1
Manager swarm-worker1 demoted in the swarm.

docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ewkgh1qli9wvpayxaeqbikidh *   swarm-manager   Ready     Active         Leader           20.10.2
lxqz76lzffifj25um47ebgx19     swarm-worker1   Ready     Active                          20.10.2
```

