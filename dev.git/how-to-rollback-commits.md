## 이미 push 해버린거 되돌리기

push되면 안될게 push 되버렸다. 커밋 히스토리가 망가지기 직전이다. 절대 당황하지 말자.  
크게 reset과 revert가 있다. 각각의 특징을 살펴보자.

### git reset
- 실수로 한 커밋을 삭제할 수 있다. 즉, 실수로 한 커밋의 이전 상태로 되돌리는 것이다.
- 다만, "삭제"하는 과정이기 때문에 force push가 불가피하다.

### git revert
- 실수로 한 커밋의 이전 상태로 되돌리는 커밋을 하나 더 추가하는 것이다.
- force push를 하지 않아도 되기 때문에 위험 부담이 덜하다.

### 내 생각
오래 전 학부생 시절, force push를 해서 코드를 다 날려먹은 적이 있다. 깃허브에 잔디도 열심히 심었는데 다 사라지더라. (코드 백업을 깃허브에만 해뒀기 때문에) 약간 트라우마로 남아서 다시는 force push를 하지 않는다. 만약 회사의 main 브랜치라면 더더욱 할 수 없다.

## revert의 단점

revert는 커밋 하나씩만 되돌릴 수 있다. 2개의 커밋을 되돌리려면 2개의 revert commit 기록이 남게 된다.
당황하지 말자. `--no-commit` 옵션을 사용하면 얼마든 가능하다.

아래와 같이 커밋 로그가 찍혀있다고 가정해보자.
```sh
$ git log --oneline -n 3

aaa111 (HEAD) 잘 먹었다고 소문이 나지?
bbb222 뭘 먹어야
ccc333 오늘 저녁은
```

`ccc333` 커밋이 완료된 시점으로 되돌아 가고 싶다면? 그럼 2개의 커밋을 되돌리면 된다.

```sh
$ git revert --no-commit aaa111
$ git revert --no-commit bbb222
$ git commit -m "Revert 2 recent commits"

$ git log --oneline -n 2

abc123 (HEAD) Revert 2 recent commits
ccc333 오늘 저녁은
```

위와 같이 요래요래 하면 2개의 커밋을 되돌리면서 커밋 히스토리에는 1개만 남겨진다.

---

그럼에도 불구하고 github에 올라간 커밋을 정리하고 싶다면,

1. 커밋 내역을 살펴보자
```Shell
git reflog
```
살펴보다가 이동하고 싶은 커밋 시점의 커밋 해시를 복사해두자.

2. hard reset을 하자
```Shell
git reset --hard {commit-hash}
```
로컬에서 우선 커밋을 되돌려보자. 잘 되돌아갔는지 확인하자.

3. 강제로 github에 동기화하자.
```Shell
git push origin {branch-name} --force
```
로컬에서 문제 없다면 force push를 통해 github에 강제로 동기화해주자.