# Spring Batch

[jojoldu | Spring Batch 가이드](https://jojoldu.tistory.com/324) 학습 내용


### 개요

> 배치(Batch)는 일괄처리 라는 뜻을 가지고 있다.

##### 배치 어플리케이션이 필요한 경우

- 매일 전날 데이터 집계를 해야 할 때
- 매일 집계하는 데이터가 너무 많아서 처리중 실패했을 때 실패 지점부터 다시 실행이 필요할 때
- 집계 데이터가 중복되지 않기를 바랄 때 (같은 파라미터로 같은 함수를 다시 실행하는 경우 실패하도록)

위와 같은 단발성 대용량 데이터 처리 어플리케이션을 배치 어플리케이션이라고 한다. 배치 어플리케이션을 구성하기 위해서는 비즈니스 로직 외에 부가적으로 신경써야할 부분들이 많다.
Spring Batch는 배치 어플리케이션을 보다 쉽고 안정적으로 구성할 수 있도록 제공한다.


##### 배치 어플리케이션

배치 어플리케이션은 다음 조건들을 만족해야 한다.

- 대용량 데이터 : 대량의 데이터를 전달, 계산, 가져오는 등의 처리를 할 수 있어야 한다
- 자동화 : 심각한 문제 해결을 제외하고 사용자 개입없이 실행되어야 한다
- 안정성 : 잘못된 데이터 충돌/중단 없이 처리할 수 있어야 한다
- 신뢰성 : 무엇이 잘못되었는지 추적 가능해야 한다 (로깅, 알림)
- 성능 : 지정한 시간안에 처리를 완료하거나 동시에 실행되는 다른 어플리케이션을 방해하지 않도록 수행되어야한다



---
### Spring Batch

##### Batch / Quartz

Spring Quartz 는 스케줄러 역할으로, Batch와 같은 대용량 데이터 배치 처리에 대한 기능을 지원하지 않는다. Batch 역시 Quartz의 다양한 스케줄 기능을 지원하지 않아서 보통 둘을 조합해서 사용한다. (정해진 스케줄마다 Quartz가 Spring Batch를 실행)


##### Batch 사례

- 일매출 집계

많은 거래가 이루어지는 커머스는 하루 거래가 50 ~ 100만 까지 발생한다. 이 경우 하루 데이터는 최소 100만 ~ 200만 row, 한달에 5000만 ~ 1억 정도가 될 수 있다.

실시간 집계 쿼리로 해결하면 조회 시간이나 서버 부하가 심하다. 그래서 매일 새벽에 전날 매출 집계 데이터를 만들어 외부 요청이 올 경우 미리 만들어준 집계 데이터를 바로 전달하면 성능과 부하 문제를 해결할 수 있다.

![spring Batch 일매출 집계](https://t1.daumcdn.net/cfile/tistory/9995544C5B606DFE0F)


- ERP 연동

대부분의 서비스에서 ERP를 사용한다.

> ERP는 자원 관리 시스템으로 사내 구성원, 매출, 지출 등을 모두 관리하는 소프트웨어 시스템을 뜻한다 (SAP)

재무팀의 요구사항으로 매일 매출 현황을 ERP로 전달해야하는 상황에서 Spring Batch가 많이 사용된다. 매일 아침 8시에 ERP에 전달해야할 매출 데이터를 전송해야한다면 아래와 같은 구조로 쉽게 구현할 수 있다.

![Spring Batch ERP](https://t1.daumcdn.net/cfile/tistory/991BB2365B606DFE1B)

> 이외에도 정해진 시간마다 데이터 가공이 필요한 경우 Spring Batch가 자주 사용된다


##### Spring Batch 프로젝트

![Spring Batch 구조](https://t1.daumcdn.net/cfile/tistory/99E8E3425B66BA2713)

Spring Batch에서 Job은 하나의 배치 작업 단위를 뜻한다.
Job 안에는 여러 Step이 존재하고 Step 안에 Tasklet 혹은 Reader & Processor & Writer 묶음이 존재한다. 


Tasklet 하나와 Reader & Processor & Writer 한 묶음이 같은 레벨이다. 그래서 Reader & Processor가 끝나고 Tasklet으로 마무리 짓는 것은 불가능하다.

> Tasklet은 개발자가 지정한 커스텀한 기능을 위한 단위로 보면 된다.



##### 메타 테이블

- BATCH_JOB_INSTANCE

Job Parameter에 따라 생성되는 테이블이다. 
Spring Batch가 실행될때 외부에서 받을 수 있는 파라미터인 Job Parameter를 통해 해당 날짜 데이터로 조회/가공/입력 등의 작업을 할 수 있다.

![job instance](https://t1.daumcdn.net/cfile/tistory/992174475B66D86811)

BATCH_JOB_INSTANCE 테이블에 동일한 Job이 Job Parameter가 달라지면 생성되고, 동일한 Job Parameter는 여러개 존재할 수 없다. (마치 set 같고 instance 처럼 생성됨)


- BATCH_JOB_EXECUTION

JOB_EXECUTION 과 JOB_INSTANCE 는 부모-자식 관계이다. JOB_INSTANCE가 성공/실패했던 모든 내역을 가지고 있다.

![job execution](https://t1.daumcdn.net/cfile/tistory/99F6D73C5B66D8681D)

또한 동일한 Job Parameter로 2번 실행했지만, 같은 파라미터 실행 오류가 발생하지 않았다. 즉, Spring Batch는 동일한 Job Parameter로 성공한 기록이 있을 경우에만 재수행이 안된다.


![job](https://t1.daumcdn.net/cfile/tistory/995743415B66D86825)

(job = spring batch job)


##### next

```java
@Bean
public Job stepNextJob() {
    return jobBuilderFactory.get("stepNextJob")
            .start(step1())
            .next(step2())
            .next(step3())
            .build();
}
```

`next()`는 순차적으로 Step들을 연결시킬때 사용된다. (순차적 실행)

application.yml 에 `spring.batch.job.names: ${job.name:NONE}`을 추가하고 아래와 같이 설정하면 지정한 이름의 배치만 실행된다.

![job.name](https://t1.daumcdn.net/cfile/tistory/993049425B6FC6BB20)


##### Job flow

Step은 앞에서 오류가 나면 뒤에 있는 Step들은 실행되지 못한다. 하지만 Step에 오류가 있을 때 분기되어 실행되야 하는 경우가 있다.
이러한 경우에 Spring Batch Job에서는 조건별로 Step을 사용할 수 있다.

```java
@Bean
public Job stepNextConditionalJob() {
    return jobBuilderFactory.get("stepNextConditionalJob")
            .start(conditionalJobStep1())
                .on("FAILED") // FAILED 일 경우
                .to(conditionalJobStep3()) // step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이 
                .end() // step3으로 이동하면 Flow가 종료한다.
            .from(conditionalJobStep1()) // step1로부터
                .on("*") // FAILED 외에 모든 경우
                .to(conditionalJobStep2()) // step2로 이동한다.
                .next(conditionalJobStep3()) // step2가 정상 종료되면 step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이 
                .end() // step3으로 이동하면 Flow가 종료한다.
            .end() // Job 종료
            .build();
}
```

- `on()`

캐치할 ExitStatus 지정 (`*`는 모든 ExitStatus 지정)

- `to()`

다음으로 이동할 Step 지정

- `from()`

일종의 이벤트 리스너로 상태값을 보고 일치하는 상태라면 `to()`에 포함된 step을 호출한다 (step1의 이벤트 캐치가 FAILED로 되어있는 상태에서 추가로 이벤트 캐치하려면 `from` 사용해야 함)

- `end()`

FlowBuilder를 반환하는 end와 FlowBuilder를 종료하는 end 2개가 있다. `on("*")` 뒤에 있는 end는 FlowBuilder를 반환하는 end, `build()` 앞에 있는 end는 FlowBuilder를 종료하는 end. 
FlowBuilder를 반환하는 end 사용시 계속해서 `from`을 이어갈 수 있다

`contribution.setExitStatus(ExitStatus.FAILED);`와 같이 분기 처리를 위한 상태값 조정을 통해 제어가능하다. (값을 세팅하면 해당 분기를 타고 실행)


##### Batch Status vs. Exit Status

BatchStatus는 Job 또는 Step의 실행 결과를 Spring에서 기록할 때 사용하는 Enum이다. (COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN)

```java
.on("FAILED").to(stepB())
```

> `on` 메소드가 참조하는 것은 BatchStatus가 아닌 Step의 ExitStatus이다
 
ExitStatus는 Step의 실행 후 상태를 뜻한다. (ExitStatus는 Enum이 아님)

따라서 위 구문은 exitCode가 FAILED로 끝나면 StepB로 가라는 의미이다.

기본적으로 ExitStatus의 exitCode는 Step의 BatchStatus와 동일하다. 커스텀한 코드도 설정 가능하다.

```java
// 커스텀 exitCode
public class SkipCheckingListener extends StepExecutionListenerSupport {

    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) && 
              stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else {
            return null;
        }
    }
}
```


##### Decide


Step flow 분기 처리에는 2가지 문제가 있다. 

- Step이 담당하는 역할이 2개 이상된다 (실제 해당 Step이 처리해야할 로직외에도 분기처리를 시키기 위해 ExitStatus 조작이 필요함)
- 다양한 분기 로직 처리가 어렵다 (ExitStatus를 커스텀하게 고치기 위해서 Listener를 생성하고 JobFlow에 등록해야 함)

그래서 Spring Batch 에는 Step들의 Flow 속에서 분기만 담당하는 `JobExecutionDecider` 타입이 있다.

```java
@Bean
public Job deciderJob() {
    return jobBuilderFactory.get("deciderJob")
            .start(startStep())
            .next(decider())        // 홀수 | 짝수 구분
            .from(decider())        // decider의 상태가
                .on("ODD")          // ODD라면
                .to(oddStep())      // oddStep로 간다
            .from(decider())        // decider의 상태가
                .on("EVEN")         // EVEN이라면
                .to(evenStep())     // evenStep으로 간다
            .end()                  // builder 종료
            .build();
}
```

분기 로직에 대한 모든 일을 OddDecider가 전담하고 있어서 Step과 명확히 역할, 책임이 분리된다.







---
[jojoldu | Spring Batch 가이드](https://jojoldu.tistory.com/324)
