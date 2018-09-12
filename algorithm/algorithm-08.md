## Codility 3-2 FrogJmp

Codility 3-2 [FrogJmp](https://app.codility.com/programmers/lessons/3-time_complexity/frog_jmp/)

A small frog wants to get to the other side of the road. The frog is currently located at position X and wants to get to a position greater than or equal to Y. The small frog always jumps a fixed distance, D.

Count the minimal number of jumps that the small frog must perform to reach its target.

Write a function:

> `class Solution { public int solution(int X, int Y, int D); }`

that, given three integers X, Y and D, returns the minimal number of jumps from position X to a position equal to or greater than Y.

For example, given:

  X = 10  Y = 85  D = 30

the function should return 3, because the frog will be positioned as follows:

> - after the first jump, at position 10 + 30 = 40
> - after the second jump, at position 10 + 30 + 30 = 70
> - after the third jump, at position 10 + 30 + 30 + 30 = 100

Assume that:

> - X, Y and D are integers within the range [1..1,000,000,000];
> - X ≤ Y.

Complexity:

> - expected worst-case time complexity is O(1);
> - expected worst-case space complexity is O(1).

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.



Y위치까지 도달하기 위해서 D의 길이만큼 점프를 몇번 하면 되는지를 구하는 문제이다.

이 문제는 위에 풀었던 코딜리티 문제에 비해 너무 쉬웠다..처음에 아무 생각 없이 반복문으로 풀었다가 시간 복잡도를 초과한 것 빼면 O(1)을 잘 보고 풀자.



```java
// you can also use imports, for example:
// import java.util.*;

// you can write to stdout for debugging purposes, e.g.
// System.out.println("this is a debug message");

class Solution {
    public int solution(int X, int Y, int D) {
        int val = Y-X;
        int solve = val/D;
        return val%D == 0 ? solve : solve + 1;
    }
}
```

