## Codility 5-2 PassingCars


#### [PassingCars](https://app.codility.com/programmers/lessons/5-prefix_sums/passing_cars/)

A non-empty array A consisting of N integers is given. The consecutive elements of array A represent consecutive cars on a road.

Array A contains only 0s and/or 1s:

> - 0 represents a car traveling east,
> - 1 represents a car traveling west.

The goal is to count passing cars. We say that a pair of cars (P, Q), where 0 ≤ P < Q < N, is passing when P is traveling to the east and Q is traveling to the west.

For example, consider array A such that:

> A[0] = 0
>
> A[1] = 1
>
> A[2] = 0
>
> A[3] = 1
>
> A[4] = 1

We have five pairs of passing cars: (0, 1), (0, 3), (0, 4), (2, 3), (2, 4).

Write a function:

> `class Solution { public int solution(int[] A); }`

that, given a non-empty array A of N integers, returns the number of pairs of passing cars.

The function should return −1 if the number of pairs of passing cars exceeds 1,000,000,000.

For example, given:

> A[0] = 0
>
> A[1] = 1
>
> A[2] = 0
>
> A[3] = 1
>
> A[4] = 1

the function should return 5, as explained above.

Assume that:

> - N is an integer within the range [1..100,000];
> - each element of array A is an integer that can have one of the following values: 0, 1.

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(1) (not counting the storage required for input arguments).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.

 

0은 동쪽으로 가는 자동차, 1은 서쪽으로 가는 자동차를 뜻하고..P가 동쪽, Q가 서쪽으로 가는 자동차의 쌍을 찾는 문제이다. 단 P는 Q보다 작다.

> A[0] = 0
>
> A[1] = 1
>
> A[2] = 0
>
> A[3] = 1
>
> A[4] = 1

위와 같은 예에서는 동쪽으로가는 0이 있으니까 `(0, 1), (0, 3), (0, 4)` 와 `(2, 3), (2, 4)` 의 5개의 쌍을 구할 수 있다.

최악의 시간 복잡도는 O(N)이니까. 반복문 한번으로 끝내면 될 듯 하다.



> 첫번째 코드

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        // write your code in Java SE 8
        int sum = 0;
        int countOne = Arrays.stream(A).sum();
        for(int i=0; i<A.length; i++) {
            if(A[i] == 0) {
                sum += countOne;
            } else {
                countOne--;
            }
        }
        return sum;
    }
}
```

A 배열의 합은 1의 갯수가 될 테고, 서쪽으로 가는 차의 대수가 나올거다. 아마도

그 합을 가지고 있다가 A 배열의 반복문을 돌면서 A배열에서 동쪽으로 가는 차가 나올 경우 그 차보다 오른쪽에 있는 서쪽으로 가는 차의 대수를 모두 sum에 더한다.

그 과정에서 서쪽으로 가는 차가 나오는 경우 서쪽으로 가는 차의 대수를 하나씩 빼 줌.

정확도는 100%가 나왔고 O(N)일거 같은데 sum을 구하는 과정에서 뭔가 시간이 초과된것 같다.

그리고 문제를 다시 유심히 보니까 100000000 쌍을 넘었을 경우의 예외처리를 해주지 않았다.

> 최종 코드

```java
class Solution {
    public int solution(int[] A) {
        int sum = 0;
        int countOne = 0;
        for(int i=0; i<A.length; i++) {
            countOne+=A[i];
        }
        for(int i=0; i<A.length; i++) {
            if(A[i] == 0) {
                sum += countOne;
                if(1000000000 < sum) {
                    return -1;
                }
            } else {
                countOne--;
            }
        }
        return sum;
    }
}
```

스트림은 쓰지 말자. 그리고 문제의 조건을 좀 더 유심히 살펴보자. 그래도 나름 빠르게 구현 방법을 찾아서 잘 푼거 같다. 쉬워서 그렇겠지만..