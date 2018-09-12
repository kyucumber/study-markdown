## 프로그래머스 최고의 집합 - Level 4


자연수 N개로 이루어진 집합 중에, 각 원소의 합이 S가 되는 수의 집합은 여러 가지가 존재합니다. 최고의 집합은, 위의 조건을 만족하는 집합 중 각 원소의 곱이 최대가 되는 집합을 의미합니다. 집합 원소의 개수 n과 원소들의 합 s가 주어지면, 최고의 집합을 찾아 원소를 오름차순으로 반환해주는 bestSet 함수를 만들어 보세요. 만약 조건을 만족하는 집합이 없을 때는 배열 맨 앞에 –1을 담아 반환하면 됩니다. 예를 들어 n=3, s=13이면 [4,4,5]가 반환됩니다.
(자바는 집합이 없는 경우 크기가 1인 배열에 -1을 담아 반환해주세요.)



처음에는 모든 경우의 수를 구해서 최대값을 비교해 담아준 다음 리턴해줘야 할거라고 생각했는데 그러면 너무 많은 경우의 수가 나온다. 그래서 곰곰히 생각해보니까 곱이 최대가 되려면 숫자가 비슷한 값을 가져야 한다고 생각했다. 13을 예로 들면 1, 5, 7보다는 4, 4, 5 처럼 셋 다 비슷한 값을 가지는 경우 곱이 가장 클거라고 생각했다.

그래서 먼저 주어진 n으로 s를 나눈 몫을 n의 크기를 가진 배열에 채우고.. 나머지를 그 배열에 뒤에서부터 골고루 1을 더하면 가장 균등하면서 s를 합으로 가지는 n개의 값이 나오지 않을까 하고..풀었는데 맞았다.

예외처리 때문에 중간에 틀려서 이렇게 풀면 안되나? 했는데 여튼 맞긴 맞았다. 더 좋은 방법이 없을지..

```java
import java.util.Arrays; //테스트로 출력해 보기 위한 코드입니다.

public class BestSet {

    public int[] bestSet(int n, int s){
        int[] answer = null;
        int duplicatedNum = s/n;
        int remain = s%n;
        int i = 0;
        answer = new int[n];
        for(; i<n; i++) { answer[i] = duplicatedNum; }
        i -= 1;
        while(remain > 0) {
            answer[i--] += 1;
            remain--;
            if(i == 0) { i = answer.length - 1; }
        }
        if(n > s) { return new int[]{-1}; }
        return answer;
    }
    public static void main(String[] args) {
        BestSet c = new BestSet();
        //아래는 테스트로 출력해 보기 위한 코드입니다.
        System.out.println(Arrays.toString(c.bestSet(3,13)));
    }

}
```