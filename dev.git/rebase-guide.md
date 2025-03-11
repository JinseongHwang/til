### Git Rebase를 사용하여 부모 브랜치의 최신 커밋 반영하기

**상황 설명:**

- 현재 작업 중인 브랜치(`child`)는 부모 브랜치(`parent`)에서 만들어졌다.
- 부모 브랜치(`parent`)에서 브랜치를 만든 이후 추가 커밋이 생겼다.
- 최신 상태의 부모 브랜치 커밋을 내 브랜치(`child`)에 깔끔하게 반영하고 싶다.

---

### 🔧 사용 명령어 (Step-by-Step)

1\. **부모 브랜치의 최신 커밋을 가져온다:**
```bash
git fetch origin parent
```

2\. **부모 브랜치를 기준으로 내 브랜치를 rebase한다:**
```bash
git checkout child
git rebase origin/parent
```

- 리베이스 중 충돌(conflict)이 발생하면 충돌을 해결한 후 다음 명령어를 입력한다.

```bash
git add <해결한 파일>
git rebase --continue
```

- 리베이스를 중단하고 이전 상태로 복구하려면:

```bash
git rebase --abort
```

3\. **리베이스 결과를 원격 브랜치(GitHub)에 반영한다:**

```bash
git push origin child --force-with-lease
```

- `--force-with-lease`는 다른 사람의 변경사항을 덮어쓰지 않도록 도와주는 안전한 방법이다.

---

### 📌 명령어 요약

```bash
git fetch origin parent
git checkout child
git rebase origin/parent
# 충돌 해결 후: git rebase --continue
git push origin child --force-with-lease
```

---

**이 방식을 사용하면 깔끔한 커밋 이력을 유지할 수 있다.**

