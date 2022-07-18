# MockitoSpringJPA的findBy方法使用

```java
@RunWith(MockitoJUnitRunner.class)
@ExtendWith(MockitoExtension.class)
class TrainingPartServiceTest {

  @InjectMocks
  private TrainingPartService trainingPartService;

  @Mock
  private TrainingPartDao trainingPartDao;

  private TrainingPart samplePart = SampleDataCreator.createSamplePart();

  @Test
  void findById() {
    when(trainingPartDao.findById(anyInt()).get()).thenReturn(samplePart);
    Assert.assertNotNull(trainingPartService.findById(1));
  }

}
```

运行报错：

![image-20220520114247337](https://cdn.jsdelivr.net/gh/Brandoooon/myBlog/docs/Spring/img/image-20220520114247337.png)

因为mock的只是trainingPartDao，trainingPartDao的返回类型是Optional，正确的写法是：

```java
@RunWith(MockitoJUnitRunner.class)
@ExtendWith(MockitoExtension.class)
class TrainingPartServiceTest {

  @InjectMocks
  private TrainingPartService trainingPartService;

  @Mock
  private TrainingPartDao trainingPartDao;

  private TrainingPart samplePart = SampleDataCreator.createSamplePart();

  @Test
  void findById() {
    when(trainingPartDao.findById(anyInt())).thenReturn(Optional.of(samplePart));
    Assert.assertNotNull(trainingPartService.findById(1));
  }

}
```

