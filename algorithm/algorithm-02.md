## N보다 작은 짝수 피보나치 수의 합

피보나치 배열은 0과 1로 시작하며, 다음 피보나치 수는 바로 앞의 두 피보나치 수의 합이 된다. 정수 N이 주어지면, N보다 작은 모든 짝수 피보나치 수의 합을 구하여라.

Example

> Input: N = 12
>
> Output: 10 // 0, 1, 2, 3, 5, 8 중 짝수인 2 + 8 = 10.

피보나치를 구하는 재귀함수를 만들고, solve 함수에서 피보나치의 결과가 n보다 작을때까지 반복문을 돌리면서 짝수인 경우 sum에 합산하고 합을 리턴하는 방식으로 해결했다. 맞는지는 잘 모르겠다.

```java
public class Algorithm {
    public static void main(String[] args) {
        Algorithm al = new Algorithm();
        System.out.println(al.solve(12));
    }
    public int solve(int n) {
        int sum = 0;
        int val = 0;
        for(int i=0; n >= val; i++) {
            val = getFibonacci(i);
            if(val%2 == 0) {
                sum += val;
            }
        }
        return sum;
    }
    public int getFibonacci(int n) {
        if(n == 1 || n == 0){
           return n;
        }
        return getFibonacci(n - 1) + getFibonacci(n - 2);
    }
}
```
