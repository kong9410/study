# Revert

push 이후 이미 작업된 commit들 중 특정 commit을 원래대로 되돌리고 싶은 경우에 revert를 사용하여 롤백을 할 수 있다.

reset과는 다르게 기존의 commit은 그대로 유지하기 때문에 충돌의 위험이 없다.



## 사용법

`git revert [commit id]`

`git revert 3xdg6f`

