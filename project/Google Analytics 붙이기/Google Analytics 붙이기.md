# Google Analytics 붙이기

우테코 코치에게 배포에 대한 자문을 구했다. 배포 전에 Google Analytics를 붙이라는 조언을 받아 `GA(Google Analytics)`를 붙이려고 한다.

[공식 문서](https://support.google.com/analytics/answer/9304153?hl=ko)를 참고해 적용한 과정을 작성해 보겠다.

## 1. 계정 생성

[Google Analytics](https://analytics.google.com)에 접속해 측정 시작을 클릭한다.

![비즈니스 목표 선택](<Google Analytics 붙이기 1.png>)

비즈니스 목표로는 `웹 또는 앱 트래픽 파악`, `사용자 참여 발생 시간 및 유지율 보기`를 선택하였다.

계정을 생성하면서 `GA4 속성`을 만들고 웹 데이터 스트림을 추가한다.

## 2. 태그 추가

데이터 스트림 → 웹 → 웹 데이터 스트림 → Google 태그 → 태그 안내 보기 → 직접 설치로 이동한다.

`<!-- Google tag (gtag.js) -->`로 시작하는 코드를 복사해 `frontend/index.html`의 `<head>` 바로 다음에 추가한다.

이 서비스는 History API로 화면을 전환하는 `SPA`이므로 데이터 스트림의 향상된 측정에서 페이지 조회의 `페이지 로드`와 `브라우저 방문 기록 이벤트를 기반으로 한 페이지 변경사항`을 활성화한다.

## 3. 분석

![Google Analytics 보고서](<Google Analytics 붙이기 2.png>)

태그를 추가한 후 실시간 보고서에서 데이터 수집 여부를 확인한다. 데이터는 약 10~15분 안에 확인할 수 있으며, 일반 보고서에 반영되기까지 24~48시간이 걸릴 수 있다.
