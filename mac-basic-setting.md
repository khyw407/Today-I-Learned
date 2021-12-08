# Mac 초기환경 세팅 

## 키보드 기호

|제목|내용|
|------|---|
|⌘|Command|
|⌥|Option|
|⌃|Control|
|⇧|Shift|
|+|동시 입력|
|,|키를 떼고 다시 입력함|

## 키보드 설정

![image](https://user-images.githubusercontent.com/37721713/145139246-f2feca6d-5061-46bc-8456-2080dae44fdf.png)

![image](https://user-images.githubusercontent.com/37721713/145139329-f3ef8ab9-9e99-45d9-be8e-4909b711b93e.png)

## 트팩패드

![image](https://user-images.githubusercontent.com/37721713/145139416-9313655b-4bd3-420b-8380-87986b2b4a2c.png)

## Homebrew

각종 커맨드라인 프로그램과 일반 프로그램(크롬..)을 손쉽게 설치해주는 맥용 패키지 매니저입니다.(최근에 리눅스도 지원하기 시작했습니다.) 리눅스의 apt나 yum과 비슷하며 brew외에 MacPorts 라는 패키지 메니저가 있는데 몇몇 단점으로 요즘은 거의 brew를 사용하는 추세입니다. 다양한 프로그램을 복잡한 빌드과정 없이 손쉽게 설치할 수 있고 업데이트, 관리도 간단하므로 안쓸 이유가 없는 필수 프로그램입니다.

```
# Install
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# brew test
$ brew doctor
Your system is ready to brew.
```

## xcode

macOS는 기본적으로 gcc, make와 같은 컴파일 도구가 설치되어 있지 않기 때문에 명령어 도구Command Line Tools를 설치해야 합니다. 예전에는 Xcode를 전체 설치하고 추가로 명령어 도구를 설치해야 했으나 Xcode용량이 꽤 크고 모든 사람이 IDE가 필요한 게 아니기 때문에 명령어 도구만 따로 설치할 수 있게 변경되었습니다

- homebrew를 설치하면 자동으로 Xcode 명령어 도구를 설치합니다. 따로 설치하지 않아도 됩니다.

```
$ xcode-select --install

# gcc test
$ gcc
clang: error: no input files
```

## git

```
$ brew install git
$ git config --global user.name "Your Name"
$ git config --global user.email "you@your-domain.com"
```

## iTerm2

macOS에 기본으로 설치되어 있는 Terminal 앱 대신 iTerm2를 터미널 앱으로 사용합니다. iTerm2는 기본 앱에 없는 다양한 기능이 있고 손쉽게 테마를 설정할 수 있습니다.

```
$ brew cask install iterm2

```
