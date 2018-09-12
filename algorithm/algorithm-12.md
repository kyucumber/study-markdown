## Codility 4-3 MissingInteger


#### [MissingInteger](https://app.codility.com/programmers/lessons/4-counting_elements/missing_integer/)

This is a demo task.

Write a function:

> `class Solution { public int solution(int[] A); }`

that, given an array A of N integers, returns the smallest positive integer (greater than 0) that does not occur in A.

For example, given A = [1, 3, 6, 4, 1, 2], the function should return 5.

Given A = [1, 2, 3], the function should return 4.

Given A = [−1, −3], the function should return 1.

Assume that:

> - N is an integer within the range [1..100,000];
> - each element of array A is an integer within the range [−1,000,000..1,000,000].

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(N), beyond input storage (not counting the storage required for input arguments).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.



이 문제는 A 배열에서 발생하지 않는 가장 작은 양의 정수 (0보다 큼)를 반환하는 문제인데.. 앞의 순열문제들과 이어지는 느낌이다. 음수인 경우 가장 작은 양의 정수인 1을 반환하면 되고.. 빠진 숫자가 있다면 그 숫자를, 만약 모든 숫자를 가지고 있다면, 최대값보다 1 큰 숫자를 반환하면 되는 문제다.

일단 제일 처음에 든 생각은, 배열의 최대값이 음수면 1 반환, 순열을 만족하는 경우 최대값보다 1 큰 값 반환, 없는 값이 있다면 그 값을 반환하면 될 것 같은데.. 그래서 앞의 문제를 응용해서 체크할 배열을 만들고 false로 채운 뒤, 최대값이 음수면 1을 반환하고 false가 되는 최초의 값을 리턴, 없으면 최대값 + 1을 리턴하도록 구현했다.

아래처럼 구현했는데 일단 예상한대로 시간초과도 그렇고 저번부터 하는 바보같은 실수인데, check 배열 크기는 max + 1인데 아래에 A.lengh까지 돌게 만들어서 값이 제대로 나오지 않았다.

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        int max = Arrays.stream(A).max().getAsInt();
        Boolean check[] = new Boolean[max + 1];
        Arrays.fill(check, false);
        if(max < 0) { return 1; }
        for (int i = 0; i < A.length; i++) {
            check[A[i]] = true;
        }
        for(int i = 1; i< A.length; i++) {
            if(!check[i]) return i;
        }
        return max + 1;
    }
}
```

아래처럼 수정하면 답은 다 맞는것 같은데, TIMEOUT, RUNTIME 에러가 난다. 아마 배열 크기 때문에 런타임 에러가 발생할테고, 타임아웃은 시간 복잡도를 넘겨서 그럴것 같다.

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        int max = Arrays.stream(A).max().getAsInt();
        Boolean check[] = new Boolean[max + 1];
        Arrays.fill(check, false);
        if(max < 0) { return 1; }
        for (int i = 0; i < A.length; i++) {
            check[A[i]] = true;
        }
        for(int i = 1; i< max + 1; i++) {
            if(!check[i]) return i;
        }
        return max + 1;

    }
}
```

HashSet을 이용해 구현해보기로 하고, 아래처럼 구현했는데.. 하나의 케이스에서 시간초과.

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        Set<Integer> set = new HashSet<>();
        int max = Arrays.stream(A).max().getAsInt();
        for(int i=0; i<A.length; i++) {
            set.add(A[i]);
        }
        for(int i=1; i<=max+1; i++) {
            if(!set.contains(i)) { return i; }
        }
        return 1;
    }
}
```

아마 Stream으로 max를 구해서 시간초과가 난것같은데 max를 스트림을 이용하지 않고 구하던가 아니면 그냥 최대값을 임의로 만들어서 찾아야 시간초과가 안날것 같다.

## 최종 코드

최대값을 구하면 어떻게든 시간초과가 날 것 같아서 그냥 `Integer.MAX_VALUE`로 반복문을 돌렸다. 100%로 통과

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int[] A) {
        Set<Integer> set = new HashSet<>();
        for(int a : A) {
            set.add(a);
        }
        for(int i=1; i<=Integer.MAX_VALUE; i++) {
            if(!set.contains(i)) { return i; }
        }
        return -1;
    }
}
```

