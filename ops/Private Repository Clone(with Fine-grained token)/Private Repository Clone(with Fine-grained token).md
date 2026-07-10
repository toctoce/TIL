# Private Repository Clone(with Fine-grained token)

> 비공개 저장소를 `clone`하기 위해 Fine-grained personal access token을 생성하고 사용하는 방법

`EC2` 인스턴스에서 비공개 저장소를 `clone`하려고 한다.

비밀번호 입력란에 GitHub 계정 비밀번호를 입력하면 인증에 실패한다.

```text
$ git clone https://github.com/toctoce/<repository>
Cloning into '<repository>'...
Username for 'https://github.com': toctoce
Password for 'https://toctoce@github.com': password123
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/toctoce/<repository>/'
```

GitHub는 Git 작업에 대한 비밀번호 인증을 지원하지 않으므로 HTTPS 방식에서는 토큰으로 인증해야 한다.

## 토큰 생성

GitHub에서 아래 경로로 이동한다.

```text
Settings - Developer settings - Personal access tokens - Fine-grained tokens - Generate new token
```

토큰을 생성할 때 접근할 저장소를 선택한다.

## 권한 설정

`Permissions`에서 `Contents`를 선택한다.

- `clone`만 필요하면 `Read-only`를 선택한다.
- `push`도 필요하면 `Read and write`를 선택한다.

비밀번호 입력란에 GitHub 계정 비밀번호 대신 생성한 토큰을 입력하면 `clone`할 수 있다.

```text
$ git clone https://github.com/toctoce/<repository>
Cloning into '<repository>'...
Username for 'https://github.com': toctoce
Password for 'https://toctoce@github.com': 
remote: Enumerating objects: 1107, done.
remote: Counting objects: 100% (1107/1107), done.
remote: Compressing objects: 100% (433/433), done.
remote: Total 1107 (delta 535), reused 1060 (delta 488), pack-reused 0 (from 0)
Receiving objects: 100% (1107/1107), 279.90 KiB | 15.55 MiB/s, done.
Resolving deltas: 100% (535/535), done.
```
