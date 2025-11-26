다른 브랜치의 모든 작업을 가져와야 할 때

# squash
```bash
git checkout develop
git pull origin develop   # 최신 develop 가져오기
```

```bash
git checkout develop
git checkout -b new-branch
```

```bash
git merge --squash origin/old-branch
```
