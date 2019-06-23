# Lesson 4: Trigger에 대한 추가 정보

`job`과 같이 `trigger`은 다루기 매우 쉽습니다. 그러나 `Quartz`의 모든 기능을 활용하기 위해서는 다양한 설정을 알아야 합니다. 또한 앞서 설명한 것처럼, 다양한 스케줄링 상황을 지원하기 위해 여러  `trigger`를 제공합니다. 

다음 장인 Lesson 5: Simple Triggers, Lession 6: Cron Triggers 에서 대표적인 두 `trigger`에 대해 알아볼 것입니다.

## 공통 Trigger 속성

모든 `trigger`에는 서로 다른 `trigger`끼리 구별을 위한 `TriggerKey`라는 속성을 가지고 있습니다. 이와 별개로 공통 속성이 몇 가지 더 있는데, 이러한 속성들은 `trigger`를 정의하면서  `TriggerBuilder`를 통해 설정할 수 있습니다.

다음은 `trigger`가 가지는 공통 속성들입니다.

- `jobKey`: `trigger`가 시작했을 때, 실행되어야 하는 `job`을 가리킵니다.
- `startTime`: `trigger`가 처음으로 시작되는 시간을 가리킵니다. 이 값은 특정 시간을 정의하는 `java.util.Date` 객체 입니다. 특정 `trigger` 타입은 실제로 `startTime`에 정의된 시간에 시작하기도 하지만, 다른 타입은 이 시간을 schedule이 따라야 하는 시간을 표시하는데 사용합니다. 만약 "매월 5일"에 시작하는 `trigger`의 `startTime`이 4월 1일로 되어 있고 이 `trigger`를 1월에 생성한다면, 처음 시작하는데 몇 개월이 걸릴 것입니다.
- `endTime`: `trigger`가 더이상 시작하지 않는 시간을 가리킵니다. 만약 "매월 5일"에 시작하는 `trigger`의 `endTime`이 7월 1일이라면, 이 `trigger`가 시작하는 마지막 시간은 6월  5일이 될 것입니다.

다른 속성들은 아래에서 좀더 자세히 설명하겠습니다.

## 우선 순위

Quartz thread pool에 있는 가용 worker thread보다 더 많은 `trigger`가 존재하는 경우, Quartz는 같은 시각에 시작하기로 되어 있는 `trigger`들을 즉시 실행할 수 없을 수 있습니다. 이 경우, `trigger`의 `priority`속성을 설정함으로써, 가장 먼저 worker thread를 차지할 `trigger`를  정할 수 있습니다.  만약, N 개의 `trigger`가 동시에 시작하고 가용한 worker thread가 Z개 있는 경우, 높은 우선 순위를 가진 Z개의 `trigger`가 먼저 시작합니다. 아무런 `priority`를 설정하지 않는 경우 기본 값으로 5를 가집니다. `priority`값으로 음수 또는 양수를 지정할 수 있습니다.

> 알아 두기: `Priority`는 오직 같은 시간에 시작되는 `trigger`들끼리 우선 순위를 정할 때 사용합니다. 기본적으로 10:59분에 시작하기로 되어 있는 `trigger`는 항상 11:00분에 시작하기로 된 `trigger`보다 먼저 시작합니다.

> 알아 두기: `trigger`의 `job`이 recovery 과정을 필요로 하는 경우, 이 recovery 과정은 원래 `trigger`와 같은 `priority`를 갖습니다.

## Misfire Instructions

`Trigger`의 또다른 중요 속성은 "misfire instruction" 입니다. "misfire"란 scheduler가 셧다운 상태이거나 Quartz thread pool에 가용한 worker thread가 없는 경우 `trigger`가 시작하지 못하는 것을 말합니다. 서로다른 `trigger` 타입은 각각에 맞는 "misfire instruction"을 가집니다. 기본적으로 'smart policy' 규칙을 따르게 되는데, 이는 `trigger` 타입과 설정에 따라 다양한 행동을 하도록 되어 있습니다. Scheduler가 시작하면 "misfire"된 `trigger`가 있는지 찾습니다. 그리고`trigger` 타입에 정의된 "misfire instruction"을 따라 각 `trigger`들을 업데이트하게 됩니다. Quartz를 여러분들의 프로젝트에 사용하기로 마음먹은 경우, 가능한 주어진 `trigger` 타입에 정의된 "misfire instruction"에 익숙하도록 하십시오. "misfire instruction"에 대한 더 많고 자세한 설명은 각 `trigger` 타입에 대한 tutorial에서 하도록 하겠습니다.

## Calendars

Quartz `Calendar` 객체(Java의 `java.util.Calendar` 객체와는 다릅니다)는 `trigger`가 정의되어 scheduler에 저장된 시간에 `trigger`에 연결됩니다. 보통 `Calendar`는 `trigger`가 시작하는 스케줄에서 특정 시간 구간을 제외할 때 사용합니다. 예를 들어, 매주 평일 오전 9:30분에 시작하는 `trigger`가 있다고 합시다. 이 `trigger`에는 모든 공휴일을 제외하는 `Calendar`를 붙여두면 좋을 것입니다.

`Calendar` 객체는 Quartz에서 제공하는 interface를 구현하고 serializable한 객체이기만 하면 어떤 것도 가능합니다. `Calendar` interface는 아래와 같습니다.

```java
package org.quartz;

public interface Calendar {
	public boolean isTimeIncluded(long timestamp);
	public long getNextIncludedTime(long timestamp);
}
```

위에서 보듯이 모든 메서드의 파라미터들은 long 타입으로 정의되어 있습니다. 아시다시피 이 파라미터들은 timestamp를  millisecond 포맷으로 나타낸 것입니다. 이 말인 즉슨 `Calendar`를 사용하면 millisecond 단위로 특정 시간을 스케줄에서 제외할 수 있는 것입니다. 앞 선 예제처럼 대부분의 경우 특정일 전체를 제외하고 싶다면 Quartz에서 제공하는 `HolidayCalendar`를 사용해도 됩니다.

`Calendar`를 사용하기 위해서는 반드시 scheduler에 `addCalendar()` 메서드를 통해 등록해야 합니다. `HolidayCalendar`를 사용하려면, 객체를 생성하고 `addExcludedDate(Date date)` 메서드를 통해 스케줄링 할 때 제외할 날짜들을 추가합니다. 또한 다음과 같이 여러 `trigger`들에 대해 같은 `Calendar` 객체를 사용할 수 있습니다.

```java
HolidayCalendar cal = new HolidayCalendar();
cal.addExcludedDate(someDate);
cal.addExcludedDate(someOtherDate);

sched.addCalendar("myHolidays", cal, false);

Trigger t = newTrigger()
	.withIdentity("myTrigger")
	.forJob("myJob")
	.withSchedule(dailyAtHourAndMinute(9, 30))
	.modifiedByCalendar("myHolidays")
	.build();
	
Trigger t2 = newTrigger()
	.withIdentity("myTrigger2")
	.forJob("myJob2")
	.withSchedule(dailyAtHourAndMinute(11, 30))
	.modifiedByCalendar("myHolidays")
	.build();
```

`trigger`의 생성/구성 과정에 대해서는 다음 두 Lession에서 보다 자세히 다루도록 하겠습니다. 지금은 위 예제에서 단순히 `Calendar`에서 지정한 날짜를 제외하고 매일 시작하는 두 개의 `trigger`를 생성한다는 정도만 알아두시면 됩니다. 

더 많은 `Calendar`의 구현에 대해서는 `org.quartz.impl.calendar` 패키지를 참조하시기 바랍니다.

