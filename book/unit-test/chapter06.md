# 6. 단위 테스트 스타일
## 6.1 단위 테스트의 세 가지 스타일
- 출력 기반 테스트(output-based testing)
- 상태 기반 테스트(state-based testing)
- 통신 기반 테스트(communication-based testing)

### 6.1.1 출력 기반 테스트 정의
- 테스트 대상 시스템에 입력을 넣고 생성되는 출력을 점검하는 방식이다
- 이러한 테스트 스타일은 부작용이 없고 SUT의 작업 결과는 호출자에게 반환하는 값 뿐이다

### 6.1.2 상태 기반 스타일 정의
- 작업이 완료된 후 시스템의 상태를 확인하는 것이다
- 상태라는 용어는 SUT나 협력자 중 하나 또는 데이터베이스 같은 프로세스 외부 의존성의 상태 등을 의미한다

### 6.1.3  통신 기반 스타일 정의
- 목을 사용해 테스트 대상 시스템과 협력자 간의 통신을 검증한다

## 6.2 단위 테스트 스타일 비교
### 6.2.1 회귀 방지와 피드백 속도 지표로 스타일 비교하기
- 회귀 방지 지표는 특정 스타일에 따라 달라지지 않는다
- 회귀 방지 지표는 다음 세 가지 특성으로 결정된다
  - 테스트 중에 실행되는 코드 양
  - 코드 복잡도
  - 도메인 유의성
- 테스트 스타일과 테스트 피드백 속도 사이에는 상관관계가 거의 없다
  - 단위 테스트 영역에서는 모든 테스트의 속도가 비슷하다

### 6.2.2 리팩터링 내성 지표로 스타일 비교하기
- 리팩터링 내성은 리팩터링 중에 발생하는 거짓 양성 수에 대한 척도다
- 츨력기반 테스트는 테스트가 테스트 대상 메서드에만 결합되므로 거짓 양성 방지가 가장 우수하다
- 상태기반 테스트는 일반적으로 거짓 양성이 되기 쉽다
  - 상태 테스트는 큰 API 노출 영역에 의존하므로 구현 세부 사항과 결합할 가능성도 높다
- 통신 기반 테스트가 허위 경부에 가장 취약하다. 테스트 대역으로 상호 작용을 확인하는 테스트는 대부분 깨지기 쉽다

### 6.2.3 유지 보수성 지표로 스타일 비교하기
- 유지 보수성 지표는 단위 테스트 스타일과 밀접한 관련이 있다
- 유지 보수성은 단위 테스트 유지비를 측정하며 다음 두 가지 특성으로 정의 한다
  - 테스트를 이해하기 얼마나 어려운가?(테스트 크기에 대한 함수)
  - 테스트를 실행하기 얼마나 어려운가?(테스트에 직접적으로 관련 있는 프로세스 외부 의존성 개수에 대한 함수)
- 테스트가 크거나 프로세스 위부 의존성 개수가 많으면 유지보수가 어렵다

- 출력 기반 테스트가 유지 보수하기 쉽다
  - 거의 항상 짧고 간결하다
  - 전역 상태나 내부 상태를 변경하지 않는다
  - 프로세스 외부 의존성을 다루지 않는다

- 상태 기반 테스트는 출력 기반 테스트보다 유지 보수가 쉽지 않다
  - 상태 검증은 종종 출력 검증보다 더 많은 공간을 차지한다
  - 헬퍼 메서드, 검증 대상 클래스의 동등 멤버를 정의(값 객체 사용) 하는 등 테스트를 단축하는 기법이 존재한다

- 통신 기반 테스트는 유지 보수성 지표에서 출력 기반 테스트와 상태 기반 테스트보다 점수가 낮다
  - 테스트 대역과 상호 작용 검증을 설정해야 한다
  - 목이 사슬 형태로 있을 때 테스트는 커지고 유지 보수하기 어려워진다

### 6.2.4 스타일 비교하기: 결론
- 항상 다른 것보다 출력 기반 테스트를 선호하라
- 하지만 말하기는 쉬워도 행하기는 어렵다. 출력 기반 스타일을 함수형으로 작성된 코드에만 적용할 수 있고, 대부분의 객체지향 프로그래밍 언어에는 해당하지 않는다
- 코드를 순수 함수로 만들면 상태 기반, 통신 기반 테스트 대신 출력 기반 테스트가 가능해진다

## 6.3 함수형 아키텍처 이해
### 6.3.1 함수형 프로그래밍이란?
- 함수형 프로그래밍은 수학적 함수를 사용한 프로그래밍이다
- 수학적 함수는 숨은 입출력이 없는 함수다
  - 수학적 함수는 호출 횟수에 상관없이 주어진 입력에 대해 동일한 출력을 생성한다
- 수학적 함수는 테스트가 짧고 간결하며 이해하고 유지보수하기 쉬우므로 테스트하기가 매우 쉽다
- 숨은 입출력은 테스트를 하기 어렵게 한다. 숨은 입출력의 유형은 아래와 같다
  - 부작용: 부작용은 메서드 시그니처에 표시되지 않은 출력이다. 연산이 클래스 인스턴스의 상태를 변경하는 등 부작용을 발생시킨다
  - 예외: 메서드가 예외를 던지면 프로그램 흐름에 메서드 시그니처에 설정된 계약을 우회하는 경로를 만든다.
  - 내외부 상태에 대한 참조: 데이터베이스에서 질의할 수 있고, 비공개 변경 가능 필드를 참조할 수도 있다. 메서드 시그니처에 없는 실행 흐름에 대한 입력이므로 숨어있다

- 메서드가 수학적 함수인지 판별하는 가장 좋은 방법은 프로그램의 동작을 변경하지 않고 해당 메서드에 대한 호출을 반환 값으로 대체할 수 있는지 확인하는 것이다
  - 메서드 호출을 해당 값으로 바꾸는 것을 참조 투명성이라 한다

### 6.3.2 함수형 아키텍처란?
- 함수형 프로그래밍의 목표는 비즈니스 로직을 처리하는 코드와 부작용을 일으키는 코드를 분리하는 것이다
  - 부작용을 완전히 없앨 수는 없다
- 다음 두 가지 코드 유형을 구분해서 비즈니스 로직과 부작용을 분리할 수 있다
  - 결정을 내리는 코드: 부작용이 필요 없기 때문에 수학적 함수를 사용해 작성할 수 있다
  - 해당 결정에 따라 작용하는 코드: 수학적 함수에 의해 이뤄진 모든 결정을 데이터베이스의 변경이나 메시지 버스로 전송된 메시지와 같이 가시적인 부분으로 변환한다
- 결정을 내리는 코드는 종종 함수형 코어, 결정에 따라 작용하는 코드는 가변 셀이라고 한다

- 함수형 코어와 가변 셸은 다음과 같은 방식으로 협력한다
  - 가변 셸은 모든 입력을 수집한다
  - 함수형 코어는 결정을 생성한다
  - 셸은 결정을 부작용으로 변환한다
- 두 계층을 잘 분리하기 위해서는 가변 셸이 의사 결정을 추가하지 않게끔 결정을 나타내는 클래스에 정보가 충분히 있는지 확인해야 한다
  - 가변 셸은 아무 말도 하지 않아야 한다

- **객체지향 프로그래밍은 작동 부분을 캡슐화해 코드를 이해할 수 있게 한다. 함수형 프로그래밍은 작동 부분을 최소화해 코드를 이해할 수 있게 한다**

### 6.3.3 함수형 아키텍처와 육각형 아키텍처의 비교
- 결정과 실행을 분리한다
  - 육각형 아키텍처는 도메인 계층과 애플리케이션 서비스 계층을 구별한다. 도메인 계층은 비즈니스 로직에 책임이 있고, 애플리케이션 서비스 계층은 외부 애플리케이션과 통신에 책임이 있다
- 의존성 간의 단방향 흐름이다
  - 육각형 아키텍처에서 도메인 계층 클래스는 서로에게만 의존하며, 애플리케이션 서비스 계층의 클래스에 의존하지 않는다
  - 함수형 아키텍처에서 불변 코어는 가변 셸에 의존하지 않는다

- 차이점은 부작용 처리에 있다
  - 함수형 아키텍처는 모든 부작용을 불변 코어에서 비즈니스 연산 가장자리로 밀어낸다
  - 육각형 아키텍처는 도메인 계층에 제한하는 한, 도메인 계층으로 인한 부작용도 문제 없다
    - 도메인 클래스 인스턴스는 데이터베이스에 직접 저장할 수는 없지만 상태 변경이 가능하며, 애플리케이션 서비스에서 변경 사항을 데이터베이스에 저장한다

## 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환
### 6.4.1 감사 시스템 소개
- 조직의 모든 방문자를 추적하는 샘플 프로젝트

```java
public class AuditManager {
    private int maxEntriesPerFile;
    private String directoryName;

    public AuditManager(int maxEntriesPerFile, String directoryName) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVisit) throws IOException {
        List<File> files = new ArrayList<>(List.of(Paths.get(directoryName).toFile().listFiles()));
        files.sort(Comparator.comparing(File::getName));

        String newRecord = String.format("%s;%s", visitorName, timeOfVisit);

        if (files.isEmpty()) {
            File file = Path.of(directoryName, "audit_1.txt").toFile();
            write(newRecord, file);
            return;
        }

        int currentFileIndex = files.size();
        File currentFile = files.get(files.size() - 1);
        try(Stream<String> lineStream = Files.lines(currentFile.toPath())) {
            List<String> lines = new ArrayList<>(lineStream.toList());

            if (lines.size() < maxEntriesPerFile) {
                lines.add(newRecord);
                String newContent = String.join("\r\n", lines);
                write(newContent, currentFile);
            } else {
                int newIndex = currentFileIndex + 1;
                String newName = String.format("audit_%d.txt", newIndex);
                Path path = Path.of(directoryName, newName);
                write(newRecord, path.toFile());
            }
        }

    }

    private void write(String newRecord, File file) throws IOException {
        try(FileWriter writer = new FileWriter(file)) {
            writer.write(newRecord);
        }
    }
}
```

- `AuditManager` 클래스는 파일 시스템과 밀접하게 관련있어 테스트가 어렵다
  - 테스트 전에 파일을 올바른 위치에 배치하고 테스트가 끝나면 해당 파일을 읽고 내용을 확인한 후 삭제해야 한다
- 파일 시스템이 병목지점이다
  - 테스트의 실행 흐름을 방해한다
  - 테스트를 느리게 한다

### 6.4.2 테스트를 파일 시스템에서 분리하기 위한 목 사용
- 테스트가 밀접하게 결합된 문제는 일반적으로 파일 시스템을 목으로 처리해 해결한다
- 파일의 모든 연산을 별도의 클래스로 도출하고 `AuditManager`에 생성자로 해당 클래스를 주입할 수 있다

```java
public interface FileSystem {
    List<File> getFiles(String directoryName);

    void write(String newRecord, File file);

    List<String> readLines(File file);
}
```

```java
public class AuditManager {
    private final int maxEntriesPerFile;
    private final String directoryName;
    private final FileSystem fileSystem;

    public AuditManager(int maxEntriesPerFile, String directoryName, FileSystem fileSystem) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
        this.fileSystem = fileSystem;
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVisit) {
        List<File> files = fileSystem.getFiles(directoryName);
        files.sort(Comparator.comparing(File::getName));

        String newRecord = String.format("%s;%s", visitorName, timeOfVisit);

        if (files.isEmpty()) {
            File file = Path.of(directoryName, "audit_1.txt").toFile();
            fileSystem.write(newRecord, file);
            return;
        }

        int currentFileIndex = files.size();
        File currentFile = files.get(files.size() - 1);
        List<String> lines = fileSystem.readLines(currentFile);

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join("\r\n", lines);
            fileSystem.write(newContent, currentFile);
        } else {
            int newIndex = currentFileIndex + 1;
            String newName = String.format("audit_%d.txt", newIndex);
            Path path = Path.of(directoryName, newName);
            fileSystem.write(newRecord, path.toFile());
        }

    }
}
```
- `AuditManager`가 이제 파일 시스템에서 분리되어 공유 의존성이 사라지고 테스트를 서로 독립적으로 실행할 수 있다
- 파일 시스템과의 통신과 이러한 통신의 부작용은 애플리케이션의 식별할 수 있는 동작이다
- 따라서 목을 사용하여 테스트할 수있을 것이다

### 6.4.3 함수형 아키텍처로 리팩터링 하기
```java
public class AuditManagerV2 {
    private final int maxEntriesPerFile;

    public AuditManagerV2(int maxEntriesPerFile) {
        this.maxEntriesPerFile = maxEntriesPerFile;
    }

    public FileUpdate addRecord(FileContent[] files, String visitorName, LocalDateTime timeOfVisit) throws IOException {
        Arrays.sort(files, Comparator.comparing(FileContent::getFileName));

        String newRecord = String.format("%s;%s", visitorName, timeOfVisit);

        if (files.length == 0) {
            new FileUpdate("audit_1.txt", newRecord);
        }

        int currentFileIndex = files.length;
        FileContent currentFile = files[files.length - 1];

        List<String> lines = Arrays.asList(currentFile.getLines());

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join("\r\n", lines);
            return new FileUpdate(currentFile.getFileName(), newContent);
        } else {
            int newIndex = currentFileIndex + 1;
            String newName = String.format("audit_%d.txt", newIndex);
            return new FileUpdate(newName, newRecord);
        }
    }
}
```

```java
public class FileContent {
    public final String fileName;
    public final String[] lines;

    public FileContent(String fileName, String[] lines) {
        this.fileName = fileName;
        this.lines = lines;
    }

    public String getFileName() {
        return fileName;
    }

    public String[] getLines() {
        return lines;
    }
}
```

```java
public class FileUpdate {
    public final String fileName;
    public final String newContent;
    public FileUpdate(String fileName, String newContent) {
        this.fileName = fileName;
        this.newContent = newContent;
    }
}
```

```java
public class Persister {
    public FileContent[] readDirectory(String directoryName) {
        return Arrays.stream(Objects.requireNonNull(Paths.get(directoryName).toFile().listFiles()))
                .map(file -> {
                    try {
                        return new FileContent(file.getName(), Files.lines(file.toPath()).toArray(String[]::new));
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                })
                .toArray(FileContent[]::new);
    }

    public void applyUpdate(String directoryName, FileUpdate update) {
        File file = Path.of(directoryName, update.fileName).toFile();
        try(FileWriter writer = new FileWriter(file)) {
            writer.write(update.newContent);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class ApplicationService {
    private final String directoryName;
    private final AuditManagerV2 auditManager;
    private final Persister persister;

    public ApplicationService(String directoryName, int maxEntriesPerFile) {
        this.directoryName = directoryName;
        this.auditManager = new AuditManagerV2(maxEntriesPerFile);
        this.persister = new Persister();
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVisit) throws IOException {
        FileContent[] fileContents = persister.readDirectory(directoryName);
        FileUpdate fileUpdate = auditManager.addRecord(fileContents, visitorName, timeOfVisit);
        persister.applyUpdate(directoryName, fileUpdate);
    }

    public static void main(String[] args) {
        ApplicationService applicationService = new ApplicationService("/Users/user/Workspace/study-code/unit-test/file", 1);
        try {
            applicationService.addRecord("hi", LocalDateTime.now());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
- `AuditManagerV2`는 함수형 아키텍처에서 불변의 함수형 코어 역할을 한다
- `Persister`는 가변 셸 역할을 한다
- `FileContent`는 함수형 코어의 인풋, `FileUpdate`는 함수형 코어의 아웃풋이다
- 이제 목을 사용하지 않고 `AuditManagerV2`를 테스트할 수 있다

### 6.4.4 예상되는 추가 개발
- 생략

## 6.5 함수형 아키텍처의 단점 이해하기
### 6.5.1 함수형 아키텍처 적용 가능성
- 감사 시스템은 결정을 내리기 전에 입력을 모두 수집할 수 있으므로 함수형 아키텍처가 잘 작동했다
  - 하지만 상황에 따라 의사 결정 절차 중간 결과에 따라 프로세스 외부 의존성에 추가 데이터 질의를 할 수도 있다
  - 숨은 입력이 생기면 더 이상 수학적 함수가 될 수 없다

- 일단 도메인 모델이 데이터베이스에 의존하는 것은 좋은 생각이 아니다

### 6.5.2 성능 단점
- 함수형 아키텍처는 결정을 내리기 전 입력을 모두 수집해야한다
- 따라서 프로세스 외부 의존성을 더 많이 호출하며 성능이 떨어진다
- 함수형 아키텍처와 전통적인 아키텍처 사이의 선택은 성능과 코드 유지 보수성간의 절충이다

### 6.5.3 코드베이스 크기 증가
- 함수형 아키텍처는 함수형 코어와 가변 셸 사이를 명확하게 분리해야 한다
  - 이를 통해 코드 복잡도가 낮아지고 유지 보수성이 향상된다
  - 하지만 코딩이 더 필요하다
- 항상 시스템의 복잡도와 중요성을 고려해 함수형 아키텍처를 전략적으로 적용하라
- 대부분의 프로젝트에서는 모든 도메인 모델을 불변으로 할 수 없기 때문에 출력 기반 테스트만 의존할 수 없다
