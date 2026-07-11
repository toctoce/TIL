# 알고리즘 사이트 배포 - HTTPS 설정 (3)

## 1. Certbot 설치

```bash
sudo dnf install -y python3 python3-devel augeas-devel gcc

sudo python3 -m venv /opt/certbot
sudo /opt/certbot/bin/pip install --upgrade pip
sudo /opt/certbot/bin/pip install certbot certbot-nginx
sudo ln -sf /opt/certbot/bin/certbot /usr/local/bin/certbot

# 확인
certbot --version
```

## 2. 인증서 발급

```bash
sudo certbot --nginx -d algorithm.zzanggyu.com
```

다음 내용을 입력한다.

1. 인증서 만료 알림을 받을 이메일
2. 이용약관 동의
3. 이메일 공유 여부
4. HTTP를 HTTPS로 리다이렉트할지 선택

인증서가 생성된 것을 확인한다.

```bash
$ sudo ls -la /etc/letsencrypt/live/algorithm.zzanggyu.com
total 4
drwxr-xr-x. 2 root root  93 Jul 11 15:15 .
drwx------. 3 root root  50 Jul 11 15:15 ..
-rw-r--r--. 1 root root 692 Jul 11 15:15 README
lrwxrwxrwx. 1 root root  46 Jul 11 15:15 cert.pem -> ../../archive/algorithm.zzanggyu.com/cert1.pem
lrwxrwxrwx. 1 root root  47 Jul 11 15:15 chain.pem -> ../../archive/algorithm.zzanggyu.com/chain1.pem
lrwxrwxrwx. 1 root root  51 Jul 11 15:15 fullchain.pem -> ../../archive/algorithm.zzanggyu.com/fullchain1.pem
lrwxrwxrwx. 1 root root  49 Jul 11 15:15 privkey.pem -> ../../archive/algorithm.zzanggyu.com/privkey1.pem
```

Nginx 설정을 검사하고 반영한다.

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## 3. 인증서 자동 갱신 설정

인증서는 유효기간이 지나면 사용할 수 없으므로 만료되기 전에 갱신해야 한다.

`certbot renew` 명령어를 사용하면 인증서를 갱신할 수 있다.

- 갱신 시점이 아니면 아무 작업을 하지 않는다.
- 갱신 시점이라면 새 인증서를 발급한다.

Certbot은 인증서 유효기간이 3분의 1 미만으로 남았을 때 갱신을 시도한다. 따라서 갱신 날짜를 직접 계산할 필요 없이 `certbot renew`를 주기적으로 실행하면 된다.

하지만 인증서를 갱신해야하는지 날짜를 계속 체크하는 것은 매우 번거롭다. cron을 이용해서 이를 자동화시킬 수 있다.

`renew` 명령어가 정상적으로 동작하는지 확인한다.

```bash
sudo certbot renew --dry-run
```

성공한다면 `cron`을 설치하고 실행한다.

```bash
sudo dnf install -y cronie
sudo systemctl enable --now crond
```

```bash
sudo nano /etc/cron.d/certbot-renew
```

아래 내용을 입력한다.

```cron
0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot renew -q --deploy-hook "/usr/bin/systemctl reload nginx"
```

- `0 0,12 * * *`: 하루에 두 번 실행한다.
- `root`: 루트 권한으로 실행한다.
- `time.sleep(...)`: 실행 시간을 최대 한 시간까지 무작위로 지연한다.
- `/usr/local/bin/certbot renew`: 갱신 시점이 된 인증서를 갱신한다.
- `-q`: 오류를 제외한 출력을 생략한다.
- `--deploy-hook "/usr/bin/systemctl reload nginx"`: 인증서가 실제로 갱신된 경우에만 Nginx 설정을 다시 불러온다.

### Certbot 업데이트

공식 문서에서는 Certbot을 최신 상태로 유지하기 위해 매월 업데이트할 것을 권장한다. 따라서 한 달에 한 번 업데이트하자.

```bash
sudo /opt/certbot/bin/pip install --upgrade certbot certbot-nginx
```

---

![HTTPS 접속 확인](<알고리즘 사이트 배포 - HTTPS 설정 (3) 1.png>)

이제 HTTPS로 배포한 사이트에 접속할 수 있다.

## REFERENCE

- https://certbot.eff.org/instructions?os=pip&ws=nginx
