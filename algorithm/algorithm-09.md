## Codility 3-3 PermMissingElem


Codility 3-3 [PermMissingElem](https://app.codility.com/programmers/lessons/3-time_complexity/perm_missing_elem/)

A zero-indexed array A consisting of N different integers is given. The array contains integers in the range [1..(N + 1)], which means that exactly one element is missing.

Your goal is to find that missing element.

Write a function:

> `int solution(int A[], int N);`

that, given a zero-indexed array A, returns the value of the missing element.

For example, given array A such that:

  A[0] = 2  A[1] = 3  A[2] = 1  A[3] = 5

the function should return 4, as it is the missing element.

Assume that:

> - N is an integer within the range [0..100,000];
> - the elements of A are all distinct;
> - each element of array A is an integer within the range [1..(N + 1)].

Complexity:

> - expected worst-case time complexity is O(N);
> - expected worst-case space complexity is O(1), beyond input storage (not counting the storage required for input arguments).

이 문제는 연속된 숫자 사이에 빠진 값 하나를 찾는 문제다.

처음에는 A만큼의 크기를 갖는 boolean 배열을 만들고 true false로 값 유무를 체크해주려고 했다. 근데 공간 복잡도가 O(1)이라서 아래와 같이 전체의 합 - 하나가 빠진 합을 통해 값을 유추했는데..

```java
public int solution(int [] A) {
    return IntStream.range(1, A.length + 2).sum() - Arrays.stream(A).sum();
}
```

위는 한줄로 간단하게 출력할 수 있는데 문제가 시간 복잡도가 N * logN이 나와서 몇몇 케이스가 시간초과가 떴다.

풀이 자체가 잘못된건가 싶어서 고민해보다가 아래와 같이 for loop 두개로 바꾸니까 성공. 나중에 어떻게 시간을 더 줄일 수 있는지는 고민해봐야 할 듯

```java
public int solution(int[] A) {
        int sum = 0;
        for(int i=1; i<=A.length + 1; i++) {
            sum += i;
        }
        for(int i=0; i<A.length; i++) {
            sum -= A[i];
        }
        return sum;
    }
```