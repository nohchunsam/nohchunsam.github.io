---
title: 깃허브 계정 두 개 사용하기 (Window)
date: 2025-07-08 11:10:00 +0900
categories: 기타
tags: ["SSH"]
---

모종의 이유로 깃허브 계정을 두 개 만들었다. 이제 내 컴퓨터에서 깃 계정을... A 프로젝트면 A 계정, B 프로젝트면 B 계정을 사용하듯이 프로젝트에 따라 맞는 계정을 사용하기 위해 방법을 이리저리 찾아봤다. 

### 1. SSH 키 만들기
/.ssh 폴더에 키를 만들어준다. 보통 사용자 폴더 안에 있으니 찾아보자.

```
ssh-keygen -t rsa -C "userA@gmail.com" -f "id_userA"
ssh-keygen -t rsa -C "userB@gmail.com" -f "id_userB"
```
위의 명령어를 통해 키를 생성할 수 있다. 
나는 -t에서 rsa 유형의 키를 생성했고 파일명은 "id_userA"로 설정했다. 파일명은 본인이 직관적이다 생각하는 걸로 자유롭게 작성하자.

해당 코드를 치면 `Enter passphrase (empty for no passphrase):` 가 뜨는데, 암호 설정이다. 설정하고 싶으면 타이핑하고, 없으면 엔터를 치자.

```
PS C:\Users\nohchunsam\.ssh> ssh-keygen -t rsa -C "userA@gmail.com" -f "id_userA"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_userA
Your public key has been saved in id_userA.pub
The key fingerprint is:
SHA256:  
userA@gmail.com
The key's randomart image is:
+---[RSA 3072]----+
|ooEooo           |
|o+o.oooo .       |
|o+..o+*++ .      |
|=.o.o=B*o+ .     |
|.* ==oo+S +      |
|. o += .   .     |
|   o  o          |
|  .              |
|                 |
+----[SHA256]-----+
```
이런 글이 뜨면 잘 생성된 거다. 다른 계정에 대해서도 키를 생성해주자.

### 2. 설정 파일 추가하기
이제 SSH를 설정해야한다. .ssh 폴더 안에 config 파일을 열어주자. 없으면 확장자 없이 하나 만들자.

```
# 계정 A
Host github-userA
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_userA

# 계정 B
Host github-userB
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_userB
```

### 3. 에이전트에 등록하기
윈도우는 설정하기 위해 **PowerShell을 관리자 모드**로 켜야한다.
```
Get-Service ssh-agent | Set-Service -StartupType Manual
Start-Service ssh-agent
```
해당 커맨드를 통해 에이전트를 킬 수 있다.
아래는 켜졌는지 확인하는 커맨드
```
PS C:\Users\nohchunsam\.ssh> Get-Service ssh-agent
>>

Status   Name               DisplayName
------   ----               -----------
Running  ssh-agent          OpenSSH Authentication Agent
```
이제 ssh 폴더로 가서 키를 추가해주면 된다.
```
PS C:\Users\nohchunsam\.ssh> ssh-add id_userA
Identity added: id_userA (userA@gmail.com)
PS C:\Users\nohchunsam\.ssh> ssh-add id_userB
Identity added: id_userB(userB@gmail.com)
```

### 4. 깃허브에 키 등록하기
깃허브 페이지 > Setting > SSH and GPG keys에 가면 다음과 같은 페이지를 볼 수 있다. 

(화면은 내가 이미 설정해서 키가 있는 것! 처음 들어가는 사람들은 설정한 키가 없다고 뜰 것이다.)

![](img/github_ssh_page.jpg)

New SSH key 버튼을 눌러 키를 저장하자.

커맨드 창에 ```cat id_userA.pub``` 를 치면 ssh 부터 시작하는 암호문이 뜰 것이다. ssh까지 끝까지 복사해 key창에 집어넣고 버튼을 누르자.

![](img/add_ssh_page.png)

### 5. 레포지토리에 설정하기
이제 아래 커맨드를 통해 잘 설정됐는지 확인할 수 있다!
```
ssh -T git@github-userA
ssh -T git@github-userB
```

아래 글이 뜨면 성공!
```
Hi userA! You've successfully authenticated, but GitHub does not provide shell access.
```

이제 잘 연결이 됐다는 거니 원하는 레포지토리에 설정만 하면 된다.

원하는 레포지토리로 가서 아래 커맨드를 쳐주자
```
git remote set-url origin git@github-userA:userA/repositoryA.git
```

이렇게 설정한 후, 아무 커밋이나 하고 push하면 제대로 작동하는 걸 확인할 수 있다.
