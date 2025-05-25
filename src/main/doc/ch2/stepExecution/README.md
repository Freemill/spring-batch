![img.png](img.png)
![img_1.png](img_1.png)
![img_2.png](img_2.png)
![img_3.png](img_3.png)
![img_4.png](img_4.png)
```java
@Configuration
@RequiredArgsConstructor
public class StepExecutionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step2 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }

    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }
}
```
실행을 시켜보자 <br>
![img_5.png](img_5.png)
잘 동작을 했다. 
![img_6.png](img_6.png)
![img_7.png](img_7.png)
STATUS는 COMPLETED이다. <br>
![img_8.png](img_8.png)
EXIT_CODE도 COMPLETED이다. <br>
데이터를 삭제하고 이번엔 step2에 예외를 발생시켜보자. <br>
![img_9.png](img_9.png)

```java
@Configuration
@RequiredArgsConstructor
public class StepExecutionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step2 has executed");
                    throw new RuntimeException("step2 has failed");
                })
                .build();

    }

    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }
}
```
![img_10.png](img_10.png)
2번째 step에서 예외가 발생했다. <br>
![img_11.png](img_11.png)
JOB_INSTANCE는 동일하게 하나 생성
![img_12.png](img_12.png)
JOB_EXECUTION의 STATUS와 EXIT_CODE는 FAILED이다. <br>
3개의 step중 한개가 failed했기 때문에 <br>
![img_13.png](img_13.png)  
JOB_EXECUTION의 상태는 FAILED이다. <br>
![img_14.png](img_14.png)
STEP_EXECUTION에서도 step2가 생성은 되었다. 
![img_15.png](img_15.png)
이 때 step3는 실행되지도 않았다. 이 상태에서 다시 실행을 시키면 실패기 때문에 job이 재시작이 가능한다. <br>
```java
@Configuration
@RequiredArgsConstructor
public class StepExecutionConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job BatchJob() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step1 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step2 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }

    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println(">> step3 has executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }
}

```
![img_16.png](img_16.png)
예외가 발생하지 않고 모든 스탭이 성공했다. <br>
![img_17.png](img_17.png)
![img_18.png](img_18.png)
2번째에서는 step2, step3가 실행되고, 상태가 COMPLETED로 변경되었다. <br> 
![img_19.png](img_19.png)


