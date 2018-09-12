## Hackerrank - Bear and Steady Gene


[Bear and Steady Gene](https://www.hackerrank.com/challenges/bear-and-steady-gene/problem)

A gene is represented as a string of length  (where  is divisible by ), composed of the letters , , , and . It is considered to be *steady* if each of the four letters occurs exactly  times. For example,  and  are both steady genes.

Bear Limak is a famous biotechnology scientist who specializes in modifying bear DNA to make it steady. Right now, he is examining a gene represented as a string . It is not necessarily steady. Fortunately, Limak can choose one (maybe empty) substring of  and replace it with any string of the same length.

Modifying a large substring of bear genes can be dangerous. Given a string , can you help Limak find the length of the smallest possible substring that he can replace to make  a steady gene?

*Note*: A substring of a string  is a subsequence made up of zero or more *contiguous* characters of .

**Input Format**

The first line contains an interger  divisible by , denoting the length of a string . 
The second line contains a string  of length .

**Constraints**

- `4 <= n <= 500,000`
- N is divisible by 4

**Subtask**

- `4 <= n <= 2000` in tests worth  points.

**Output Format**

Print the length of the minimum length substring that can be replaced to make  stable.

**Sample Input**

```
8  
GAAATAAA
```

**Sample Output**

```
5
```

**Explanation**

One optimal solution is to replace **AAATA**  with **TTCCG** resulting in **GTTCCGAA**
The replaced substring has length 5



유전자의 배열에서 A, T, C, G가 같은 횟수로 나오면 안정적인 것으로 간주된다.

AATTCCGG처럼 각각의 해당하는 알파벳이 나온 횟수가 같도록 부분 문자열을 대체하는데, 그중 가장 짧은 문자열의 길이를 구하는 문제이다.

GAAATAAA가 나오면, G는 1, A는 6, T가 1이 나오므로 안정적인 유전자 배열이 되지 않는다.

여기서 AAATA를 TTCCG로 변경하면.. GTTCCGAA로 G가 2번, T가 2번, C가 2번, A가 2번으로 안정적인 유전자 문자열을 가지게 된다.



일단 이중 포문으로 문자의 첫번째부터 자르고, 두번째부터 잘라보고, 세번째부터 잘라보고, 안정적인 유전자 배열이 나올 수 있는 경우중 가장 작은 문자열의 길이를 저장하면 될 것 같다고 생각했고, 실제로 구현해보니까 되긴 되는데... 코드가 참 길고 이중포문을 사용해서 그런지 시간초과가 난다. 

그리고, 이미 안정적인 유전 배열을 가진 문자에 대한 예외처리를 추가하지 않아 에러가 났었다. 아래 부분 추가

```java
if(checkGeneValid(checked, gene.length()/4)) {
    return 0;
}
```

시간초과가 나는 코드는 아래와 같다. 어떻게 해결하지?

```java

import java.util.*;


public class Algorithm {
    static int steadyGene(String gene) {
        // Complete this function
        int min = Integer.MAX_VALUE;
        int n = gene.length();
        Map<String, Integer> checked = new HashMap<>();
        checked.put("A", 0);
        checked.put("C", 0);
        checked.put("G", 0);
        checked.put("T", 0);
        for(int i=0; i<n; i++) {
            String ch = String.valueOf(gene.charAt(i));
            checked.put(ch, checked.get(ch) + 1);
        }
        if(checkGeneValid(checked, gene.length()/4)) {
            return 0;
        }
        for(int i=0; i<n; i++) {
            int j = i, len = 0;
            Map<String, Integer> copiedCheck = getCheckedMap(checked);
            for(; j<n; j++) {
                String ch = String.valueOf(gene.charAt(j));
                copiedCheck.put(ch, copiedCheck.get(ch) - 1);
                len++;
                if(checkGeneValid(copiedCheck, gene.length()/4)) {
                    if(min > len) {
                        min = len;
                    }
                    break;
                }
            }
        }
        return min;
    }
    private static Map<String, Integer> getCheckedMap(Map<String, Integer> checked) {
        Map<String, Integer> copiedCheck = new HashMap<>();
        copiedCheck.put("A", checked.get("A"));
        copiedCheck.put("C", checked.get("C"));
        copiedCheck.put("G", checked.get("G"));
        copiedCheck.put("T", checked.get("T"));
        return copiedCheck;
    }

    private static boolean checkGeneValid(Map<String, Integer> checked, Integer len) {
        if(checked.get("A") <= len && checked.get("C") <= len && checked.get("T") <= len && checked.get("G") <= len) {
            return true;
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        String gene = in.next();
        int result = steadyGene(gene);
        System.out.println(result);
        in.close();
    }
}
```

## 최종 코드

아직 시간복잡도를 더 줄일 방법을 생각하지 못했다.