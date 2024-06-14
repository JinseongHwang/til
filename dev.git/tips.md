## git checkout은 역사속으로

git checkout은 너무 많은 기능을 가지고 있어, 분리되기 시작했다. (꽤 됐다)  
따라서 이제는 `git --help`를 쳐도 checkout에 대한 설명은 나오지 않는다.

### 작업 브랜치 변경
```sh
# Before
$ git checkout my-branch

# After
$ git switch my-branch
```

### 새로운 브랜치 만들면서 작업 브랜치 변경
```sh
# Before
$ git checkout -b my-branch

# After
$ git switch -c my-branch
```

### 최근 commit된 상태로 파일 되돌리기
```sh
# Before
$ git checkout -- main.py

# After
$ git restore main.py
```

## 조금 더 간편한 리모트 브랜치 삭제

```sh
# Before
$ git push origin --delete my-branch

# After
$ git push origin :my-branch
```

## 이전 브랜치로 되돌아가기
```sh
git switch -
git checkout -
```

-  둘 다 가능하다.