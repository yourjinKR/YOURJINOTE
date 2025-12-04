# merge 종류
## merge
일반 머지임
## rebase merge
Rebase merge는 Merge할 Branch의 Commit 내역들을 그대로 옮긴다. 
Branch의 Base를 옮기는 일이니 말 그대로 Rebase다. 그래서 Branch는 이어지지 않는다.
## squash merge ⭐
Squash merge는 Rebase하되 Merge할 Commit들을 **하나의 Commit으로 뭉친다**.
Squash는 사전적 의미로 으깨다, 짓누르다 라는 뜻을 가지고 있다.

사실 Squash는 Rebase merge의 Option이다. Squash 옵션을 적용한 Rebase merge를 Squash merge라고 부르는 모양이다.

# 깃허브 merge 허용
![Pasted image 20251126174618](../../GALLERY/Pasted%20image%2020251126174618.png)
레포지토리 -> setting -> general -> 하단에 PR에서 설정

# 출처
[출처](https://velog.io/@code-bebop/Github-merge-squash-merge-rebase-merge)