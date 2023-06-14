- Helm은 쿠버네티스 전용 패키지 매니저이다
- 패키지 매니저는 외부에 있는 저장소에 패키지 정보를 받아와 패키지를 안정적으로 관리하는 도구이다
	- 설치에 필요한 의존성 파일들을 관리하고 간단히 설치할 수 있도록 도와준다

## Chart
- 헬름의 패키지를 말한다
- 차트는 특정 디렉터리 구조를 가진 파일들로 생성된다
- 공개된 차트를 다운로드 받기 위해서 `helm pull chartrepo/charname` 명령을 사용할 수 있다

### 차트 파일 구조
```TEXT
wordpress/
  Chart.yaml          # 차트에 대한 정보를 가진 YAML 파일
  LICENSE             # 옵션: 차트의 라이센스 정보를 가진 텍스트 파일
  README.md           # 옵션: README 파일
  values.yaml         # 차트에 대한 기본 환경설정 값들
  values.schema.json  # 옵션: values.yaml 파일의 구조를 제약하는 JSON 파일
  charts/             # 이 차트에 종속된 차트들을 포함하는 디렉터리
  crds/               # 커스텀 자원에 대한 정의
  templates/          # values와 결합될 때, 유효한 쿠버네티스 manifest 파일들이 생성될 템플릿들의 디렉터리
  templates/NOTES.txt # 옵션: 간단한 사용법을 포함하는 텍스트 파일
```

- 예시
<img width="335" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/e023d24d-8e55-44d1-b140-ea6bd010364b">


### Chart.yaml
- 차트의 필수 파일이다
```yaml
apiVersion: 차트 API 버전 (필수)
name: 차트명 (필수)
version: SemVer 2 버전 (필수)
kubeVersion: 호환되는 쿠버네티스 버전의 SemVer 범위 (선택)
description: 이 프로젝트에 대한 간략한 설명 (선택)
type: 차트 타입 (선택)
keywords:
  - 이 프로젝트에 대한 키워드 리스트 (선택)
home: 프로젝트 홈페이지의 URL (선택)
sources:
  - 이 프로젝트의 소스코드 URL 리스트 (선택)
dependencies: # 차트 필요조건들의 리스트 (optional)
  - name: 차트명 (nginx)
    version: 차트의 버전 ("1.2.3")
    repository: 저장소 URL ("https://example.com/charts") 또는 ("@repo-name")
    condition: (선택) 차트들의 활성/비활성을 결정하는 boolean 값을 만드는 yaml 경로 (예시: subchart1.enabled)
    tags: # (선택)
      - 활성화 / 비활성을 함께하기 위해 차트들을 그룹화 할 수 있는 태그들
    enabled: (선택) 차트가 로드될수 있는지 결정하는 boolean
    import-values: # (선택)
      - ImportValues 는 가져올 상위 키에 대한 소스 값의 맵핑을 보유한다. 각 항목은 문자열이거나 하위 / 상위 하위 목록 항목 쌍일 수 있다.
    alias: (선택) 차트에 대한 별명으로 사용된다. 같은 차트를 여러번 추가해야할때 유용하다.
maintainers: # (선택)
  - name: maintainer들의 이름 (각 maintainer마다 필수)
    email: maintainer들의 email (각 maintainer마다 선택)
    url: maintainer에 대한 URL (각 maintainer마다 선택)
icon: 아이콘으로 사용될 SVG나 PNG 이미지 URL (선택)
appVersion: 이 앱의 버전 (선택). SemVer인 필요는 없다.
deprecated: 차트의 deprecated 여부 (선택, boolean)
annotations:
  example: 키로 매핑된 주석들의 리스트 (선택).
```

- `dependencies` 에는 차트의 의존성을 나타낸다
- `helm dependency update` 를 실행하면 `charts/` 디렉터리 안에 다운로드 받는다
- 예를 들어 mysql, apache에 대한 의존성이 정의 되어 있으면 아래와 같이 다운로드 받는다

```text
charts/
  apache-1.2.3.tgz
  mysql-3.2.1.tgz
```

### values.yaml
- 헬름의 빌트인 객체중 `Values`가 있다
	- `values.yalm` 파일 및 사용자 제공 파일에서 템플릿으로 전달된 값이다. 기본적으로 비어있다

- `values.yaml`
```yalm
image: in28min/currency-exchange:0.0.1-RELEASE  
port: 8000  
servicetype: LoadBalancer  
replicas: 1  
name: currency-exchange
```

- deployment, service yaml
	- [go template](https://www.joinc.co.kr/w/man/12/golang/networkProgramming/template) 으로 빌트인 객체의 값을 사용할 수 있다

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: {{ .Values.name}}  
  labels:  
    app: {{ .Values.name}}  
spec:  
  replicas: {{ .Values.replicas}}  
  selector:  
    matchLabels:  
      app: {{ .Values.name}}  
  template:  
    metadata:  
      labels:  
        app: {{ .Values.name}}  
    spec:  
      containers:  
      - name: {{ .Values.name}}  
        image: {{ .Values.image}}  
        env:  
        - name: CURRENCY_EXCHANGE_URI  
          value: http://currency-exchange:8000  
        ports:  
        - containerPort: {{ .Values.port}}  
---  
apiVersion: v1  
kind: Service  
metadata:  
  name: {{ .Values.name}}  
  labels:  
    app: {{ .Values.name}}  
spec:  
  type: {{ .Values.servicetype}}  
  ports:  
  - port: {{ .Values.port}}  
    protocol: TCP  
  selector:  
    app: {{ .Values.name}}
```
 

## Repository
- 헬름 차트 파일을 공유할 수 있는 저장소이다
- 기본 저장소는 [아티팩트 허브](https://artifacthub.io/) 이다
- 레포지토리 추가: `helm repo add bitnami https://charts.bitnami.com/bitnami` 
- 레포지토리 설치 가능 차트 목록: `helm search repo bitnami`:
- 레포지토리 차트 설치: `helm install bitnami/mysql --generate-name`\

## Release
- 릴리즈는 설치된 차트를 말한다
- 릴리즈 확인: `helm list`
- 릴리즈 제거: `helm uninstall release-name`
- 릴리즈 제거(history 유지): `helm uninstall release-name --keep-history`
- 롤백, 업그레드 모두 가능하니 명령어를 확인해볼 것

