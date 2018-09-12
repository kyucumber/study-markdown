## 백준 온라인 저지 - DFS와 BFS


# DFS와 BFS

| 시간 제한 | 메모리 제한 | 제출  | 정답  | 맞은 사람 | 정답 비율 |
| --------- | ----------- | ----- | ----- | --------- | --------- |
| 2 초      | 128 MB      | 35152 | 10819 | 6520      | 29.311%   |

## 문제

그래프를 DFS로 탐색한 결과와 BFS로 탐색한 결과를 출력하는 프로그램을 작성하시오. 단, 방문할 수 있는 정점이 여러 개인 경우에는 정점 번호가 작은 것을 먼저 방문하고, 더 이상 방문할 수 있는 점이 없는 경우 종료한다. 정점 번호는 1번부터 N번까지이다.

## 입력

첫째 줄에 정점의 개수 N(1 ≤ N ≤ 1,000), 간선의 개수 M(1 ≤ M ≤ 10,000), 탐색을 시작할 정점의 번호 V가 주어진다. 다음 M개의 줄에는 간선이 연결하는 두 정점의 번호가 주어진다. 한 간선이 여러 번 주어질 수도 있는데, 간선이 하나만 있는 것으로 생각하면 된다. 입력으로 주어지는 간선은 양방향이다.

## 출력

첫째 줄에 DFS를 수행한 결과를, 그 다음 줄에는 BFS를 수행한 결과를 출력한다. V부터 방문된 점을 순서대로 출력하면 된다.

## 예제 입력 1 복사

```
4 5 1
1 2
1 3
1 4
2 4
3 4
```

## 예제 출력 1 복사

```
1 2 4 3
1 2 3 4
```

## 힌트

## 출처

- 문제를 만든 사람: [author5](https://www.acmicpc.net/user/author5)
- 빠진 조건을 찾은 사람: [pumpyboom](https://www.acmicpc.net/user/pumpyboom)

------

dfs랑 bfs를 구현해보는 문제.

다른 조건 없이 그냥 출력만 하면 되는 문제이다. 근데 구현을 이상하게 해서 에러가 발생했다.

> ArrayList 크기는 정점의 갯수만큼인데 아래에서는 엣지의 갯수만큼으로 반복문을 돌리고 있다. 그래서 런타임 에러 발생.

```java
import java.util.*;

public class Main {
    private static List<List<Integer>> graph;
    private static boolean check[];
    
    public static void main(String[] args) {
        inputGraph();

    }
    //가중치가 없는 양방향 그래프, 인접 리스트
    public static void inputGraph() {
        Scanner sc = new Scanner(System.in);
        int v = sc.nextInt();
        int e = sc.nextInt();
        int start = sc.nextInt();
        graph = new ArrayList<>();
        for(int i=0; i<e; i++) graph.add(new ArrayList<>());
        for(int i=0; i<e; i++) {
            int x = sc.nextInt();
            int y = sc.nextInt();
            graph.get(x).add(y);
            graph.get(y).add(x);
        }
        for(int i=0; i<v; i++) {
            Collections.sort(graph.get(i));
        }
        check = new boolean[v + 1];
        dfs(start);
        System.out.println();
        check = new boolean[v + 1];
        bfs(start);
    }
    
    public static void dfs(int x) {
        System.out.print(x + " ");
        for(int e : graph.get(x)) {
            if(check[e] == false) {
                check[x] = true;
                dfs(e);
            }
        }
    }
    public static void bfs(int start) {
        Queue<Integer> bfsQueue = new LinkedList<>();
        bfsQueue.add(start);
        check[start] = true;
        while(!bfsQueue.isEmpty()) {
            int v = bfsQueue.remove();
            System.out.print(v + " ");
            for(int e : graph.get(v)) {
                if(check[e] == false) {
                    check[e] = true;
                    bfsQueue.add(e);
                }
            }
        }
    }
}
```

## 최종 코드

위의 문제를 해결하고 코드를 조금 수정했다. 통과

```java
import java.util.*;

public class Main {
    private static List<List<Integer>> graph;
    private static boolean check[];
    
    public static void main(String[] args) {
        inputGraph();

    }
    //가중치가 없는 양방향 그래프, 인접 리스트
    public static void inputGraph() {
        Scanner sc = new Scanner(System.in);
        int v = sc.nextInt();
        int e = sc.nextInt();
        int start = sc.nextInt();
        graph = new ArrayList<>();
        for(int i=0; i<=v; i++) graph.add(new ArrayList<>());
        for(int i=0; i<e; i++) {
            int x = sc.nextInt();
            int y = sc.nextInt();
            graph.get(x).add(y);
            graph.get(y).add(x);
        }
        for(int i=0; i<=v; i++) {
            Collections.sort(graph.get(i));
        }
        check = new boolean[v + 1];
        dfs(start);
        System.out.println();
        check = new boolean[v + 1];
        bfs(start);
    }
    
    public static void dfs(int x) {
        check[x] = true;
        System.out.print(x + " ");
        for(int e : graph.get(x)) {
            if(check[e] == false) {
                dfs(e);
            }
        }
    }
    public static void bfs(int start) {
        Queue<Integer> bfsQueue = new LinkedList<>();
        bfsQueue.add(start);
        check[start] = true;
        while(!bfsQueue.isEmpty()) {
            int v = bfsQueue.remove();
            System.out.print(v + " ");
            for(int e : graph.get(v)) {
                if(check[e] == false) {
                    check[e] = true;
                    bfsQueue.add(e);
                }
            }
        }
    }
}
```

