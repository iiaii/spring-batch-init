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


---
### Spring Batch 프로젝트

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


##### Job Parameter와 Scope

> `@StepScope`, `@JobScope` 와 Job Parameter 에 대한 내용

Spring Batch의 경우 외부, 내부에서 파라미터를 받아 여러 Batch 컴포넌트에서 사용할 수 있게 지원된다.
이 파라미터를 Job Parameter라고 한다. Job Parameter를 사용하기 위해선 항상 Spring Batch 전용 Scope를 선언해야만 한다.
크게 2가지 스코프가 있다.

- `@StepScope`
- `@JobScope`

```java
// 사용법 (SpEL)
@Value("#{jobParameters[파라미터명]}")
```

> jobParmeters 외에도 jobExecutionContext, stepExecutionContext 도 SpEL로 사용가능하다. @JobScope에선 stepExecutionContext는 사용할 수 없고, jobParameters와 jobExecutionContext만 사용할 수 있다.

- JobScope

```java
@Bean
public Job simpleJob() {
    return jobBuilderFactory.get("simpleJob")
            .start(simpleStep1(null))
            .next(simpleStep2(null))
            .build();
}

@Bean
@JobScope
public Step simpleStep1(@Value("#{jobParameters[requestDate]}")String requestDate) {
    return stepBuilderFactory.get("simpleStep1")
            .tasklet(((contribution, chunkContext) -> {
                log.info(">>>>>> This is Step1");
                log.info(">>>>>> requestDate = {}", requestDate);
                return RepeatStatus.FINISHED;
            }))
            .build();
}
```

- StepScope

```java
@Bean
public Step scopeStep2() {
    return stepBuilderFactory.get("scopeStep2")
            .tasklet(scopeStep2Tasklet(null))
            .build();
}

@Bean
@StepScope
public Tasklet scopeStep2Tasklet(@Value("#{jobParameters[requestDate]}") String requestDate) {
    return (contribution, chunkContext) -> {
        log.info(">>>>> This is scopeStep2");
        log.info(">>>>>> requestDate = {}", requestDate);
        return RepeatStatus.FINISHED;
    }
}
```

예제 코드에서 호출하는 쪽에서 null을 할당하는데 Job Parameter의 할당이 어플리케이션 실행시에 하지 않기 때문에 가능하다.

##### `@StepScope`와 `@JobScope`

Spring Bean의 기본 Scope는 singleton이다.
하지만 `@StepScope`를 사용하게 되면 Spring Batch가 Spring 컨테이너를 통해 지정된 Step의 실행지점에 해당 컴포넌트를 Spring Bean으로 생성한다.
(`@JobScope`은 Job 실행지점에 Bean이 생성된다) 즉, Bean의 생성 시점을 지정된 Scope가 실행되는 시점으로 지연시킨다.

> MVC의 request scope와 비슷하다. request scope가 request가 왔을 때 생성되고, response를 반환하면 삭제되는 것 처럼, JobScope, StepScope 역시 Job이 실행되고 끝날때, Step이 실행되고 끝날때 생성/삭제가 이루어진다.


Bean의 생성시점을 Step, Job의 실행시점으로 지연시키면 2가지 장점이 있다.

- JobParameter의 Late Binding이 가능하다

Job Parameter가 StepContext 또는 JobExecutionContext 레벨에서 할당시킬 수 있다. 꼭 Application이 실행되는 시점이 아니더라도 Controller나 Service와 같은 비즈니스 로직 처리 단계에서 Job Parameter를 할당시킬 수 있다.

- 동일한 컴포넌트를 병렬 혹은 동시에 사용할 때 유용하다

Step안에 Tasklet이 있고, Tasklet은 멤버 변수와 이 멤버 변수를 변경하는 로직이 있다고 할때, 
`@StepScope` 없이 Step을 병렬로 실행시키게 되면 서로 다른 Step에서 하나의 Tasklet을 두고 상태를 변경하려 할것이다.
하지만 `@StepScope`가 있으면 Step에서 별도의 Tasklet을 생성하고 관리하기 때문에 서로의 상태를 침범하지 않는다.


##### Job Parameter 생성 

Job Parameters는 `@Value`를 통해서 가능하다. Job Parameters는 Step이나, Tasklet, Reader 등 Batch 컴포넌트 Bean의 생성 시점에 호출할 수 있다. (정확히는 Scope Bean을 생성할때만 가능)\
즉, `@StepScope`, `@JobScope` Bean을 생성할때만 Job Parameters가 생성되기 때문에 사용할 수 있다.

```java
// @Bean과 @Value("#{jobParameters[파라미터]}") 제
@Bean
public Job simpleJob() {
    log.info(">>>>> definition simpleJob");
    return jobBuilderFactory.get("simpleJob")
            .start(simpleStep1())
            .next(simpleStep2(null))
            .build();
}

private final SimpleJobTasklet tasklet1;    // 생성자 DI

public Step simpleStep1() {
    log.info(">>>>> definition simpleStep1");
    return stepBuilderFactory.get("simpleStep1")
            .tasklet(tasklet1)
            .build();
}
```

```java
// SimpleJobTasklet 생성 
@Slf5j
@Component
@StepScope  // StepScope Bean
public class SimpleJobTasklet implements Tasklet {
    @Value("#{jobParameters[requestDate]}")
    private String requestDate;

    public SimpleJobTasklet() { log.info(">>>>> tasklet 생성"); }

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        log.info(">>>>> This is Step1");
        log.info(">>>>> requestDate = {}", requestDate);
        return RepeatStatus.FINISHED;
    }
}
```


##### Job Parameter vs. 시스템 변수

- Job Parameter

```java
@Bean
@StepScope
public FlatFileItemReader<Partner> reader(@Value("#{jobParameters[pathToFile]}") String pathToFile) {
    FlatFileItemReader<Partner> itemReader = new FlatFileItemReader<Partner>();
    itemReader.setLineMapper(lineMapper());
    itemReader.setResource(new ClassPathResource(pathToFile));
    return itemReader;
}
```

- 시스템 변수 (application.properties와 -D 옵션으로 실행하는 변수)

```java
@Bean
@ConfigurationProperties(prefix = "my.prefix")
protected class JobProperties {

    String pathToFile;

    ...getters/setters
}

@Autowired
private JobProperties jobProperties;

@Bean
public FlatFileItemReader<Partner> reader() {
    FlatFileItemReader<Partner> itemReader = new FlatFileItemReader<Partner>();
    itemReader.setLineMapper(lineMapper());
    String pathToFile = jobProperties.getPathToFile();
    itemReader.setResource(new ClassPathResource(pathToFile));
    return itemReader;
}
```

시스템 변수를 사용하는 경우 Spring Batch의 Job Parameter 관련 기능을 사용할 수 없다. (예를 들어 Job Parameter가 같은 Job을 두번 실행하지 않는 것)
또한 Spring Batch에서 자동으로 관리해주는 Parameter 관련 테이블이 전혀 관리되지 않는다.

그리고 Command line 외의 방법으로 Job을 실행하기 어렵다. 전역상태(시스템 변수, 환경변수)를 동적으로 변경시킬 수 있도록 Spring Batch를 구성해야 한다.

> Job Parameter를 못쓰면 Late Binding을 못하게 된다.


```java
// Job Parameter 활용
@Slf4j
@RequiredArgsConstructor
@RestController
public class JobLauncherController {

    private final JobLauncher jobLauncher;
    private final Job job;

    @GetMapping("/launchjob")
    public String handle(@RequestParam("fileName") String fileName) throws Exception {

        try {
            // Request Parameter로 받은 값을 Job Parameter로 생성
            JobParameters jobParameters = new JobParametersBuilder()
                                    .addString("input.file.name", fileName)
                                    .addLong("time", System.currentTimeMillis())
                                    .toJobParameters();
            // Job PArameter로 Job을 수행
            jobLauncher.run(job, jobParameters);
        } catch (Exception e) {
            log.info(e.getMessage());
        }

        return "Done";
    }
}
```

> 개발자가 원하는 어느 시점에든 Job Parameter를 생성하고 수행할 수 있다. Job Parameter를 각 Batch 컴포넌트들이 사용하면 되기 때문에 변경이 심한 경우에도 쉽게 대응할 수 있다.
> (위 예제는 예제일 뿐 웹서버에서 Batch를 관리하는 것은 권장되지 않는다)

`@Bean`과 `@StepScope`를 함쎄 쓰는 것은 `@Scope (value = "step", proxyMode = TARGET_CLASS)`와 같고, proxyMode로 인해서 문제가 발생할 수 있다.

[@StepScope 사용시 주의사항](http://jojoldu.tistory.com/132)


##### Chunk

Spring Batch에서 Chunk란 데이터 덩어리로 작업할 때 각 커밋 사이에 처리되는 row 수를 이야기한다.
즉, Chunk 지향 처리란 한번에 하나씩 데이터를 읽어 Chunk라는 덩어리를 만든 뒤, Chunk 단위로 트랜잭션을 다루는 것을 의미한다.

Chunk 단위로 트랜잭션을 수행하기 때문에 실패할 경우에 해당 Chunk 만큼 롤백이 되고, 이전에 커밋된 트랜잭션 범위까지는 반영이 된다.

![spring batch chunk](https://t1.daumcdn.net/cfile/tistory/999A513E5B814C4A12)


- Reader에서 데이터를 하나 읽어온다
- 읽어온 데이터를 Processor에서 가공한다
- 가공된 데이터들을 별도의 공간에 모은 뒤, Chunk 단위만큼 쌓이게 되면 Writer에 전달하고 Writer는 일괄 저장한다

**Reader와 Processor에서는 1건씩 다뤄지고, Writer에선 Chunk 단위로 처리된다**

```java
// Chunk 지향 처리
for(int i=0; i<totalSize; i+=chunkSize){ // chunkSize 단위로 묶어서 처리
    List items = new Arraylist();
    for(int j = 0; j < chunkSize; j++){
        Object item = itemReader.read()
        Object processedItem = itemProcessor.process(item);
        items.add(processedItem);
    }
    itemWriter.write(items);
}
```


##### ChunkOrientedTasklet

Chunk 지향 처리의 전체 로직을 다루는 것은 `ChunkOrientedTasklet` 클래스이다.

해당 클래스의 `execute()`를 보면 Chunk 단위로 작업하기 위한 전체 코드가 있다. 

- `chunkProvider.provide()` 로 Reader에서 Chunk size 만큼 데이터를 가져온다(read)
- `chunkProcessor.process()` 에서 Reader로 받은 데이터를 가공(process)하고 저장(write)한다

![chunkProvider.provide()](https://t1.daumcdn.net/cfile/tistory/9969C3335B814C4D2A) 

`chunkProvider.provide()`는 inputs이 ChunkSize 만큼 쌓일때까지 read()를 호출한다.
read() 내부에서는 doRead()라는 메서드를 호출하는데, ItemReader.read에서 1건씩 데이터를 조회해 Chunk Size 만큼 데이터를 쌓는 것이 `provide()`가 하는 일이다.


##### SimpleChunkProcessor

Processor와 Writer 로직을 담고 있는 것은 `ChunkProcessor`가 담당한다. 인터페이스이기 때문에 기본적으로 사용되는 구현체가 `SimpleChunkProcessor`이다.

- `process()`

![process](https://t1.daumcdn.net/cfile/tistory/99DE13375B814C4C36)

transform() 은 반복문을 통해 doProcess() 를 호출한다. 가공된 데이터들은 doWriter()를 통해 일괄처리된다.


##### Page Size vs Chunk Size

- Chunk Size : 한번에 처리될 트랜잭션 단위
- Page Size : 한번에 조회할 Item의 양

즉 Page Size는 Page 단위로 끊어서 조회하는 것이고, 페이징 쿼리에서 Page Size를 지정하기 위한 값이다.

> PageSize가 10이고, ChunkSize가 50이라면 ItemReader에서 Page 조회가 5번 일어나면 1번의 트랜잭션이 발생하여 Chunk가 처리된다.

한번의 트랜잭션 처리를 위해 5번의 쿼리 조회가 발생하기 때문에 성능상 이슈가 발생할 수 있다. 
그래서 큰 페이지 크기를 설정하고 페이지 크기와 일치하는 커밋 간격으로 사용할 것을 권장한다.
즉, 2개의 값을 일치시키는 것이 보편적으로 좋은 방법이다.


##### ItemReader

Spring Batch는 Chunk 지향처리를 하며 이를 Job과 Stepp으로 구성하였다. Step은 Tasklet 단위로 처리되고, 를
Tasklet 중에서 ChunkOrientedTasklet을 통해 Chunk를 처리하며 이를 구성하는 3요소로 ItemReader, ItemWriter, ItemProcessor가 있다.

> ItemReader & ItemProcessor & ItemWriter 묶음 역시 Tasklet이다. (묶음을 ChunkOrientedTasklet에서 관리함)

![Spring batch ItemReader](https://t1.daumcdn.net/cfile/tistory/992DD04D5CEB519E20)

Spring Batch의 Chunk Tasklet은 위와 같은 과정으로 진행된다.

- ItemReader : 데이터를 읽어들인다. DB의 데이터만을 이야기하는 것이 아니라 입력 데이터, 파일, DB 외에도 다른 소스에서 읽어온다.

Item Reader의 종류와 사용방법 모두 상이한데 [ItemReader](https://jojoldu.tistory.com/336?category=902551)를 참고하자.
(ItemReader는 Spring Batch를 구현하는데 있어 중요한 구현체이다. 어디서 데이터를 읽고 어떤 방식으로 읽느냐에 따라 성능이 달라진다)


##### ItemWriter

(Processor는 필수가 아닌 선택이다.) ItemWriter는 Spring Batch에서 사용하는 출력 기능이다. 

Chunk 프로세스를 복기하자면

- ItemReader를 통해 각 항목을 개별적으로 읽고 이를 처리하기 위해 ItemProcessor에 전달한다
- 이 프로세스는 청크의 Item 개수 만큼 처리될 때까지 계속된다
- 청크 단위만큼 처리가 완료되면 Writer에 전달되어 Writer에 명시되어 있는대로 일괄처리한다

> Reader와 Processor를 거쳐 처리된 Item을 Chunk 단위 만큼 쌓은 뒤 이를 Writer에 전달한다

ItemWriter의 종류와 사용방법 모두 상이한데 [ItemWriter](https://jojoldu.tistory.com/339?category=902551)를 참고하자.


##### ItemProcessor

ItemProcessor는 필수 구성요소가 아니며 Writer에서도 충분히 구현 가능하다. 하지만 ItemProcessor를 사용하면 비즈니스 코드가 섞이는 것을 방지해준다.

ItemProcessor는 Reader에서 넘겨준 데이터 개별건을 가공/처리 한다. 일반적으로 ItemProcessor를 사용하는 방법은 2가지이다.

- 변환 

Reader에서 읽은 데이터를 원하는 타입으로 변환

- 필터

Reader에서 넘겨준 데이터를 Writer로 넘겨줄 것인지 결정 (null을 반환하면 Writer에 전달되지 않는다) 

> 마치 Stream의 map,filter 같음


자세한 내용은 [ItemProcessor](https://jojoldu.tistory.com/347?category=902551) 참고.

---
[jojoldu | Spring Batch 가이드](https://jojoldu.tistory.com/324)
