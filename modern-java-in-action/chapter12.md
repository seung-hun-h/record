# 새로운 날짜와 시간 API
## LocalDate, LocalTime, Instant, Duration, Period 클래스
### LocalDate와 LocalTime 사용

- `LocalDate` 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체이다
  - 연도, 달, 요일 등을 반환하는 메서드를 제공한다

```java
LocalDate date = LocalDate.of(2017, 9, 21);
int year = date.getYear();
Month month = date.getMonth();
int day = date.getDayOfMonth();
DayOfWeek dow = date.getDayOfWeek();
int len = date.lengthOfMonth();
boolean leap = date.isLeapYear(); // 윤년
```

- get 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다
  - TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스이다
  - ChronoField는 TemporalField 인터페이스를 정의한다

```java
// ChronoField
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);

// 내장 메서드
int year = date.getYear();
int month = date.getMonthValue();
int day = date.getDayOfMonth();
```

- `LocalTime` 인스턴스는 시간을 표현할 수 있다

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
...
```

- 두 클래스는 문자열을 파싱하여 인스턴스를 생성할 수도 있다
  - `LocalDate.parse("2017-09-21")`
  - `LocalTime.parse("13:45:20")`
  - parse에 DateTimeFormatter를 전달할 수도 있다

### 날짜와 시간 조합
- `LocalDateTime`은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다

```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(time);
LocalDateTime dt4 = time.atDate(date);
```

### Instant 클래스: 기계의 날짜와 시간
- 주, 날짜, 시간, 분은 사람 기준의 시간이다
- 기계에서는 이러한 단위로 시간을 표현하기 어렵다
- 기계 관점에서는 연속도니 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 자연스럽다
- `Instant`는 기계적인 관점에서 시간을 표현한다
  - 유니크 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지 시간을 초로 표현한다

```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000);
Instant.ofEpochSecond(4, -1_000_000_000);
```

### Duration과 Period 정의
- 지금까지 살펴본 모든 클래스는 Temporal 인터페이스를 구현한다
  - Temporal은 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다
- Duration은 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다
  - Duration은 초와 나노초로 시간 단위를 표현하므로 LocalDate는 전달할 수 없다

```java
Duration d1 = Duration.between(time1, time2);
Duration d2 = Duration.between(dateTime1, dateTime2);
Duration d3 = Duration.between(instant1, instant2);
```

- Period는 두 LocalDate의 차이를 확인할 수 있다

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11),
                                LocalDate.of(2017, 9, 21));
```


지금까지 살펴본 클래스는 모두 불편이다. 불변 클래스는 함수형 프로그래밍 그리고 스레드 안전성과 도메인 모델의 일관성을 유지하는데 좋은 특징이다.

## 날짜 조정, 파싱, 포매팅
- withAttribute 메서드로 기존의 LocalDate를 바꾼 객체를 쉽게 얻을 수 있다

```java
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.withYear(2011) // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25) // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2) // 2011-02-25
```

- Temporal 인터페이스는 특정 시간을 정의한다
  - get, with로 Temporal 객체의 필드값을 읽거나 고칠 수 있다.(복사를 만든다)

- 선언형으로 LocalDate를 사용하는 방법도 있다

```java
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = date1.plusWeek(1) // 2017-09-28
LocalDate date3 = date2.minusYear(6) // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoField.MONTHS) // 2012-03-28
```

### TemporalAdjusters 사용하기
- TemporalAdjusters는 복잡한 날짜 조정 기능을 사용할 수 있도록 TemporalAdjuster를 제공한다

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lasyDayOfMonth());
```

### 날짜와 시간 객체 줄력과 파싱
- DateTimeFormatter를 이용해서 날짜나 시간을 특정 형식의 문자열로 만들 수 있다

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```

- 팩토리 메서드 parse를 이용해서 문자열을 날짜 객체로 만들 수 있다

```java
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

- DateTimeFormatter는 스레드 안전하고, 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
```

- ofPattern 메서드도 Locale로 포매터를 만들 수 있도록 오버로드된 메서드를 제공한다

```java
DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d. MMMM yyyy", Locale.ITALIAN);
```

- DateTimeFormatterBuilder 클래스로 복합적인 포매터를 정의해서 좀 더 세부적으로 포매터를 제어할 수 있다

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
                                    .appendText(ChronoField.DAY_OF_MONTH)
                                    .appendLiteral(". ")
                                    .appendText(ChronoField.MONTH_OF_YEAR)
                                    .appendLiteral(" ")
                                    .appendText(ChronoField.YEAR)
                                    .parseCaseInsensitive()
                                    .toFormatter(Locale.ITALIAN);
```

## 다양한 시간대와 캘린더 활용 방법
- `java.time.ZoneId`를 사용하면 서머타임(DST)와 같은 복잡한 사항이 자동으로 처리된다
  - ZoneId는 불변 클래스이다

### 시간대 사용하기
- 표준 시간이 같은 지역을 묶어서 시간대(time zone) 규칙 집합을 정의한다
- ZoneRules 클래스에는 40개 정도의 시간대가 존재한다
- ZoneId의 getRules()를 이용해서 해당 시간대의 규정을 획득할 수 있다
- ZoneId는 지역을 구분하는 지역 ID이다.
  - 지역/도시 형식으로 이루어진다

- ZoneId 객체는 LocalDate, LocalDateTime, Instant를 이용해서 ZonedDateTime 인스턴스로 변환될 수 있다

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone);
```

### UTC/Greenwich 기준의 고정 오프셋
- ZoneId의 서브 클래스인 ZoneOffset 클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있다

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

- 하지만 ZoneOffset은 서머타임을 제대로 처리할 수 없으므로 권장하지 않는 방식이다
- ZoneOffset은 오프셋으로 날짜와 시간을 표현하는 OffsetDateTime을 만둘 수 있다

```java
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

### 대안 캘린더 시스템 사용하기
- ISO-8601 캘린더 시스템은 실질적으로 전 세계에서 통용된다.
- 자바 8에서는 추가로 ThaiBuddihistDate, MinguoDate, JapaneseDate, HijrahDate를 제공한다

