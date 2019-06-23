# Lesson 6: CronTrigger

**CronTrigger**는 스케줄을 정확한 시각을 지정하기보다 cron 표현식으로 나타내고자 할 때 **SimpleTrigger**보다 더 유용합니다. 

**CronTrigger**를 통해 "매주 금요일 정오"나 "평일 오전 9시 30분", "1월 매 주 월/수/금요일 오전 9시에서 10시 사이 5분 간격" 등을 표현할 수 있습니다.

또한 **SimpleTrigger**처럼 `startTime`을과 `endTime`을 강제로 지정할 수 있습니다.

## Cron 표현식

**Cron 표현식**은 **Cron Trigger** 객체를 설정할 때 사용합니다. **Cron 표현식**은 스케줄 단위를 표현하는 7개의 하위 표현식으로 구성된 문자열입니다. 각 하위 표현식은 공백으로 구별되며, 각각 다음과 같습니다.

1. 초
2. 분
3. 시
4. 일
5. 월
6. 요일
7. 년 (선택 항목)

**Cron 표현식**의 예시를 들면 "0 0 12 ? * WED"가 있습니다. 이는 "매주 수요일 오전 12시"를 의미합니다.

각 하위 표현식은 구간을 표현할 수 있습니다. 예를 들어 "요일" 표현의 경우 "MON-FRI", "MON,WED,FRI"와 같이 쓰거나 "MON-WED,SAT"처럼 나타낼 수 있습니다.

각 하위 표현식은 당연히 올바른 날짜를 나타내기 위한 유효 값 범위가 있습니다. "초"와 "분"은 0부터 59까지의 수만 가능하고, "시"는 0부터 23까지의 수만 가능합니다. "일"은 1부터 31까지의 수가 될 수 있지만, 아시다시피 지정한 "월"이 언제냐에 따라 달라집니다. "월"은 0부터 11까지의 수 또는 JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV, DEC처럼 문자열로도 나타낼 수 있습니다. "요일"은 1부터 7까지의 수를 사용하거나 마찬가지로 SUN, MON, TUE, WED, THU, FRI, SAT 처럼 문자열로도 나타냅니다.

슬래시 '/' 문자는 시간 증가를 표현합니다. 예를 들어 "분"을 "0/15"으로 표현하는 것은 정각부터 15분 간격을 의미합니다. "분"을 "3/20"으로 사용하면 정각 3분부터 매 20분을 의미하게 됩니다. 이를 리스트로 표현하면 "3,23,43"과 같습니다. "/35"는 매 35분을 의미하는 게 아니라, 정각부터 매 35분 그러니까 "0,35"를 나타내는 것입니다.

물음표 '?' 문자는 오직 "일"과 "요일"에서만 사용할 수 있습니다. 이 문자의 뜻은 "아무런 값도 지정하지 않음"입니다. 이 문자는 "일"과 "요일" 중 어느 한 쪽에만 값을 지정하고자 할 때 사용합니다. 보다 명확한 설명은 아래의 예제를 참고하거나 **CronTrigger**의 JavaDoc 문서를 참고하세요.

'L' 문자도 "일"과 "요일"에서만 사용합니다. 이 문자의 뜻은 "마지막 시각"으로 사용되지만, 두 필드에서의 의미가 미묘하게 다릅니다. 예를 들어 "일" 필드에 'L' 값을 사용하면 "그 달의 마지막 날"로 해석됩니다. 즉, 1월 31일, 2월 28일 (윤년이 아닌 경우)가 되는 것이죠. "요일" 필드에 'L'값을 사용한다면 단순히 '7' 즉 'SAT'로 해석됩니다. 그러나 다른 요일 뒤에 'L' 문자를 붙인다면, "해당 월의 마지막 X요일"이 됩니다. 예를 들어 '6L', 'FRIL'와 같이 쓰면 "해당 월의 마지막 금요일"로 인식됩니다. 또한 "일"에서 마지막 날로부터의 offset을 표현하기도 합니다. 'L-3'은 지정월의 말일로부터 3일전을 의미하게 됩니다. 주의하실 점은 'L' 문자를 사용할 때, 값을 범위로 지정하지 말라는 것입니다. 이렇게 되면 예상치 못한 결과를 얻을 수 있습니다.

'W'는 주어진 날로부터 가장 가까운 평일을 의미합니다. 예를 들어 '15W'는 "15일에서 가장 가까운 평일"이 됩니다.

'#'은 "해당 월의 n 번째 XXX 평일" 입니다. 예를 들어 '6#3' 또는 'FRI#3'은 그 달의 세 번째 금요일입니다.

아래는 **Cron 표현식**의 예제들입니다. 보다 더 많은 예제는 `org.quartz.CronExpression`의 JavaDoc에서 확인하실 수 있습니다.

### Cron 표현식 예제

예제1: 정각부터 5분 간격으로 시작하는 `trigger`에 대한 표현식

"0 0/5 * * * ?"

예제2: 정각 10초부터 5분 간격으로 시작하는 `trigger`에 대한 표현식

"10 0/5 * * * ?"

예제3: 매주 수요일과 금요일 오전 10시 30분, 11시 30분, 12시 30분, 13시 30분에 시작하는 `trigger`에 대한 표현식

"0 30 10-13 ? * WED,FRI"

예제4: 매달 5일, 20일 오전 8시, 10시 사이 30분 간격으로 시작하는 `trigger`에 대한 표현식

"0 0/30 8-9 5,20 * ?"

너무 복잡한 스케줄을 오직 하나의 `trigger`로만 표현하기 어려울 수 있습니다. 이 경우 두 개 이상의 `trigger`를 생성하고 같은 `job`을 등록하는 방식으로 해결할 수 있습니다.

## CronTrigger 생성

**CronTrigger** 객체는 `TriggerBuilder` 클래스나 `CronScheduleBuilder` 클래스를 통해 생성할 수 있습니다. 이 builder들을 DSL 스타일로 사용하고 싶다면 static import 문을 사용하면 됩니다.

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.DateBuilder.*;
```

### 매일 오전 8시, 오후 5시 사이 2분 간격으로 시작하는 `trigger` 생성

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
	.forJob("myJob", "group1")
	.build();
```

### 매일 오전 10시 42분 시작하는 `trigger` 생성

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.withSchedule(dailyAtHourAndMinute(10, 42))
	.forJob(myJobKey)
	.build();
```

또는 

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.wtihSchedule(cronSchedule("0 42 10 * * ?"))
	.forJob(myJobKey)
	.build();
```

### 시스템 시간 기준으로 매주 수요일 오전 10시 42분에 시작하는 `trigger` 생성

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
	.forJob(myJobKey)
	.inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
	.bulid();
```

또는

```java
trigger = newTrigger()
	.withIdnetity("trigger3", "group1")
	.withSchedule(cronSchedule("0 42 10 ? * WED"))
	.inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
	.forJob(myJobKey)
	.bulid();
```

## CronTrigger Misfire Instruction

다음 instruction들은 **CronTrigger**에 misfire가 발생할 경우에 수행할 동작에 대해 기술한 것들입니다. 이 instuction들은 **CronTrigger** 클래스 내부에 상수로 정의되어 있습니다.

```java
MISFIRE_INSTRUCTION_IGNORE_MISFILE_POLICY
MISFIRE_INSTRUCTION_DO_NOTHING
MISFIRE_INSTRUCTION_FIRE_NOE
```

**SimpleTrigger**처럼 모든 `trigger`들은 `Trigger.MISFIRE_INSTRUCTION_SMART_POLICY` instruction을 기본 값으로 사용합니다. **CronTrigger**에서는 이 'smart policy'는 `MISFIRE_INSTRUCTION_FIRE_NOW`로 인식됩니다. `CronTrigger.updateAfterMisfile()`메서드의 JavaDoc 문서를 참고하시면 기본 동작에 대해 보다 상세한 설명을 확인하실 수 있습니다.

**CronTrigger**를 생성할 때, 다음과 같이 misfire instruction을 지정할 수 있습니다.

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
		.withMisfireHandlingInsturctionFireAndProceed())
	.forJob("myJob", "group1")
	.build();
```