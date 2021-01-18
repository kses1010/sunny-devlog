---
title: 'MacOS에서 python, pyenv 설치하기'
date: 2021-01-18
category: 'python'
draft: false
---

코딩테스트 대비를 위해 알고리즘 언어를 파이썬을 선택했습니다. 파이썬은 C++이나 Java에 비해는 느리지만,

빠르게 알고리즘 로직을 짤 수 있고, 특히 문자열에는 매우 강력합니다.

그래서 이번 포스트는 Python 설치법을 알아보겠습니다.

## 01. macOS 내장 파이썬 확인하기

```bash
python -V
Python 2.7.15
which python ## 바이너리 파일 위치
/usr/bin/python
```

## 02. 파이썬 설치 및 환경설정하기

```bash
brew install python

# zshrc일 경우 bash면 끝에 ~/.bash_profile
echo "alias python=/usr/local/bin/python3" >> ~/.zshrc
echo "alias pip=/usr/local/bin/pip3" >> ~/.zshrc

# vi ~/.zshrc
# 위 명령어가 제대로 입력된다면 다음 스크립트가 있어야한다.
alias python=/usr/local/bin/python3
alias pip=/usr/local/bin/pip3

# shell 재기동
exec "$SHELL"

# 확인
python -V
Python 3.8.5
```

## 03. 파이썬 버전 관리

파이썬 알고리즘 인터뷰, 이것이 취업을 위한 코딩테스트다 라는 책을 읽으면서 알고리즘테스트 환경이 파이썬 버전 3.6이하 일 경우를 대비하여 버전관리가 필요하다고 생각했습니다. 그래서 pyenv를 설치했습니다.

```bash
brew install pyenv

# zshrc일 경우 bash면 끝에 ~/.bash_profile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc

# vi ~/.zshrc
alias python # alias 파이썬은 전부 삭제.

# 위 명령어가 제대로 입력된다면 다음 스크립트가 있어야한다.
if command -v pyenv 1>/dev/null 2>&1; then
	eval "$(pyenv init -)"
fi

# shell 재기동
exec "$SHELL"

# 파이썬 설치
pyenv install 3.7.8
pyenv install 3.8.5

# 파이썬 버전 선택하기
pyenv global 3.7.8

# 파이썬 버전 확인하기
pyenv versions
	system
* 3.7.8 (set by /Users/dale/.pyenv/version)
  3.8.5
```

파이썬 깔끔하게 설치하고 즐겁게 코딩하세요:yum:
