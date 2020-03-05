---
layout: post
title: Quartz misfireThreshold
tags: [Quartz, 쿼츠, misfireThreshold, TroubleShooting, TS]
---

## threshold
일단 위 단어를 찾으면 제일 먼저 나오는 뜻은 '문지방'이지만 프로그램에서 사용하는 의미는 임계치, 시작점 정도로 이해하면 될 듯 하다.

## 발단
SpringBoot + Quartz 로 배치관련 어플리케이션을 개발중에 스케쥴링 된 트리거를 정지(pause) 후 재개(resume)하는 기능을 개발하고 있었다. (Trigger는 Job이 실행될 시점이라고 이해하자.)  
등록된 Trigger는 Quartz의 Scheduler 인터페이스 구현체에 의해 제어될수 있는데 다음과 같은 호출로 정지/재개한다.

서비스 클래스 코드의 일부이다. 생성된 스케쥴러 빈을 주입받아 TriggerKey를 인자로 등록된 Trigger를 제어한다.
```java
@Autowired
Scheduler scheduler;

public void pause(TriggerKey triggerkey) throws SchedulerException {
    scheduler.pauseTrigger(triggerkey);
}

public void resume(TriggerKey triggerkey) throws SchedulerException {
    scheduler.resumeTrigger(triggerkey);
}
```

문제는 pause 후 몇 초가 지나고 resume을 해보면 이미 Trigger에 세팅된 시작시점이 지난 Trigger가 resume과 함께 시작해버린다. (실행 되어야 하나 실행하지 못하는 것은 misfire라 하며, Trigger에 misfire 정책을 MISFIRE_INSTRUCTION_DO_NOTHING 으로 했음에도 발생)  

예를들어 매분 10초마다 실행되는 Trigger-Job이 있다고 하면,
- 0초에 해당 배치를 paues → 10초에 pause 상태이므로 실행 안됨 → 20초에 resume → 다음 분 10초에 실행

이런 동작을 기대했지만 실제로는 아래와 같이 되어버린다.
- 0초에 해당 배치를 paues → 10초에 pause 상태이므로 실행 안됨 → 20초에 resume과 함께 10초에 실행되어야 할 Trigger-Job이 실행 → 다음 분 10초에 실행

## 해결
Quartz에서는 `misfireThreshold` 값이 존재하고 디폴트가 60초로 세팅된다.  
Trigger가 재개될 때는 두 가지 스탭으로 실행여부를 결정하는 것 같다.
1. nextFireTime(다음 실행시간)이 (현재시간 - `misfireThreshold`) 보다 크면 실행한다.
2. Trigger에 설정된 misfire 정책에 따라 실행한다.(MISFIRE_INSTRUCTION_DO_NOTHING이면 실행 안함)

즉, 나는 `misfireThreshold`가 60초 디폴트 설정 된 상태에서 60초가 지나기 전에 pause를 풀었으므로 misfire정책에 상관없이 즉시 실행이 되었던 것이다.

그래서 해결법은  `misfireThreshold` 를 짧게 설정 하던지, pause 후에 60초가 지나서 resume을 하면 된다.  
`misfireThreshold`는 properties 설정으로 다음과 같이 설정할 수 있다.
```yaml
quartz:
    properties:
      org.quartz.jobStore.misfireThreshold: 10000 # misfire라고 판단하는 기준시간
```

## 참고
> The problem is, the trigger doesn't know it is being paused. To overcome this there is this misfire handling. After resuming the jobs the trigger's updateAfterMisfire() method will be invoked which corrects the nextFireTime. But not if the difference between nextFireTime and now is smaller than the misfireThreshold. Then the method is never called. This threshold's default value is 60,000. Thus if your pause period would be longer than 60s everything would be fine.

[https://stackoverflow.com/questions/1933676/quartz-java-resuming-a-job-executes-it-many-times](https://stackoverflow.com/questions/1933676/quartz-java-resuming-a-job-executes-it-many-times)