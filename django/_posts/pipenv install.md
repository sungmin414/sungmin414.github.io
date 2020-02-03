# pipenv install

### Installation

### Ubuntu Linux

```
pip install --user --upgrade pip
pip install --user --upgrade pipenv
```

`vi ~/.zshrc`

```
export PATH=$HOME/bin:/usr/local/bin:$PATH
export PATH=$HOME/.local/bin:$PATH
```

### macOS
```
brew install pipenv
```

### Using

- 초기화 (Python버전 지정 및 virtualenv생성)

```
pipenv --python 3.6.8
```

- 패키지 설치

```
pipenv install <패키지명>
```

- virtualenv 위치

```
~/.local/share/virtualenvs/<가상환경명>
```

- 해당 가상환경의 Shell적용

```
pipenv shell
```

- 가상환경 지우기

```
pipenv --rm
```

## pipenv settings

+ pipenv는 알아서 가상환경까지 설정함
+ 기본 셋팅 해보기

```
> settings 순서

pipenv --python 3.6.8 -> pipenv shell (해당 가상환경적용) 

-> pipenv install django djangorestframework pygments

-> django-admin startproject config(mv config app)

-> interpreter설정(~/.local/share/virtualenvs/<가상환경명>)


```