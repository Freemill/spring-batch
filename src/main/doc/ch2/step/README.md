![img.png](img.png)
![img_1.png](img_1.png)
![img_2.png](img_2.png)
![img_3.png](img_3.png)
![img_4.png](img_4.png)
![img_5.png](img_5.png)

```java
@Component
@RequiredArgsConstructor
public class StepConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .build();
    }

    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Step 1 executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    public Step step2() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Step 2 executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```
이렇게 해도 되는데 아래처럼 해도 됨. <br>
```java
public class CustomTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {

        System.out.println("step2 was executed");

        return RepeatStatus.FINISHED;
    }
}
```
```java
@Component
@RequiredArgsConstructor
public class StepConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job batchJob() {
        return this.jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .build();
    }

    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new CustomTasklet())
                .build();
    }

    public Step step2() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Step 2 executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```
CustomTasklet을 @Component로 등록해서 사용해도 됨<br>
디버깅을 해보자. <br>
![img_6.png](img_6.png)
![img_7.png](img_7.png)
![img_8.png](img_8.png)
![img_9.png](img_9.png)
![img_10.png](img_10.png)
빌드는 아래서 빌드한것.
![img_11.png](img_11.png)
![img_12.png](img_12.png)
getName할 때 
![img_13.png](img_13.png)
저기서 정한 "step1"을 사용한다. <br>
![img_14.png](img_14.png)
쭉 설정을 하면서 여기까지 온다. <br>
createTasklet() 결국은 tasklet을 저정하는 구문이다. <br>
![img_15.png](img_15.png)
![img_16.png](img_16.png)
구현체 중 TaskletBuilder를 사용해서 아래로 온다. <br>
![img_17.png](img_17.png)
그럼 이 tasklet은 customTasklet을 사용한다. <br>
그런데 아까 TaskletStepBuilder가 이 값을 저장해 놓았다. 그럼 여기서 저장한 값을 <br>
![img_18.png](img_18.png)
여기에 return해서 step에 저장한다. <br>
결론적으로 taskletStep은 api에서 설정한 tasklet이라는 클래스의 값을 가지고 있다!<br>
그 다음 step2도 마찬가지이다. 근데 이건 익명 클래스이기 때문에 <br>
![img_19.png](img_19.png)
lambda로 표시되 있는것을 확인할 수 있다. <br>
이렇게 해서 2개의 tasklet을 만든다. 그 뒤에 마지막으로 <br>
![img_22.png](img_22.png)
SimpleJob을 만든다. <br>
![img_21.png](img_21.png)
보면 <br>
![img_23.png](img_23.png)
SimpleJob을 만들고 거기에 job.setSteps(steps);를 통해서 <br>
![img_24.png](img_24.png)
2개의 step을 넣는다. <br>
이렇게 설정이 끝나고 <br> 
![img_25.png](img_25.png)
스프링 부트가 우리가 만든 job을 자동으로 실행시킨다. <br>
![img_26.png](img_26.png)
![img_27.png](img_27.png)
![img_28.png](img_28.png)
execute로 가보면 SimpleJob으로 간다. <br>
![img_29.png](img_29.png)
![img_30.png](img_30.png)
![img_31.png](img_31.png)
![img_32.png](img_32.png)
![img_33.png](img_33.png)
![img_34.png](img_34.png)
![img_35.png](img_35.png)
![img_36.png](img_36.png)
![img_37.png](img_37.png)
![img_38.png](img_38.png)
CustomTasklet이다. 
![img_39.png](img_39.png)
![img_41.png](img_41.png)
이렇게 끝나고 step2로 간다.








