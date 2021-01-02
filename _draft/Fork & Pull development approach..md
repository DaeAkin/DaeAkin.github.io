개발을 하다가 좋은 오픈소스가 있길래, 나도 컨트리뷰터가 되기로 했다.

그런데 이 오픈소스는 다음과 같은 전략을 쓴다고 한다.

![]({{site.url}}/images/fork/image2.png)

**Fork & Pull development approach** 이게 뭔지 지금 부터 알아보자.

## Fork & Pull development approach

fork and pull 모델은 누구나 오픈소스 저장소를 fork를 한 다음 수정사항을 내 저장소에 반영하는 것이다. 이렇게 하면 푸쉬 할 때 마다 오픈소스 저장소에 승인을 요청하지 않아도 된다. 내 저장소에서 오픈소스 저장소로 나의 수정사항을 pull request를 보낼 수 있다. 이런 방법은 새로운 컨트리뷰터와의 마찰을 줄이고, 독립적으로 개발을 진행할 수 있기 때문에, 오픈소스에서 널리 쓰이는 방법이다.

만약 shared 저장소 모델(저장소에 직접적인 접근이 가능한 경우)인 경우 collaborator 들은 저장소에 대한 push 권한을 얻어야 하며, 수정사항 마다 새로운 브랜치를 만들어 작업한다. 

## Fork 하기

가장 먼저 해야할 일은 내가 협력하고 싶은 저장소를 Fork 하는 것이다. Fork는 하는 방법은 간단하다. Fork 하려는 저장소에 가서, Fork 버튼만 누르면 자동으로 된다.

![]({{site.url}}/images/fork/image1.png)

fork를 뜬 다음 fork 뜬 저장소를 내 로컬에 다운을 받자.

```
# fork한 저장소를 로컬로 클론하자.
git clone git@github.com:USERNAME/FORKED-PROJECT.git
```

## 포크된 저장소를 최신으로 유지하기

굳이 꼭 최신으로 유지해야 되는 건 아니다. 만약 간단한 걸 수정하는 것 이상을 작업해야 한다면, 내가 fork 한 원본 저장소를 추적해서 지속적으로 최신화를 하고 싶을 때가 있다. 최신화를 하고 싶으면 다음과 같이 remote를 추가하면 된다.

```
# 원본 remote를 "업스트림" 이란 이름으로 추가
git remote add upstream https://github.com/UPSTREAM-USER/ORIGINAL-PROJECT.git

# 추가된 remote 업스트림 확인
git remote -v
```

업스트림에서 최신 버전을 내 저장소에 반영하고 싶다면, 업스트림의 저장소 브랜치에서 fetch를 하게 되면, 최신 커밋이 내 저장소에 들어오게 된다.

```
# 업스트림 remote에서 가져오기
git fetch upstream

# 모든 브랜치 확인하기
git branch -va
```

이제 내 저장소의 master 브랜치에서, 업스트림에 있는 master 브랜치로 병합을 해보자.

```
# master 브랜치로 Checkout 하고 upstream에서 merge 하기
git checkout master
git merge upstream/master
```

내가 마스터 브랜치에 어떤 커밋도 없다면, git은 fast-forward로 병합을 한다. 그러나 마스터 브랜치에 작업이 존재하면, 충돌이 나게 된다. 그러므로 업스트림에서 받아올 때 조심하자.

이제 내 브랜치는 최신으로 업데이트 되었다.

## Doing Your Work

### 브랜치 만들기

새로운 기능을 추가하거나, 버그를 수정을 하려면 새로운 브랜치를 만들어야 한다. 이런 방법은 깃 workflow의 적절한 방법일 뿐 아니라, 수정사항을 일관되게 관리할수 있으며, 마스터 브랜치로부터 분리를 할 수 있다. 그래서 매번 작업을 완료할 때마다 여러 개의 풀 리퀘스트를 보낼 수 있다.

To create a new branch and start working on it:

```
# Checkout the master branch - you want your new branch to come from master
git checkout master

# Create a new branch named newfeature (give your branch its own simple informative name)
git branch newfeature

# Switch to your new branch
git checkout newfeature
```

Now, go to town hacking away and making whatever changes you want to.

## Pull Request 제출하기

### Cleaning Up Your Work



Prior to submitting your pull request, you might want to do a few things to clean up your branch and make it as simple as possible for the original repo's maintainer to test, accept, and merge your work.

If any commits have been made to the upstream master branch, you should rebase your development branch so that merging it will be a simple fast-forward that won't require any conflict resolution work.

```
# Fetch upstream master and merge with your repo's master branch
git fetch upstream
git checkout master
git merge upstream/master

# If there were any new commits, rebase your development branch
git checkout newfeature
git rebase master
```

Now, it may be desirable to squash some of your smaller commits down into a small number of larger more cohesive commits. You can do this with an interactive rebase:

```
# Rebase all commits on your development branch
git checkout 
git rebase -i master
```

This will open up a text editor where you can specify which commits to squash.

### Submitting

Once you've committed and pushed all of your changes to GitHub, go to the page for your fork on GitHub, select your development branch, and click the pull request button. If you need to make any adjustments to your pull request, just push the updates to GitHub. Your pull request will automatically track the changes on your development branch and update.

## Accepting and Merging a Pull Request

Take note that unlike the previous sections which were written from the perspective of someone that created a fork and generated a pull request, this section is written from the perspective of the original repository owner who is handling an incoming pull request. Thus, where the "forker" was referring to the original repository as `upstream`, we're now looking at it as the owner of that original repository and the standard `origin` remote.

### Checking Out and Testing Pull Requests

Open up the `.git/config` file and add a new line under `[remote "origin"]`:

```
fetch = +refs/pull/*/head:refs/pull/origin/*
```

Now you can fetch and checkout any pull request so that you can test them:

```
# Fetch all pull request branches
git fetch origin

# Checkout out a given pull request branch based on its number
git checkout -b 999 pull/origin/999
```

Keep in mind that these branches will be read only and you won't be able to push any changes.

### Automatically Merging a Pull Request

In cases where the merge would be a simple fast-forward, you can automatically do the merge by just clicking the button on the pull request page on GitHub.

### Manually Merging a Pull Request

To do the merge manually, you'll need to checkout the target branch in the source repo, pull directly from the fork, and then merge and push.

```
# Checkout the branch you're merging to in the target repo
git checkout master

# Pull the development branch from the fork repo where the pull request development was done.
git pull https://github.com/forkuser/forkedrepo.git newfeature

# Merge the development branch
git merge newfeature

# Push master with the new feature merged into it
git push origin master
```

Now that you're done with the development branch, you're free to delete it.

```
git branch -d newfeature
```



