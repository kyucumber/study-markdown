## 매일프로그래밍 - 가장 긴 SubString의 길이


매일프로그래밍 - Question 10 

String이 주어지면, 중복된 char가 없는 가장 긴 서브스트링 (substring)의 길이를 찾으시오.  

예제)

> Input: “aabcbcbc”
>
> Output: 3 // “abc”

>  Input: “aaaaaaaa”
>
> Output: 1 // “a”

> Input: “abbbcedd”
>
> Output: 4 // “bced”

요즘따라 왜 이렇게 잘 못하는 스트링 관련 문제가 많은지 모르겠다.

해커랭크의 두 문제부터, 매일 프로그래밍에서 준 문제도 죄다 스트링 관련 문제.. 셋 다 진짜 잘 모르겠다.

1시간 가량 고민하다가 그냥 답이 너무 대놓고 있어서 봤는데 진짜 다들 머리가 참 좋은것 같다.



답의 전부를 보지 않고, 힌트만 살짝 보고 다시 구현했는데 답이랑 조금 다른 코드가 나왔다.

```java
import java.util.*;


public class Algorithm {

    public static int solve(String str) {
        Map<Character, Integer> check = new HashMap<>();
        int start = 0;
        int result = 0;
        for(int i=0; i<str.length(); i++) {
            if(check.containsKey(str.charAt(i))) {
                start = i;
            }
            result = Math.max(result, (i - start) + 1);
            check.put(str.charAt(i), i);
        }
        return result;
    }
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int result = solve("abbbcedd");
        System.out.println(result);
        in.close();
    }
}


```

다른 부분은

```java
if(check.containsKey(str.charAt(i))) {
    start = i;
}

if (map.containsKey(s[j])) {
    start = Math.max(map.get(s[j]), start);
}
```

위에가 내 코드고, 아래가 내 코든데 밑에처럼 해 주는 이유가 이해가 안됐었는데

`aaaackkuabbababbaa`이런 경우에 내가 짠 코드에서는

kua부분에서 a에서 중복이 있기 때문에 start를 a위치로 변경해줄텐데, 지금의 시작점은 k이고 k가 중복된게 아니기 때문에 a로 시작점을 변경해주는게 아니라 k가 start로 지정되어야 제대로 된 substring을 구할 수 있다.

그래서 위의 해답의 코드에서는 중복이 발생된 이전의 map에 든 index값과 현재 시작점의 start 값을 비교해서 큰 값을 시작점으로 삼게 된다.



매일 프로그래밍 풀이 코드)

이 문제는 해쉬맵을 사용하여 char와 char의 인덱스를 저장하여 풀면 됩니다. string의 각 char를 보면서 해쉬맵에 있다면 substring 시작점을 char의 인덱스+1 로 두면 됩니다. 그리고 현재 char의 인덱스와 시작점의 거리를 계속 계산하여 가장 큰 값을 리턴하면 됩니다.

```java
int longestSubstringLength(String s) {
	int ret = 0;
	int start = 0;
	Map<char, int> map = new HashMap<>();
	for (int j = 0; j < s.length(); j++) {
		if (map.containsKey(s[j])) {
			start = Math.max(map.get(s[j]), start);
		}
		ret = Math.max(ret, j - start + 1);
		map.put(s[j], j + 1); // 캐릭터 인덱스 저장
	} 
	return ret;
}
```



## 최종 코드

```java
import java.util.*;

public class Algorithm {

    public static int solve(String s) {
        Map<Character, Integer> check = new HashMap<>();
        int ret = 0;
        int start = 0;
        Map<Character, Integer> map = new HashMap<>();
        for (int j = 0; j < s.length(); j++) {
            if (map.containsKey(s.charAt(j))) {
                start = Math.max(map.get(s.charAt(j)), start);
            }
            ret = Math.max(ret, (j - start) + 1);
            map.put(s.charAt(j), j + 1); // 캐릭터 인덱스 저장
        }
        return ret;
    }
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int result = solve("abbbcedd");
        System.out.println(result);
        in.close();
    }
}
```

