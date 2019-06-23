# Lesson 5: SimpleTrigger

**SimpleTrigger**는 특정 시간에 정확히 한번 또는 일정 간격으로 특정 횟 수만큼 실행되는 `job`을 실행해야 하는 경우 사용합니다. 예를 들면 "2015년 1월 13일 오전 11시 23분 54초에 한번 시작"이나 "10초 간격으로 5회 시작" 같은 경우 사용할 수 있는 것입니다.

앞서 설명한 스케줄을 표현하기 위해서 **SimpleTrigger**는 start-time, end-time, repeat count, repeat interval 속성을 가집니다. 각 속성의 이름만 보면 어떤 것인지 짐작이 가시겠지만, end-time 속성에 대해서는 좀 더 자세히 설명해 보겠습니다.

Repeat count는 0이나 양수, 또는 `SimpleTrigger.REPEAT_INDEFINITELY`를 지정할 수 있습니다. Repeat interval은 반복할 시간을 millisecond 단위로 0이상의 수로 지정하면 됩니다. Repeat interval을 0으로 지정한다는 것은 repeat count 갯수만큼의 `trigger`가 시작된다는 것을 알아두세요.

End time 속성은 repeat count 속성보다 우선합니다. 이러한 특징을 사용하면 일정 기간 동안 매 10초마다 반복하는 스케줄을 작성하는 경우, start time과 end time 사이에서 시작 횟수를 계산할 필요가 없어집니다. 단순히 end time을 지정하고 repeat count를 `REPEAT_INDEFINITELY`로 두면 됩니다. (아니면 충분히 큰 수로 지정하여 end time이전에 스케줄이 종료되지 않게 하는 것도 가능합니다.)

**SimpleTrigger** 객체는 `TriggerBuilder` 클래스나 `SimpleScheduleBuilder` 클래스를 통해 생성할 수 있습니다. 이러한 builder 클래스들을 DSL 스타일로 사용하고 싶으시면 `static import` 구문을 사용하면 됩니다.

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.SimpleSchedulerBuilder.*;
import static org.quartz.DateBuilder.*;
```

아래 예제는 단순 스케줄을 가진 `trigger`를 정의하는 몇 가지 방법을 보여줍니다. 예제들을 참고하시고 차이점을 확인해보세요.

### 반복 없이 특정 시간에 시작하도록 `trigger`를 생성하는 경우

```java
SimpleTrigger trigger = (SimpleTrigger) newTrigger()
	.withIdentity("trigger1", "group")
	.startAt(myStartTime)
	.forJob("job1", "group1")	// 실행할 job을 name, group 명으로 지정
	.build();
```

### 특정 시간에 시작한 후 10초 간격으로 10번 반복하는 `trigger`를 생성하는 경우

```java
trigger = newTrigger()
	.withIdentity("trigger3", "group1")
	.startAt(myTimeToStartFiring)
	.withSchedule(simpleSchedule()
		.withIntervalInSeconds(10)
		.withRepeatCount(10))
	.forJob(myJob)	// 실행할 job을 jobDetail 객체를 그대로 넘겨서 지정
	.build();
```

### 5분 뒤 시작하는 `trigger`를 생성하는 경우

```java
trigger = (SimpleTrigger) newTrigger()
	.withIdentity("trigger5", "group1")
	.startAt(futureDate(5, IntervalUnit.MINUTE))
	.forJob(myJobkey)	// 실행할 job을 jobKey를 통해 지정
	.build();
```

### 지금 시작해서 22시 정각까지 5분 간격으로 시작하는 `trigger`를 생성하는 경우

```java
trigger = newTrigger()
	.withIdentity("trigger7", "group1")
	.withSchedule(simpleSchedule()
		.withIntervalInMinutes(5)
		.repeatForever())
	.endAt(dateOf(22, 0, 0))
	.build();
```

### 매 두시간 마다 반복해서 시작하는 `trigger`를 생성하는 경우

```java
trigger = newTrigger()
	.withIdentity("trigger8")
	.startAt(evenHourDate(null))
	.withSchedule(simpleSchedule()
		.withIntervalInHours(2)
		.repeatForever())
	.build();
	
scheduler.scheduleJob(trigger, job);	// trigger에 job을 지정하지 않고 스케줄러에 등록할때 함께 지정
```

`TriggerBuider` 클래스와 `SimpleScheduleBuilder` 클래스의 구현을 보고 사용가능한 모든 메서드를 확인하는 것도 좋습니다.

> 알아두기: `TriggerBuilder` 클래스 (와 Quartz의 다른 builder 클래스들)는 일반적으로 사용자가 지정하지 않은 속성에 대해서 나름 합리적인 값을 선택합니다. 예를 들어, `withIdentity()` 메서드를 호출하지 않으면, `TriggerBuilder`가 `trigger`에 대해 랜덤한 값을 이름으로 지정합니다. `startAt()` 메서드를 호출하지 않으면 현재 시간을 시작 시간으로 설정합니다.

## **SimpleTrigger**의 Misfire Instruction

**SimpleTrigger**는 misfire가 발생했을 때, 수행할 동작을 지시하는 몇가지 instruction들을 제공합니다. 이 instruction들은 `SimpleTrigger` 클래스에 상수로 정의되어 있습니다. Instruction들은 다음과 같습니다.

```JAVA
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFILE_INSTRUCTION_FIRE_NOW
MISFILE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT
MISFILE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT
MISFILE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT
MISFILE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISITING_COUNT
```

이전 lesson에서 모든 `trigger`는 기본적으로 `Trigger.MISFILE_INSTRUCTION_SMART_POLICY` instruction으로 지정된 것을 배우셨을 겁니다. 

만약 'smart policy'가 지정된 경우, **SimpleTrigger**는 현재 `trigger`의 설정과 상태를 기반으로 다양한 MISFIRE instruction 중 하나를 선택합니다.  `SimpleTrigger.updateAfterMisfile()`메서드에 대한 JavaDoc 문서를 읽어보시면 이 선택 동작에 대한 자세한 설명을 확인하실 수 있습니다.

**SimpleTrigger** 을 생성할 때, `SimpleSchedulerBuilder`를 통해 misfilre insturction을 지정할 수 있습니다.

```java
trigger = newTrigger()
	.withIdentity("trigger7", "group1")
	.withSchedule(simpleSchedule()
		.withIntervalInMinutes(5)
		.repeatForever()
		.withMisfireHandlingInstructionNextWithExistingCount())
	.build();
```

