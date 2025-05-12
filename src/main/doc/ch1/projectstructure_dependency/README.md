![img.png](img.png)
![img_2.png](img_2.png)
![img_4.png](img_4.png)
@EnableBatchProcessing이 실제로 작동이 되면 -> SimpleBatchConfiguration이 초기화 하는 class가 호출이 되고 <br>
그 다음에 이 클래스 다음으로는 BasicBatchConfigurer, JpaBasicConfigurer의 설정 클래스가 다시 동작을 하고 마지막으로 <br>
BatchAutoConfiguration이 동작을 하게 된다. <br>

<br>

코드로 확인해보자. <br>
![img_5.png](img_5.png)
디버깅 모드로 실행을 하게 되면 가장 먼저 아래로 온다. <br>
![img_6.png](img_6.png)
SimpleBatchConfiguration 클래스를 쭉 보면 <br>
![img_7.png](img_7.png)
Spring Batch에 중요한 클래스들의 bean들을 생성하고 있다. <br>
그런데 빈들을 생성할 때 모두 createLazyProxy로 Proxy 객체로 생성을 하고 있다. <br>
그렇기 때문에 실제 생성되는 객체는 proxy 객체고 실제 객체는 아니다. <br>
![img_8.png](img_8.png)
프록시 객체 확인!
그리고 이 작업이 끝나게 되면 <br>
![img_9.png](img_9.png)
BatchConfigurerConfiguration 클래스가 작동을 하게 된다. <br>
여기에는 2가지가 있다. 하나는 BasicBatchConfigurer이고 또 하나는 JpaBatchConfigurer이다.  <br>
![img_10.png](img_10.png)
근데 JpaBatchConfigurer는 BasicBatchConfigurer를 상속받고 있다. 그래서 data들이 서로 공유가 되는것이다. <br>
그래서 이 클래스들이 생성이 되고 설정이진행이된다. 그리고 마지막으로 <br>
![img_11.png](img_11.png)
이 설정 클래스가 동작을 하게 된다. <br>
 





