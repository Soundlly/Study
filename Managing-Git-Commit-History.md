# Managing Git Commit History

Git을 이용한 버전관리는 편리하고 명확한 커뮤니케이션을 가능하게 만들어준다.
작은 프로젝트는 대충 사용해도 별 문제 없지만, 프로젝트의 참여자가 많아지거나 지속 기간이 길어지면 여러 문제가 발생한다.

- 시간이 지남에 따라 프로젝트 전체의 문맥을 잊어버리게 됨
- 코드 충돌을 해소하면서 의도치 않게 남의 코드를 변경하게 됨
- 이슈가 발생할 경우, 언제부터 있던 문제인지 추적하기 어려움
- 왜 코드를 이렇게 짰는지 그때의 의도가 명확히 기억나지 않음

보통 이런 현상은 서서히 진행되어 복합적인 형태로 발생하는데, 예를 들면 다음과 같다:

> 이슈가 발생했는데, 내가 예전에 구현한게 이상하게 좀 다른 형태고 있는 것 같고, 언제부터 그랬는지도 잘 모르겠다. 아참, 근데 그때는 왜 이렇게 짰더라...??

이슈가 발생했을 때, 내가 모든 내용을 알고 있다면 가장 좋다(~~그랬으면 이 문제가 생겼겠냐~~).
하지만 그런 일은 일어나지 않으니, 우리가 얻을 수 있는 최상의 결론에 가까운 것은 **"문제가 어느 부분에서 발생했는지만 알 수 있다면, 언제 누가 왜 그렇게 짰는지 알 수 있다"** 일 것이다. 그리고 이런 일이 발생하지 않도록 할 수 있으면 더 좋을 것이다.

### Git Blame

Git Blame은 특정 파일의 Commit 내역을 볼 수 있게 해 준다. Git을 리누스 토발즈가 만들었다는 것에서 수많은 컨트리뷰터가 있는 상황에서 코드 작성자의 책임을 명시 ~~해서 조지려고~~ 하려고 만든 것 같다. 뭐, 아무튼 기능 자체는 아름답다.

기본적으로는 [Git](https://git-scm.com/docs/git-blame)의 자체 기능이지만, [GitHub](https://help.github.com/articles/tracing-changes-in-a-file/)에서 사용하게 되는 경우가 더 많다. 적어도 나는 그런데, 무엇보다 정보가 다양해서 터미널 UI로는 깔끔하게 보기 어렵기 때문이 아닐까 싶다. GitHub의 특정 파일 뷰에서 우측 상단의 `Blame` 버튼을 누르면 해당 모드로 진입하며, 현재 코드 기준으로 각 라인에 영향을 준 Commit을 모두 볼 수 있다. 각각의 조각에는 Commit Title과 시간이 나타나 있고, 시간은 가운데 세로선의 색상으로도 확인할 수 있다(진할수록 최근). 추가로, Commit Message와 코드 블록 사이의 아이콘을 누르면, 파일의 버전을 해당 커밋 이전으로 건너뛰어 과거의 변경을 볼 수도 있다. 리팩토링 등으로 Blame View가 추적하기 어려운 경우 사용하면 좋다.

여기서도 알 수 있듯이 **Commit Title은 정말 중요**하다. 위에서도 50글자 넘어가면 ...으로 짤리기 시작한다.
매번 권장하는 글이지만, Commit Message는 [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/) 글을 읽을 때 마다 점점 더 좋아지는 것 같다. 처음 읽었을 때는 *왜 굳이...?* 싶기도 한데, 사용하다 보면 그 이유들을 체감하게 된다. 위 글에도 언급되는 Git Commit의 50/72 Rule의 뒷이야기가 궁금하다면 [이 글](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)을 읽어보면 좋다(재밌음).

## GitHub Pull Request

GitHub의 Pull Request는 기본적으로 `compare` 브랜치를 `base` 브랜치에 반영하고 싶을 때 사용한다.
이 때, GitHub은 세 종류의 반영 방법을 제공한다. 각각의 장단점이 있으니 잘 사용해보면 될 것 같다.

### Create a Merge Commit

GitHub에서 프로젝트를 생성하면 가장 일반적으로 설정되어 있는 반영 방법이다.
이 버튼을 누르면 **Merge Commit Title & Body** 를 작성할 수 있는 입력창이 나타나며, 기본 메시지는 다음과 같다

> **Merge Commit Title**
> Merge pull request #**Number** from **Username**/**compare**
>
> **Merge Commit Body**
> Title of Pull Request

가장 쉽지만, 기본 기능을 그대로 사용하면 몇 가지 아쉬운 점이 발생한다.

- Branch 이름과 Pull Request 이름에 영향을 많이 받는다.
- Git Commit Message의 50/72 Rule을 타이틀 길이 제한이 넘어가기 쉽다.
- Git Blame 으로 추적하는 경우 파악이 어렵고, Branch 이름을 제대로 지었더라도 길면 짤려 보인다.
- Pull Request에 추가적으로 기술한 정보를 Commit Level에서 알기 어렵다.
  - GitKraken, SourceTree, Fork 등의 데스크톱 프로그램만 사용해서는 내용을 알기가 어렵다.

이를 개선하기 위해 다음 방식으로 개선해 보았다.

1. **Branch Name**

   보통 Git-Flow 를 사용하면 브랜치 이름은 곧 Feature 이름이 되는데, 이 때, 구현할 단위 기능을 최대한 잘 반영하도록 명명한다. SDK는 `feature/yyddmm-this-is-feature-headline` 같은 네이밍으로 사용중.

   - 간단한 한 줄 짜리 문장으로 만들 수 없다면, 단위 기능을 충분히 쪼개지 못했을 가능성을 생각해 보자.

2. **Pull Request Title**

   왠만해선 PR 제목은 Branch Name을 그대로 사용하되, 전치사와 조동사를 제외한 단어를 Capitalize 하는 것이 좋다.
   위의 예를 그대로 옮기면 **This is Feature Headline** 가 된다.

3. **Pull Request Body**

   PR 내용은 Commit Title / Message 들을 종합한 요약이 되면 좋다. **정 안되면 Commit Title 만이라도 모아서 리스트로 만들자**. 중요하고 큰 작업일 수록 위쪽에 적고, 가독성을 고려하여 작성한다.

4. **Merge Commit Title**

   다음과 같이 Pull Request의 번호와 제목을 이용하여 작성하고, 곧 Feature Branch의 이름이 되면 가장 좋다.

   >  (#**Number**) Pull Request Title

   위의 예를 그대로 옮기면, **(#486) This is Feature Headline** 이 된다.

5. **Merge Commit Body**

   Pull Request의 Body를 그대로 사용하는 것이 수고를 덜고 GitHub 없이 Commit History 만으로도 기능 단위가 구현된 요약을 볼 수 있어서 좋다. 다만, Heading의 경우에는 `#`들을 지우고, 윗 줄과 공백을 한 줄 확보해두도록 한다.

   > **Example**
   >
   > Pull Request Body
   >
   > ```
   > ## Note : Git will no longer track files under `build/`
   >
   > ### Changes
   > - Remove files in `build/` and ignore in `.gitignore`
   > - ...
   > ```
   > Merge Commit Body
   > ```
   > Note : Git will no longer track files under `build/`
   >
   > Changes
   > - Remove files in `build/` and ignore in `.gitignore`
   > - ...
   > ```


### Squash and Merge

Squash and Merge는 위와 같이 Commit Message를 작성하기는 하지만, PR에 올라온 모든 Commit의 변경점을 가지고 있는 하나의 Commit을 `base` 로부터 생성한다. 따라서, PR 반영으로 만들어지는 Commit은 기존 `base` 브랜치의 헤드 하나만을 부모로 가진다. 기본 메시지도 Pull Request에 올라온 Commit List 내용이 정해진 포맷으로 작성되어 있는 등, 차이가 있다. Conflict가 발생하지 않는다면, 이는 다음 커맨드를 GitHub이 대신 실행해주는 것과 같다.

```
git fetch origin
git checkout <compare>
git reset --soft HEAD~`git rev-list --count <base>...HEAD`
git checkout <base>
git commit
```

이를 사용하는 경우는, 작은 작업인데 버그픽스 등으로 나중에 읽을 필요가 없는 커밋을이 양산된 경우, 이를 깔끔하게 정리해서 `base` 브랜치에 반영할 수 있다는 장점이 있다. 단점으로는

- PR의 총량이 많은 경우, Commit 하나로 만듦으로서 나중에 볼 때 작업의 맥락을 파악하기 어려워질 가능성 존재
- 여러명이 하나의 작업을 진행한 경우, Squash로 Commit을 만든 사람의 Author만 남기 때문에 작업자 판단이 어려움

등이 있는데, Squash Merge Commit에 PR 번호를 남겨 필요한 경우 추가로 추적할 수 있도록 보완하는 방법이 있다.

### Rebase and Merge

Rebase and Merge는 위의 두 반영 방식과는 다르게 Commit Message를 적는 입력창이 나타나지 않는다.
그도 당연한게, `compare` 브랜치는 `base` 브랜치에 rebase하면 곧 `base` 브랜치를 `compare` 브랜치로 Fast-Forward할 수 있게 되기 때문이다. 이는 다음 커맨드를 GitHub이 대신 실행해주는 것과 같다.

```bash
git fetch origin
git checkout <compare>
git rebase <base>
git push -f origin <compare>
```

이를 이용하면 아주 깔끔한 Commit History를 만들 수 있으나, 약 2달 이상 사용한 경험에서 다음의 단점을 크게 느껴 이후로는 사용하지 않게 되었다. 이는 GitHub의 Rebase and Merge 기능을 사용할 때 느껴지는 단점이지, `git rebase`의 단점이 아니다. `git rebase`는 여전히 잘 쓰고 있고, 또 유용하다.

- Feature 단위의 작업을 Commit History에서 분리해서 볼 수 없음
- Commit History에 Pull Request 번호 흔적을 남길 수 없음

Commit History에서 작업 단위로 Commit을 구분할 수 없다는 것은 이슈 추적과 흐름 파악에 있어서 치명적이라고 생각한다.

## Modifying Git Commit History

Git Commit History 관리를 더 잘할 수 있게 해 주는 `git cherry-pick` `git-rebase` 를 소개한다.

### Cherry-Pick

체리나무에서 체리를 하다 따는 것 처럼, Commit History에서 특정 Commit을 가져와서 현재 브랜치(의 **HEAD**)에 붙일 수 있다. 명령어는 다음과 같이 사용한다. `git checkout` 처럼  `commit-hash` 항목에는 첫 7글자만 사용해도 동작한다.

```bash
git cherry-pick <commit-hash>
```

물론 호환되지 않는 경우에는 Conflict가 발생하며, 기본적으로 기존 Commit의 Author가 유지된다. GitHub에서는 다른 사람이 Cherry-Pick한 경우 메인 아이콘에는 기존의 Author가, 서브 아이콘에는 Cherry-Pick을 시행한 Author가 나타나는데, 주로 고정적인 작업의 Commit을 재사용할 때 유용하다.

### Rebase

Rebase는 **Re** + **base**, 즉 `base` 대상을 재설정한다는 뜻이다.
이 기능을 이용하면 특정 브랜치를 마치 **대상 브랜치에서 작업을 시작한 것 같이** 만들어 준다.

주로 특정 Feature 브랜치가 오래되어 PR 충돌이 나는 경우, PR의 `base` 브랜치로 리베이스하여 다시 올리는 경우가 발생한다. 이 때, PR의 `base` 브랜치를 병합하여 다시 올리는 것 보다 Rebase를 사용하면 더 명확한 PR Commit 기록을 만들 수 있다.

또 다른 용도로는 자신의 과거 브랜치를 대상으로 실행함으로써 위에서 말한 **Squash**같은 동작을 할 수도 있다 ([참고](https://www.devroom.io/2011/07/05/git-squash-your-latests-commits-into-one/)).

```
git rebase -i HEAD~3
```

`-i` 옵션은 interactive 옵션으로, 텍스트 입력기가 뜨게 되는데, 여기서 `pick` 대신 `squash` 를 입력하여 저장하면 해당 커밋들은 흡수되어 처리된다. Squash는 실제로 존재하는 Git Command는 아니고, 추상적인 개념이라고 생각하면 된다.
