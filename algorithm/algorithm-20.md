##  Codility 5-3 GenomicRangeQuery

Codility 5-3 [GenomicRangeQuery](https://app.codility.com/programmers/lessons/5-prefix_sums/genomic_range_query/)

A DNA sequence can be represented as a string consisting of the letters `A`, `C`, `G` and `T`, which correspond to the types of successive nucleotides in the sequence. Each nucleotide has an *impact factor*, which is an integer. Nucleotides of types `A`, `C`, `G` and `T` have impact factors of 1, 2, 3 and 4, respectively. You are going to answer several queries of the form: What is the minimal impact factor of nucleotides contained in a particular part of the given DNA sequence?

The DNA sequence is given as a non-empty string S = `S[0]S[1]...S[N-1]` consisting of N characters. There are M queries, which are given in non-empty arrays P and Q, each consisting of M integers. The K-th query (0 ≤ K < M) requires you to find the minimal impact factor of nucleotides contained in the DNA sequence between positions P[K] and Q[K] (inclusive).

For example, consider string S = `CAGCCTA` and arrays P, Q such that:

​    P[0] = 2    Q[0] = 4     P[1] = 5    Q[1] = 5     P[2] = 0    Q[2] = 6

The answers to these M = 3 queries are as follows:

> - The part of the DNA between positions 2 and 4 contains nucleotides `G` and `C` (twice), whose impact factors are 3 and 2 respectively, so the answer is 2.
> - The part between positions 5 and 5 contains a single nucleotide `T`, whose impact factor is 4, so the answer is 4.
> - The part between positions 0 and 6 (the whole string) contains all nucleotides, in particular nucleotide `A` whose impact factor is 1, so the answer is 1.

Assume that the following declarations are given:

> ```
> struct Results {   int * A;   int M; // Length of the array };
> ```

Write a function:

> ```
> struct Results solution(char *S, int P[], int Q[], int M);
> ```

that, given a non-empty string S consisting of N characters and two non-empty arrays P and Q consisting of M integers, returns an array consisting of M integers specifying the consecutive answers to all queries.

Result array should be returned as a structure `Results`.

For example, given the string S = `CAGCCTA` and arrays P, Q such that:

​    P[0] = 2    Q[0] = 4     P[1] = 5    Q[1] = 5     P[2] = 0    Q[2] = 6

the function should return the values [2, 4, 1], as explained above.

Write an **efficient** algorithm for the following assumptions:

> - N is an integer within the range [1..100,000];
> - M is an integer within the range [1..50,000];
> - each element of arrays P, Q is an integer within the range [0..N − 1];
> - P[K] ≤ Q[K], where 0 ≤ K < M;
> - string S consists only of upper-case English letters `A, C, G, T`.

Copyright 2009–2018 by Codility Limited. All Rights Reserved. Unauthorized copying, publication or disclosure prohibited.

영어라 문제를 이해하는데만도 한참 걸렸다. 영어공부 좀 해야지...

A, C, G, T는 순서대로 1, 2, 3, 4 값을 가지며 주어진 P, Q 배열을 이용해 뉴클레오타이드의 최소 값을 구하는 문제.

> CAGCCTA 라는 문자열이 주어졌을 때,
>
> P[0] = 2, Q[0] = 4인 경우
>
> 2 - 4번째 DNA 서열인 (GCC) 중 가장 작은 값을 가지는 값은 C인 2
>
> P[1] = 5, Q[1] = 5인 경우
>
> 5 - 5번째 DNA 서열인 (T) 중 가장 작은 값은 T인 4
>
> 이런식으로 [2, 4, 1]의 배열을 구하면 되는 문제이다.

난이도가 RESPECTABLE인 것 치고는 쉽다고 생각하고 아래의 방법으로 바로 풀었는데 정답률 100%, 성능 0%가 나왔다.

```java
class Solution {
    private enum NUCLEOTIDE {
        A(1), C(2), G(3), T(4);
        private int value;

        NUCLEOTIDE(int value) {
            this.value = value;
        }

        public int getValue() {
            return value;
        }

        public static int getNum(String str) {
            return valueOf(str).getValue();
        }
    }

    public int[] solution(String S, int[] P, int[] Q) {
        // write your code in Java SE 8
        int[] solveArray = new int[P.length];
        for(int i=0; i<P.length; i++) {
            solveArray[i] = getMinimalImpact(S, P[i], Q[i]);
        }
        return solveArray;
    }

    public int getMinimalImpact(String S, int startIndex, int endIndex) {
        int min = NUCLEOTIDE.getNum(S.charAt(startIndex) + "");
        for(int i=startIndex; i<=endIndex; i++) {
            int currentValue = NUCLEOTIDE.getNum(S.charAt(i) + "");
            if(min > currentValue) {
                min = currentValue;
            }
        }
        return min;
    }
}
```







