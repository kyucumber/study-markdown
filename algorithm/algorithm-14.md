## 백준 온라인 저지 - 연결 요소의 개수

방향 없는 그래프가 주어졌을 때, 연결 요소 (Connected Component)의 개수를 구하는 프로그램을 작성하시오.

## 입력

첫째 줄에 정점의 개수 N과 간선의 개수 M이 주어진다. (1 ≤ N ≤ 1,000, 0 ≤ M ≤ N×(N-1)/2) 둘째 줄부터 M개의 줄에 간선의 양 끝점 u와 v가 주어진다. (1 ≤ u, v ≤ N, u ≠ v) 같은 간선은 한 번만 주어진다.

## 출력

첫째 줄에 연결 요소의 개수를 출력한다.

## 예제 입력 1 복사

```
6 5
1 2
2 5
5 1
3 4
4 6
```

## 예제 출력 1 복사

```
2
```

## 예제 입력 2 복사

```
6 8
1 2
2 5
5 1
3 4
4 6
5 4
2 4
2 3
```

## 예제 출력 2 복사

```
1
```

## 힌트

## 출처

- 문제를 만든 사람: [baekjoon](https://www.acmicpc.net/user/baekjoon)
- 잘못된 조건을 찾은 사람: [songjuh](https://www.acmicpc.net/user/songjuh)

------

연결 요소를 찾는 문제이다.  

연결 요소를 찾는 문제이다.. 연결 요소가 뭔지 하고 한번 찾아봤는데..

연결 요소는 아래처럼 그래프가 나누어져 있는 경우, 각각의 요소를 연결 요소라고 한다. 연결요소는 아래와 같은 조건을 가진다.

- 연결 요소에 속한 모든 정점을 연결하는 경로가 있어야 한다.
- 다른 연결 요소와 연결되는 경로가 있으면 안된다.

![](/Users/kyunam/Documents/hexo/hexo/themes/overdose/source/images/data/connectedComponent.png)

DFS나 BFS로 탐색하면 쉽게 풀 수 있을 것 같다.

dfs 탐색을 1번 정점부터 시작하는데 만약 모든 그래프가 서로 연결되었다면, 1번 정점에서 시작한 dfs 탐색으로 모든 check 배열이 true가 되어 그대로 끝날거고 만약 그래프가 위처럼 나누어져 있다면 여러번 dfs 탐색을 할 것 같다.

## 최종 코드

```java
import java.util.*;

public class Main {
    public static void dfs(List<List<Integer>> graph, boolean check[], int x) {
        check[x] = true;
        for(int v : graph.get(x)) {
            if(check[v] == false){
                dfs(graph, check, v);
            }
        }
    }
    public static void inputGraph() {
        Scanner sc = new Scanner(System.in);
        int v = sc.nextInt();
        int e = sc.nextInt();
        boolean check[] = new boolean[v + 1];
        List<List<Integer>> graph = new ArrayList<>();
        for(int i=0; i<=v; i++) graph.add(new ArrayList<>());
        for(int i=0; i<e; i++) {
            int x = sc.nextInt();
            int y = sc.nextInt();
            graph.get(x).add(y);
            graph.get(y).add(x);
        }
        int solve = 0;
        for(int i=1; i<=v; i++) {
            if(check[i] == false) {
                dfs(graph, check, i);
                solve++;
            }
        }
        System.out.println(solve);
    }
    public static void main(String args[]) {
        inputGraph();
    }
}
```

