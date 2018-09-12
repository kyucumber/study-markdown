## Codility 4-1 PermCheck


Codility 4-1  [PermCheck](https://app.codility.com/programmers/lessons/4-counting_elements/perm_check/)

A non-empty zero-indexed array A consisting of N integers is given.

A *permutation* is a sequence containing each element from 1 to N once, and only once.

For example, array A such that:

	A[0] = 4

	A[1] = 1

	A[2] = 3

	A[3] = 2

is a permutation, but array A such that:

	A[0] = 4

	A[1] = 1

	A[2] = 3

is not a permutation, because value 2 is missing.

The goal is to check whether array A is a permutation.

Write a function:

> `int solution(int A[], int N);`

that, given a zero-indexed array A, returns 1 if array A is a permutation and 0 if it is not.

For example, given array A such that:

	A[0] = 4

	A[1] = 1

	A[2] = 3

	A[3] = 2

the function should return 1.

Given array A such that:

	A[0] = 4

	A[1] = 1

	A[2] = 3

the function should return 0.

Assume that:

> - N is an integer within the range [1..100,000];
> - each element of array A is an integer within the range [1..1,000,000,000].

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(N), beyond input storage (not counting the storage required for input arguments).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.



주어진 배열이 순열인지 아닌지를 체크해야 하는 문제.

`4, 1, 3, 2`는 1~4를 가지기 때문에 순열이라서 1을 리턴, `4, 1, 3`은 2가 빠진 순열이기때문에 0을 리턴하는 문제다.

처음에는 아래처럼 풀었다. 이전 문제처럼 전체 합을 비교하면 되지 않을까? 하고.

```java
// you can also use imports, for example:
import java.util.*;
import java.util.Arrays;
import java.util.stream.IntStream;
// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        long sum = IntStream.range(1, Arrays.stream(A).max().getAsInt() + 1).sum();
        return (Arrays.stream(A).sum() == sum) ? 1 : 0;
    }
}
```

근데 일단 스트림을 써서 시간초과가 난다. 그리고 더 큰 문제는 `{2, 2, 2, 4}`랑 `{1, 2, 3, 4}`는 같은 합을 가지기 때문에 순열의 조건에 맞지 않는다.

그리고 두번째는..해당 값이 있는지 체크하는 boolean 배열을 만들어서 체크하는 방법이었다. A 배열과 같은 크기의 배열이 있어야 하기 때문에 공간 복잡도가 늘어나지만 확실히 될 것 같았음.

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        int n = Arrays.stream(A).max().getAsInt();
        Boolean check[] = new Boolean[n + 1];
        Arrays.fill(check, false);
        check[0] = true;
        for(int i=0; i<A.length; i++) {
            if(n < A[i]) return 0;
            check[A[i]] = true;
        }
        return (Arrays.stream(check).filter(c -> c == false).count() == 0) ? 1 : 0;
    }
}
```

위의 방법에서는 check할 배열의 크기를 A배열의 최대값으로 줬다. `4,5,6,7 ... 5000`인 경우 4~5000까지의 순서쌍이 맞으니까 저런걸 체크할 수 있어야 할것 같아서 저렇게 줬는데. 저렇게 주는 경우 `1, 1`이런 순서쌍에 대해서 크기 1의 check 배열이 생성되고, A의 길이는 2이기 때문에 에러가 난다.

```java
// you can also use imports, for example:
import java.util.*;
import java.util.Arrays;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        Boolean check[] = new Boolean[A.length + 1];
        Arrays.fill(check, false);
        check[0] = true;
        for(int i=0; i<A.length; i++) {
            if(A.length < A[i]) return 0;
            check[A[i]] = true;
        }
        return (Arrays.stream(check).filter(c -> c == false).count() == 0) ? 1 : 0;
    }
}
```

마지막으로 위처럼 구현했을때 답은다 맞지만 시간초과가 나는 테스트케이스가 몇 있었다.



#### 최종 코드

아래처럼 스트림을 걷어내고, for loop로 만들면 아래와 같은 시간복잡도와, 100% 정답률이 나온다.

별로 안 어려운 문제인데 너무 고생을 많이 한 것 같다..

Detected time complexity: **O(N) or O(N \* log(N))**

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        Boolean check[] = new Boolean[A.length + 1];
        Arrays.fill(check, false);
        check[0] = true;
        for(int i=0; i<A.length; i++) {
            if(A.length < A[i]) return 0;
            check[A[i]] = true;
        }
        for(int i=0; i<check.length; i++) {
            if(check[i] == false) return 0;
        }
        return 1;
    }
}
```

