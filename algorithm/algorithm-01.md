## 가장 큰 이어지는 원소들의 합 구하기 



정수 배열(int array)가 주어지면 가장 큰 이어지는 원소들의 합을 구하시오. 단, 시간복잡도는 O(n).



Example

>  Input: [-1, 3, -1, 5]
>
>  Output: 7 // 3 + (-1) + 5

> Input: [-5, -3, -1]
>
> Output: -1 // -1

>  Input: [2, 4, -2, -3, 8]
>
>  Output: 9 // 2 + 4 + (-2) + (-3) + 8



시간복잡도 O(n) 이니까 반복문 한번으로 풀어야 할 것 같다.

이어지는 가장 큰 값이니까, 제일 처음 값에서 더해가면서 더한 값보다 다음 값이 크면 이어지는 연결을 자르고, 다음값부터 다시 시작한다.

예를 들어서 -1, 3 에서 (-1 + 3) < 3이니까 -1을 버리고, 3부터 이어지는 연결을 새로 만든다.

currentSum = max(2, 3) = 3 여기서 다시 다음값부터 더해가며 반복하고

max와 currentSum을 비교해 더 큰 값을 max에 저장해두고 최종적으로 가장 컸던 이어지는 합인 max를 리턴해준다.

```java
public class Algorithm {
    public static void main(String[] args) {
        Algorithm algorithm = new Algorithm();
        int arr[] =  {-1, 3, -1, 5};
        System.out.println(algorithm.solve(arr));
    }
    public int solve(int arr[]) {
        int currentSum = arr[0];
        int max = arr[0];
        for(int i = 1; i<arr.length; i++) {
            currentSum = Math.max(currentSum + arr[i], arr[i]);
            max = Math.max(currentSum, max);
        }
        return max;
    }
}
```