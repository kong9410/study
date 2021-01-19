---
title: 도커 스웜(Docker Swarm) #4
tags: docker
---

## 노드 관리

### 노드 AVAILABILTY 변경하기

스웜 노드를 확인하면 다음과 같은 결과를 확인할 수 있다.

```shell
docker node ls

ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
loyay0tci9tld4yfmdsqstrih *   swarm-manager   Ready     Active         Leader           20.10.2
sagxhrre8zg1l0w2mo9x87kg3     swarm-worker1   Ready     Active                          20.10.2
voeiy82lar5g73xgbcukjmkb6     swarm-worker2   Ready     Active                          20.10.2
```

모든 노드의 AVAILABILTY가 Active인 것을 확인 할 수있다. 그러나 매니저 노드같은 경우 최대한 부하를 안받게 한다던가 특정 노드에 컨테이너를 할당하지 않게 하고싶을 수가 있다. 이를 위해 특정 노드의 AVAILABILITY를 설정할 수 있다

#### Active

Active는 스웜 클러스터에 추가되면 기본으로 설정되는 상태다. Active가 아닌 노드를 Active 상태로 변경하려면 `docker node update` 명령어를 사용한다.

```shell
docker node update \
--availability active \
swarm-worker1
```

#### Drain

Drain 상태로 설정하면 스웜 매니저의 스케줄러는 컨테이너를 해당 노드에 할당하지 않는다. 일반적으로 매니저 노드에 설정한다. Drain 상태가 되면 해당 노드에서 실행 중이던 컨테이너는 전부 중지되고 Active 상태의 노드에서 다시 할당된다.

```shell
docker node update \
--availability drain \
swarm-worker1
```

#### Pause

Pause 상태는 서비스의 컨테이너를 더는 할당받지 않는다는 점에서 Drain과 같지만 실행 중인 컨테이너가 중지되지 않는다.

```shell
docker node update \
--availability pause \
swarm-worker1
```

### 노드 라벨 추가

노드에 라벨을 추가하는 것은 노드를 분류하는 것과 비슷하다. 라벨은 키-값 형태를 갖는다. 라벨을 추가하면 노드의 특정 그룹을 선택할 수 있다. 예를들어 ssd를 사용하는 노드에만 컨테이너를 할당하게 할 수 있다.

#### 노드 라벨 추가하기

swarm-worker1 노드의 라벨을 storage=ssd로 설정한다.

```shell
docker node update \
--label-add storage=ssd \
swarm-worker1
```

설정된 라벨은 `docker node inspect`로 확인할 수 있다.

#### 서비스 제약 설정

##### node.labels 제약조건

`docker service create` 명령어에 `--constraint` 옵션을 추가해 서비스의 컨테이너가 할당될 노드의 종류를 선택할 수 있다. 

```shell
docker service create --name label_test \
--constraint 'node.labels.storage == ssd' \
--replicas 5 \
ubuntu:14.04 \
ping docker.com
```

`docker service ps`를 통해 확인하면 ssd로 설정된 swarm-worker1 노드에만 컨테이너가 생성된 것을 확인할 수 있다. 여러 노드에 할당되어 있다면 매니저 노드의 스케줄러에 의해 할당하게 된다.

##### node.id 제약조건

노드의 ID를 명시해 서비스의 컨테이너를 할당할 노드를 선택할 수 있다. 다른 도커 명령어와 달리 ID 전부 입력해야한다.

```shell
docker service create --name label_test2 \
--constraint 'node.id == iojoijfwoijfklnmxvcoinowenornw'
--replicas=5 \
ubuntu:14.04 \
ping docker.com
```

##### node.hostname과 node.role 제약조건

스웜 클러스터에 등록된 호스트 이름 및 역할로 제한 조건을 설정할 수 있다. 

```shell
docker service create --name label_test3 \
--constraint 'node.hostname == swarm-worker1'
ubuntu:14.04 \
ping docker.com
```

```shell
docker service create --name label_test4 \
--constraint 'node.role != manager'
--replicas=2 \
ubuntu:14.04 \
ping docker.com
```

##### engine.labels 제약조건

도커 엔진, 도커 데몬 자체에 라벨을 설정해 제한 조건을 설정할 수 있지만 이를 사용하려면 도커 데몬의 실행 옵션을 변경해야 한다. 서비스를 생성할 때 `engine.labels` 접두어로 제한조건을 설정하면 도커 데몬의 라벨을 사용할 수 있다.

```shell
docker service create --name engine_label \
--constraint 'engine.labels.mylabel == worker2'
--replicas=3 \
ubuntu:14.04 \
ping docker.com
```



위 예시들은 제약 조건을 하나로만 했지만 여러개 붙여서 사용할 수도 있다.