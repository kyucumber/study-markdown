## Codility 5-1 CountDiv


#### [CountDiv](https://app.codility.com/programmers/lessons/5-prefix_sums/count_div/)

Write a function:

> `class Solution { public int solution(int A, int B, int K); }`

that, given three integers A, B and K, returns the number of integers within the range [A..B] that are divisible by K, i.e.:

> { i : A ≤ i ≤ B, i **mod** K = 0 }

For example, for A = 6, B = 11 and K = 2, your function should return 3, because there are three numbers divisible by 2 within the range [6..11], namely 6, 8 and 10.

Assume that:

> - A and B are integers within the range [0..2,000,000,000];
> - K is an integer within the range [1..2,000,000,000];
> - A ≤ B.

Complexity:

> - expected worst-case time complexity is O(1);
> - expected worst-case space complexity is O(1).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.



3개의 수 A, B, K가 주어지고, A와 B 사이에서 K로 나누어지는 A와 B 범위 내의 정수를 반환하는 문제인것 같다.

예제에서는 6, 11이 주어지고.. 2로 나누어지는 6, 8, 10 세개가 있으므로 3을 반환한다.

쉽네..하고 보니까 시간 복잡도가 1이다. 조금 고민이 필요할 것 같은데 일단은 그냥 당장 떠오르는, 반복문과 if문의 조합으로 풀어봤는데 아래처럼 시간 복잡도가.. 정확도 100%, 성능에서 0%가 나와서 50점을 기록했다.

Detected time complexity: **O(B-A)**

```java
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int A, int B, int K) {
        int count = 0;
        for(int i=A; i<=B; i++) {
            count += i%K == 0 ? 1 : 0;
        }
        return count;
    }
}
```

뭔가 한번에 나눠서 바로 결과가 나와줘야 할 것 같은데 어떻게 풀까..

생각해보니까 0과 1500이란 두 숫자가 있고, K에 3을 주면 단순히 1500에 3을 나누면 답이 나온다. 거기서 범위가 0부터가 아닌 A부터로 추가된거니까 그냥 B에 3을 나누어 값을 구하고, A에 3을 나눈 값을 빼자라는 결론이 나왔다.

```java
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int A, int B, int K) {
        // write your code in Java SE 8
        int solve = B/K - A/K;
        return solve;
    }
}
```

근데 문제는 A가 나누어지는 경우 포함되어야 하기 때문에 아래 예외조건을 추가하니까 통과.

## 최종 코드

```java
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int A, int B, int K) {
        // write your code in Java SE 8
        int solve = B/K - A/K;
        solve += A%K == 0 ? 1 : 0;
        return solve;
    }
}
```

