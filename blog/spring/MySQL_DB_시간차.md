Java에서 MySQL에 저장된 `datetime` 형식의 데이터를 가져올 때 시간차가 발생할 수 있다.<br/>
오늘은 이 시간 차이를 해소할 수 있는 방법에 대해서 알아본다.
## 문제 상황
---
토이 프로젝트 진행 중 MySQL에 저장된 `created_at`과 자바의 `createdAt`의 값이 일치하지 않는 경우가 발생했다. 
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134630169-1b02405f-52fa-4045-a453-b7f480f7d0b5.png height=300 width=200>
</p>

주문 후 DB에 저장된 값과 JSON으로 전달되어 웹에 렌더링한 값을 비교해본다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134630718-7d824337-64b9-4cd2-8808-8a17a28d587b.png width=600>
</p>

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134630599-baa96665-0427-4361-a0c1-6419d14c40b7.png width=600>
</p>

DB에는 created_at이 `2021-09-24T15:47:43` 으로 저장된 반면 웹 페이지에는 createdAt이 `2021-09-25T00:47:43`로 출력된다. 웹 페이지에 렌더링된 값 보다 DB에 저장된 값이 정확히 9시간이 빠르다.

## 문제 분석
---
웹 페이지에 렌더링된 값이 DB 보다 9시간이나 늦은 이유는 타임존이 달라서이다.<br/>
DB의 타임존을 먼저 확인해본다.

```mysql
SELECT @@global.time_zone @@session.time_zone
```
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134632700-ed23ad70-4a51-46a3-9c6b-c4584ec56887.png width=600>
</p>

현재 DB의 타임존이 별도의 설정 없이 SYSTEM을 따르고 있다. 따라서 현재는 한국 시간 기준으로 시간이 저장된다. 

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134632954-b6f5f615-43c0-4943-be50-f0fbfebe960d.png width=600>
</p>

그리고 현재 application.properties에 작성된 설정에는 `serverTimeZone=UTC`로 되어있다. 따라서 DB의 타임존과 application.properties의 타임존의 불일치로인해 시간차가 발생한다.

## 문제 해결
---
타임존의 불일치를 해결하기 위해서는 **application.properties에서 타임존을 변경** 해야 한다.

### application.properties에서 타임존 변경하기

application.properties에서 타임존을 변경하는 것은 간단하다. `serverTimezone=UTC`를 `serverTimezone=Asia/Seoul`로 변경하면된다. 

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134634774-08c584be-cc3a-48b0-8f5a-ce5a87fe8186.png width=600>
</p>

`serverTimezone=Asia/Seoul`로 변경 후 두 데이터를 비교하면 동일한 것을 볼 수 있다.

<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134635074-05b354e4-cefa-4788-af48-5954779437a7.png width=600>
</p>
<p align=middle>
    <img src=https://user-images.githubusercontent.com/60502370/134630718-7d824337-64b9-4cd2-8808-8a17a28d587b.png width=600>
</p>

그리고 DB의 타임존을 변경하는 방법이 있는데 [여기](https://hunda.tistory.com/entry/MySQL-Windows-%EC%97%90%EC%84%9C-Timezone-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)를 참고하길 바란다. 아직 DB 자체 타임존을 변경 해야할 이유를 잘모르겠지만 나중에 변경이 필요한 경우가 생기면 게시물을 작성하기로 한다.
---
**참고**
- https://blog.jojonari.dev/entry/LocalDateTime%EC%9D%98-%EB%8D%B0%EC%9D%B4%ED%84%B0%EA%B0%80-9%EC%8B%9C%EA%B0%84-%EC%B0%A8%EC%9D%B4%EB%82%A8