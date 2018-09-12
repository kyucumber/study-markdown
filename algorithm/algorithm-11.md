## Codility 4-2 FrogRiverOne


Codility 4-2 [FrogRiverOne](https://app.codility.com/programmers/lessons/4-counting_elements/frog_river_one/)

A small frog wants to get to the other side of a river. The frog is initially located on one bank of the river (position 0) and wants to get to the opposite bank (position X+1). Leaves fall from a tree onto the surface of the river.

You are given a zero-indexed array A consisting of N integers representing the falling leaves. A[K] represents the position where one leaf falls at time K, measured in seconds.

The goal is to find the earliest time when the frog can jump to the other side of the river. The frog can cross only when leaves appear at every position across the river from 1 to X (that is, we want to find the earliest moment when all the positions from 1 to X are covered by leaves). You may assume that the speed of the current in the river is negligibly small, i.e. the leaves do not change their positions once they fall in the river.

For example, you are given integer X = 5 and array A such that:

A[0] = 1

A[1] = 3

A[2] = 1

A[3] = 4

A[4] = 2

A[5] = 3

A[6] = 5

A[7] = 4

In second 6, a leaf falls into position 5. This is the earliest time when leaves appear in every position across the river.

Write a function:

> `class Solution { public int solution(int X, int[] A); }`

that, given a non-empty zero-indexed array A consisting of N integers and integer X, returns the earliest time when the frog can jump to the other side of the river.

If the frog is never able to jump to the other side of the river, the function should return −1.

For example, given X = 5 and array A such that:

A[0] = 1

A[1] = 3

A[2] = 1

A[3] = 4

A[4] = 2

A[5] = 3

A[6] = 5

A[7] = 4

the function should return 6, as explained above.

Assume that:

> - N and X are integers within the range [1..100,000];
> - each element of array A is an integer within the range [1..X].

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(X), beyond input storage (not counting the storage required for input arguments).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.



이문제는 일단 이해부터 제대로 잘 되지 않았다. 무슨 소린지 했는데, 결국 앞의 문제들과 연결되는 순열 문제였다.

X가 5가 주어졌을 때, 은행은 X+1 위치에 있기 때문에 6이고, 6에 도달하기 위해서는 1, 2, 3, 4, 5 위치 모두에 잎이 떨어져있어야 한다. 그러니까 결국 1~X까지의 순열이 채워지는 가장 빠른 배열의 인덱스 번호를 구하면 되는 문제인것 같다.

위의 마지막 예제에서는 6번 인덱스에서 1~5까지의 순열이 완성되기 때문에 6을 리턴하면 된다.

일단 순열을 완성하려면 중복을 허용하지않는 자료구조인 Set을 사용하면 좋을 것 같았고..이제 다음 문제는 그 HashSet이 지금 순열을 만족하는지를 체크해야하는데 이걸 어떻게 해야할지가 고민이었다.

```java
public static int solution(int X, int [] A) {
    Set<Integer> per = new HashSet<>();
    for(int i=0; i<A.length; i++) {
        per.add(A[i]);
        //순열인지 체크하고 순열이면 현재 인덱스 반환
    }
    return -1;
}
```

위에서 HashSet에 값을 넣는다 치고, `1, 2, 3, 4, 5`가 완성된걸 어떻게 체크할지 고민했는데, 1~5가 나오기 전 6, 7, 8이 나올수도 있기 때문에 size로 체크하기도 애매하고.. 그러다가 생각해보니까 X보다 큰 값이 나오면 굳이 HashSet에 넣어줄 필요도 없을것 같았다. HashSet에 X보다 큰 값은 넣지 않다가, HashSet의 size가 X와 같아지는 경우에 순열을 만족하는거로 치고 그때의 인덱스 값을 리턴해줬다.

```java
// you can also use imports, for example:
import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int X, int[] A) {
        Set<Integer> per = new HashSet<>();
        for(int i=0; i<A.length; i++) {
            if(A[i] <= X) per.add(A[i]);
            if(per.size() == X) return i;
        }
        return -1;
    }
}
```

Detected time complexity: **O(N)**

시간 복잡도는 반복문 하나로 끝나서 O(N)이다. 만약 처음에 X보다 큰 값을 넣어주고 반복문 안에서 또 HashSet이 순열을 만족하는지 체크했다면 **O(N^2)**이 나왔을거같다.