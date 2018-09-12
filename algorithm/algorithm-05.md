## 프로그래머스 N개의 최소공배수 - Level 3

프로그래머스 [N개의 최소공배수](https://programmers.co.kr/learn/challenge_codes/28) 문제

> 두 수의 최소공배수(Least Common Multiple)란 입력된 두 수의 배수 중 공통이 되는 가장 작은 숫자를 의미합니다. 예를 들어 2와 7의 최소공배수는 14가 됩니다. 정의를 확장해서, n개의 수의 최소공배수는 n 개의 수들의 배수 중 공통이 되는 가장 작은 숫자가 됩니다. nlcm 함수를 통해 n개의 숫자가 입력되었을 때, 최소공배수를 반환해 주세요. 예를들어 [2,6,8,14] 가 입력된다면 168을 반환해 주면 됩니다. 

두 수의 최소공배수는 a * b / 최대공약수
배열을 순환하면서 배열의 두 요소의 최대 공약수를 구한 뒤 두 수의 최소공배수를 구하고, 점차적으로 연산하면 전체의 최소공배수가 나올 것 같다. 

[1, 2, 3, 4, 5]라 하면 1, 2의 최소공배수를 구하고 그 값을 a라 하면  a와 3 두수의 최소공배수.. 이런식으로 점차적으로 연산하는 식으로 구현했다.

```java
import java.util.*;
class NLCM {
	public long nlcm(int[] num) {
		return Arrays.stream(num)
            .mapToLong(i -> i)
            .reduce((a, b) -> a * b / gcb(a, b)).getAsLong();
	}
	public static void main(String[] args) {
		NLCM c = new NLCM();
	}
  public long gcb(long a, long b) {
      if(b == 0) {
          return a;
      }
      return gcb(b, a%b);
  }
}
```

알고리즘에서 stream은 안 쓰는게 좋을거 같지만 내가 생각하는 방식으로 구현하려면 reduce를 쓰면 바로 해결될 것 같아서 reduce를 사용했다.

