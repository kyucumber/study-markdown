# @MockBean, @SpyBean 이용 Caching 사용 시 단위 테스트 작성하기

Service단 코드에 EhCache를 적용했었다. 그때 그와 관련된 테스트를 어떻게 진행해야 할 지 고민했었는데 @MockBean과 @SpyBean 어노테이션을 이용해 Repository 코드가 몇번 호출되는지 체크해 단위 테스트를 진행할 수 있었음.

> Spring Boot 1.4에서 @MockBean과 @SpyBean이 추가됨.
>
> @MockBean은 껍데기만 그 객체의 형태를 유지하고, 내부 구현은 사용자가 when, given이나 기타 등등을 통해서 사용자가 직접 정의해주어야 함.
>
> @SpyBean을 사용하면 given, when을 한 것 외에 실제 객체를 사용한다. 여기서는 verify를 사용하기 위해 @SpyBean을 사용함.

아래의 코드의 bestCategoryService의 findAll에 캐싱이 적용되어 있다.

findAll 메소드를 3번 호출했을 때 1번만 호출되는지에 대한 검증을 통해 캐싱이 제대로 되는지에 대한 테스트를 진행했다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
public class BestCategoryServiceTest {
    @MockBean
    private BestCategoryRepository bestCategoryRepository;
    @SpyBean
    private BestCategoryService bestCategoryService;

    @Test
    public void findAllTestCaching() {
        BestCategory bestCategory1 = BestCategoryDto.defaultBestCategoryDto().setCategoryName("첫번째 카테고리").toEntity();
        when(bestCategoryRepository.findAll()).thenReturn(Arrays.asList(bestCategory1));
        bestCategoryService.findAll();
        bestCategoryService.findAll();
        bestCategoryService.findAll();
        verify(bestCategoryService, atMost(1)).findAll();
    }
}
```

