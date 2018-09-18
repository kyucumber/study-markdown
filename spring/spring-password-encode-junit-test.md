# @Spy를 이용해 BcryptPasswordEncoder 랜덤값 단위 테스트 작성하기

코드를 작성하면서 회원 정보의 비밀번호 컬럼을 BcryptPasswordEncoder로 암호화해 저장하도록 구현했는데 그 부분과 관련된 단위 테스트를 작성했을 때, 매번 랜덤한 값이 인코딩되어서 나와서 단위 테스트 작성을 어떻게 해야할지에 대해 고민..

Mockito기반 단위 테스트 환경에서 PasswordEncoder를 직접 구현해서 MockPasswordEncoder를 생성한 뒤 @Spy로 주입하고 테스트를 작성했다.

큰 의미를 가지는 테스트는 아닌데 앞으로 랜덤값을 가지는 경우 DI로 주입 될 객체를 바꿔치기 해서 테스트를 하면 될 것 같다.

```java
@RunWith(MockitoJUnitRunner.class)
public class MemberServiceTest {
    public static final Logger log =  LoggerFactory.getLogger(MemberServiceTest.class);
    @Mock
    private MemberRepository memberRepository;
    @Spy
    private PasswordEncoder mockPasswordEncoder = new MockPasswordEncoder();
    @InjectMocks
    private MemberService memberService;

    @Test
    public void createTest() {
        MemberDto memberDto = MemberDto.defaultMemberDto();
        memberService.save(memberDto);
        Member member = Member.fromDto(memberDto, mockPasswordEncoder);
        verify(memberRepository).save(member);
    }

    private class MockPasswordEncoder implements PasswordEncoder {
        @Override
        public String encode(CharSequence rawPassword) {
            return new StringBuilder(rawPassword).reverse().toString();
        }

        @Override
        public boolean matches(CharSequence rawPassword, String encodedPassword) {
            return encode(rawPassword).equals(encodedPassword);
        }
    }
}
```