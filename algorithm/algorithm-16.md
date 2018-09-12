## Hackerrank - Common Child


[Common Child](https://www.hackerrank.com/challenges/common-child/problem)

A string is said to be a child of a another string if it can be formed by deleting 0 or more characters from the other string. Given two strings of equal length, what's the longest string that can be constructed such that it is a child of both?

For example, `ABCD` and `ABDC` have two children with maximum length 3, `ABC` and `ABD`. They can be formed by eliminating either the `D` or `C` from both strings. Note that we will not consider `ABCD` as a common child because we can't rearrange characters and `ABCD`  `ABDC`.

**Input Format**

There is one line with two space-separated strings,  and .

**Constraints**

- All characters are upper case in the range ascii[A-Z].
- 1 <= |s1| , |s2| <= 5000

**Output Format**

Print the length of the longest string , such that  is a child of both  and .

**Sample Input**

```
HARRY
SALLY
```

**Sample Output**

```
 2
```

**Explanation**

The longest string that can be formed by deleting zero or more characters from  and  is , whose length is 2.

**Sample Input 1**

```
AA
BB
```

**Sample Output 1**

```
0
```

**Explanation 1**

 and  have no characters in common and hence the output is 0.

**Sample Input 2**

```
SHINCHAN
NOHARAAA
```

**Sample Output 2**

```
3
```

**Explanation 2**

The longest string that can be formed between  and  while maintaining the order is .

**Sample Input 3**

```
ABCDEF
FBDAMN
```

**Sample Output 3**

```
2
```

**Explanation 3** 
 is the longest child of the given strings.



누가 문제를 링크해주고, 2시간 가량 계속 고민했는데.. 두 문자에서 일부 문자들을 제거하고, 일치하는 문자를 구하는데 그중 가장 긴 수를 구하는 문제였다.

ABCDE

TBTDT

두 문자가 있으면, 첫번째 문자에서 A, C, E를 자르고 두번째 문자에서 T 세개를 각각 잘라서 일치하는 `BD`를 구하고, 가장 긴 일치하는 문자가 `BD`니까 2를 반환하면 되는 문제다.

2시간 가량 고민하면서, 첫번째 문자에 없는 알파벳을 두번째 문자에서 제거, 두번째 문자에 없는 알파벳은 첫번째 문자에서 제거하고 조합을 통해서 모든 지워질 경우의 수를 구해서 제거한 다음 나오는 문자들을 비교해서 가장 긴 문자를 반환해주려고 했는데 구현이 쉽지도 않았고 당연히 시간초과가 나왔다.



최대 2 - 3시간 가량 고민해보고 그때까지 잘 모르겠으면 그냥 답을 보라던 알고리즘을 잘하던 어떤 분의 말을 참고해서 그냥 다른 분들의 코드를 참고해서 문제를 풀었다...



2차원 배열을 만들어서, 문자 2개를 비교하면서 나올 수 있는 최대 길이를 저장하는 방식으로 푼 것 같다.

해당 문자가 어떤건지는 저장하지 않고 길이만 저장하는 방식으로 구현한거 같은데..머리가 정말 좋은것 같다.

```
SHINCHAN
NOHARAAA
```

위의 두 문자가 나왔을 때, 아래의 조건문 3개를 차근차근 살펴보자.

> if (i == 0 || j ==0)

여기는 그냥 문자의 1 인덱스부터 사용하기 때문에 0번째를 0으로 채워주는 부분이다. 별 거 아닌 부분



> Else if str1.charAt(i-1)==str2.charAt(j-1)

첫번째로 같은 문자가 나오는 `L[2][3]`부터 차근차근 진행하면 첫번째 문자의 H와, 두번째 문자의 H을 비교해서 같으니까 `L[2][3]`에 `L[1][2]` 값에 1을 더한 값을 넣는다.

첫번째 문자의 이전 문자(S)와 두번째 문자의 이전 문자(O)까지를 비교해서 나온 가장 큰 문자의 길이에 1을 더한다.



> Else, 만약 값이 같지 않은 경우? (str1.charAt(i-1)!=str2.charAt(j-1))

그리고 다음으로 넘어가서, `L[2][4]`로 넘어가면. 첫번째 SHINCHAN의 H와 두번째 NOHARAAA에서 A를 비교한다.

A는 H와 다르기 때문에 `L[1][4], L[2][3]` 중 큰 값을 `L[2][4]`에 할당하고 넘어간다.

이 부분은 첫번째 SHINCHAN에서 S까지, NOHARAAA에서 A까지 나온 가장 큰 값과 SHINCHAN에서 H까지, NOHARAAA에서 H까지 나온 가장 큰 값을 비교해 둘중 큰 값을 저장하는 부분이다.



솔직히 아직 완전 이해되지 않았는데, 각각의 이차원 배열에 나올 수 있는 가장 큰 길이를 저장해두고 넘어가면서 비교해 최종적으로 배열의 `L[str1.length()][str2.length()];`에 할당된 값에 답이 저장되게 된다.



## 최종 코드 - 자바

```java
import java.io.*;
import java.util.*;
import java.text.*;
import java.math.*;
import java.util.regex.*;

public class Solution {

    static int commonChild(String str1, String str2){
        int L[][] = new int[str1.length()+1][str2.length()+1];
        for(int i=0;i<=str1.length();i++){
            for(int j=0;j<=str2.length();j++){
                if(i==0 || j==0)
                    L[i][j]=0;
                else if(str1.charAt(i-1)==str2.charAt(j-1)){
                    L[i][j] = L[i-1][j-1]+1;
                }
                else{
                    L[i][j] = Math.max(L[i-1][j],L[i][j-1]);
                }
            }
        }
        return L[str1.length()][str2.length()];
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        String s1 = in.next();
        String s2 = in.next();
        int result = commonChild(s1, s2);
        System.out.println(result);
    }
}

```

## 최종 코드 - 자바스크립트

프론트도 같이 다룰 일이 많아져서 프론트 공부 겸, 자바스크립트로도 짜보고 있다. 잘 모르겠다..

```js
process.stdin.resume();
process.stdin.setEncoding('ascii');

var input_stdin = "";
var input_stdin_array = "";
var input_currentline = 0;

process.stdin.on('data', function (data) {
    input_stdin += data;
});

process.stdin.on('end', function () {
    input_stdin_array = input_stdin.split("\n");
    main();    
});

function readLine() {
    return input_stdin_array[input_currentline++];
}

/////////////// ignore above this line ////////////////////

function commonChild(s1, s2){
    let check = new Array(s1.length + 1);
    for(var i=0; i<=s1.length; i++) {
        check[i] = new Array(s2.length + 1);
    }
    for(let i=0; i<=s1.length; i++) {
        for(let j=0; j<=s2.length; j++){
            if(i==0 || j==0) {
                check[i][j] = 0;
            } else if(s1.charAt(i-1) === s2.charAt(j-1)) {
                check[i][j] = check[i-1][j-1] + 1;
            }else {
                check[i][j] = Math.max(check[i-1][j], check[i][j-1]);
            }
        }
    }
    return check[s1.length][s2.length];
}

function main() {
    var s1 = readLine();
    var s2 = readLine();
    var result = commonChild(s1, s2);
    process.stdout.write("" + result + "\n");

}
```

