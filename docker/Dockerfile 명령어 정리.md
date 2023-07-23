## Format
```Dockerfile
# comment
INSTRUCTION arguments
```
- `FROM` 으로 시작해야 한다
	- 예외적으로 comments, parser directives, 전역 ARGs가 앞에 올 수 있다

## Parser Directives
- Parser Directive는 Dockerfile의 하위 라인을 어떻게 처리할 지 결정한다
- Dockerfile의 최상위에 작성해야 한다
- `# directive=value` 형태로 작성한다
- 한 번에 하나의 directive만 적용된다
- 상위 라인에서 comments로 처리되면 하위에 parser directives가 존재하더라도 comments로 처리된다
- case-sensitive하지 않다
	- 컨벤션은 lower-case이다

```Dockerfile
# directive=value
FROM ImageName
```

## Environment replacement
```Dockerfile
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}   # WORKDIR /bar
ADD . $FOO       # ADD . /bar
COPY \$FOO /quux # COPY $FOO /quux
```

- 특정 명령어에서 사용할 수 있는 환경변수를 정의한다
- `ENV`를 사용해 작성한다
- `$variable_name`, `${variable_name}`으로 사용한다

## ARG
```Dockerfile
ARG <name>[=<default value>]
```

```Dockerfile
FROM busybox
ARG user1=someuser
ARG buildno
# ...
```

- 사용자가 빌드 타임에 전달할 수 있는 인자를 정의한다
- `ARG variable=default_variable` 형태로 기본값을 정의할 수 있다


#### Scope
```Dockerfile
FROM busybox
USER ${username:-some_user} # 1
ARG username
USER $username # 2
# ...
```

```bash
docker build --build-arg username=what_user .
```

1. 환경 변수 `username`이 정의되어 있지 않으므로 `some_user`가 할당된다
2. 인자로 전달된 `what_user`가 할당된다

```Dockerfile
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS
```

- `ARG`는 빌드 스테이지에 종속된다
	- multi-stage을 사용하면 각각 `ARG`를 정의해야 한다

## FROM
- 새로운 빌드 스테이지를 초기화한다
- 하위 명령어를 위한 기본 이미지를 설정한다

```Dockerfile
FROM [--platform=<platform>] <image> [AS <name>]

OR

FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]

OR

FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

```Dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

- `ARG` 명령어를 `FROM` 이전에 작성하여 이미지를 동적으로 설정할 수도 있다
- 한 `Dockerfile`에 여러개의 `FROM`을 사용할 수 있다
	- 각 `FROM`은 이전 스테이지에 대한 모든 상태를 제거한다
	- `AS`를 사용해 alias를 줄 수 있다
	- alias는 `FROM --from=alias` ,`COPY --from=alias` 형태로 사용할 수 있다
		- 현재 스테이지에서 alias에 해당하는 스테이지를 참조할 수 있다

## RUN

```Dockerfile
# shell form
RUN <command> 

# exec form
RUN ["executable", "param2", "param2"]
```

- 현재 이미지의 최상위에 위치한 새로운 레이어에서 명령어를 실행하고, 그 결과를 커밋한다
- 커밋한 결과는 이미지를 구성하는 새로운 레이어가된다
	- Dockerfile의 다음 단계를 실행하는데 새롭게 생성된 레이어가 사용될 수 있다

#### Shell form
```Dockerfile
RUN /bin/bash -c 'source $HOME/.bashrc && \
echo $HOME'
```

- `\`을 사용해 다음 라인에서도 `RUN` 명령어를 계속 작성할 수 있다

#### Exec form
```Dockerfile
RUN ["/bin/bash", "-c", "echo hello"]
```
- exec form은 shell form과 다르게 command shell을 실행하지 않는다
	- 예를 들어, `RUN [ "echo", "$HOME" ]`에서 `$HOME`에 환경 변수를 사용할 수 없다
	- `RUN [ "sh", "-c", "echo $HOME" ]`와 같이 사용해야 한다
- exec form은 Json Array로 변환되기 때문에 single quote(`'`)이 아닌 double-quotes(`"`)을 사용해야 한다

### RUN --mount
- 빌드 과정에서 접근할 수 있는 마운트를 생성한다

#### RUN --mount=type=cache
```Dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu
RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked \
  apt update && apt-get --no-install-recommends install -y gcc
```

|Option|Description|
|---|---|
|`id`|Optional ID to identify separate/different caches. Defaults to value of `target`.|
|`target`[1](https://docs.docker.com/engine/reference/builder/#fn:1)|Mount path.|
|`ro`,`readonly`|Read-only if set.|
|`sharing`|One of `shared`, `private`, or `locked`. Defaults to `shared`. A `shared` cache mount can be used concurrently by multiple writers. `private` creates a new mount if there are multiple writers. `locked` pauses the second writer until the first one releases the mount.|
|`from`|Build stage to use as a base of the cache mount. Defaults to empty directory.|
|`source`|Subpath in the `from` to mount. Defaults to the root of the `from`.|
|`mode`|File mode for new cache directory in octal. Default `0755`.|
|`uid`|User ID for new cache directory. Default `0`.|
|`gid`|Group ID for new cache directory. Default `0`.|

## CMD
```Dockerfile
# exec form, preferred form
CMD ["executable", "param1", "param2"]

OR

# as default parameters to ENTRYPOINT
CMD ["param1", "param2"]

OR

# shell form
CMD command param1 param2
```

- Dockerfile에는 단 하나의 `CMD`를 작성할 수 있다
- `CMD`의 주요 사용 목적은 컨테이너에 기본값을 제공하는 것이다

## EXPOSE
```Dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

```Dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```
- 도커 컨테이너가 특정 포트에서 네트워크 연결을 수신 대기하는 것을 명령한다


## ADD
```Dockerfile
ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>
ADD [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]
```

- `<src>`의 파일, 디렉토리 등을 이미지 파일 시스템의 `<dest>` 경로에 복사한다
	- `<src>`는 호스트의 파일 시스템 혹은 Remote URL이 될 수 있다
- 특정 포맷의 압축 파일의 경우 자동 압축 해제를 지원한다

## COPY
```Dockerfile
COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>
COPY [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]
```
- `<src>`의 파일, 디렉토리 등을 이미지 파일 시스템의 `<dest>` 경로에 복사한다
- 단순히 호스트의 파일 혹은 디렉토리를 복사하는 경우에는 `COPY`를 사용한다
- 압축 파일이라도 자동으로 압출을 해제하지 않는다


## ENTRYPOINT
```Dockerfile
# Exec form, preferred
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2
```
- 컨테이너가 실행될 때 실행되는 기본 실행 명령어나 스크립트를 설정하는 역할을 한다

### Understand how CMD and ENTRYPOINT interact
- Dockerfile은 최소한 하나의 `CMD`나 `ENTRYPOINT`를 포함해야 한다
- 컨테이너를 Executable로 사용하면 `ENTRYPOINT`는 반드시 정의되어야 한다
- `CMD`는 `ENTRYPOINT`의 기본 인자를 제공하거나, 컨테이너의 ad-hoc 명령어를 실행하는데 사용한다
- `CMD`는 인자와 함께 컨테이너를 실행하는 경우 덮어써진다

## USER
```Dockerfile
USER <user>[:<group>]

OR

USER <UID>[:<GID>]
```
- 현재 스테이지의 USER를 지정한다
- 지정된 USER가 `RUN`, `ENTRYPOINT`, `CMD`를 실행한다

## WORKDIR
```Dockerfile
WORKDIR /path/to/workdir
```
- 작업 디렉토리를 지정한다