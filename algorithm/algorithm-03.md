## Codility 3-1 TapeEquilibrium

[TapeEquilibrium](https://app.codility.com/programmers/lessons/3-time_complexity/tape_equilibrium/)

A non-empty zero-indexed array A consisting of N integers is given. Array A represents numbers on a tape.

Any integer P, such that 0 < P < N, splits this tape into two non-empty parts: A[0], A[1], ..., A[P − 1] and A[P], A[P + 1], ..., A[N − 1].

The *difference* between the two parts is the value of: |(A[0] + A[1] + ... + A[P − 1]) − (A[P] + A[P + 1] + ... + A[N − 1])|

In other words, it is the absolute difference between the sum of the first part and the sum of the second part.

For example, consider array A such that:

  A[0] = 3  A[1] = 1  A[2] = 2  A[3] = 4  A[4] = 3

We can split this tape in four places:

> - P = 1, difference = |3 − 10| = 7 
> - P = 2, difference = |4 − 9| = 5 
> - P = 3, difference = |6 − 7| = 1 
> - P = 4, difference = |10 − 3| = 7 

Write a function:

> `int solution(int A[], int N);`

that, given a non-empty zero-indexed array A of N integers, returns the minimal difference that can be achieved.

For example, given:

  A[0] = 3  A[1] = 1  A[2] = 2  A[3] = 4  A[4] = 3

the function should return 1, as explained above.

Assume that:

> - N is an integer within the range [2..100,000];
> - each element of array A is an integer within the range [−1,000..1,000].

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(N), beyond input storage (not counting the storage required for input arguments).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.

배열을 두 구간으로 나누고, 그 구간들간의 합의 차의 절대값이 가장 적은 값을 구하는 문제.

이번 문제는, 얼마 안 걸려 풀기는 했는데 계속 정답률이 41%, 83%가 나왔다.

```Java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        int sum = 0;
        int currentSum = 0;
        int solve = 0;
        for(int i=0; i<A.length; i++) {
            sum += A[i];
        }
        for(int i=0; i<A.length-1; i++){
        	currentSum += A[i];
        	int v = Math.abs(currentSum - (sum - currentSum));
        	if(i==0) solve = v;
        	Math.min(solve, v);
        	if(v < solve) solve = v;
        }
        return solve;
    }
}
```

위의 반복문에서` i=1, i<A.length;` 로 준 상태에서 41%의 정답률, `i=0, i<A.length`에서 83% 가 나왔다.

`i=0, i<A.length-1`로 주어야 위의 문제의 조건에 맞는 결과가 나올 것 같다.

시간 복잡도를 `O(n^2)`이 나오지 않게 하기 위해서, sum을 구하고 빼나가는 방식으로 구현했다. stream으로 sum을 구하려고 했으나 스트림이 더 성능이 안좋다는 말을 들어서 사용하지 않았다.