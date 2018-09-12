## 프로그래머스 숫자의 표현 - Level 4

수학을 공부하던 민지는 재미있는 사실을 발견하였습니다. 그 사실은 바로 연속된 자연수의 합으로 어떤 숫자를 표현하는 방법이 여러 가지라는 것입니다. 예를 들어, 15를 표현하는 방법은

> (1+2+3+4+5)
> (4+5+6)
> (7+8)
> (15)

로 총 4가지가 존재합니다. 숫자를 입력받아 연속된 수로 표현하는 방법을 반환하는 expressions 함수를 만들어 민지를 도와주세요. 예를 들어 15가 입력된다면 4를 반환해 주면 됩니다.



그냥 엄청 단순하게 생각하면 1부터 연속되게 더해서 15가 나오는지, 2부터 더해서 연속해서 15가 나오는지, ... 14랑 15를 더해서 15가 나오는지.. 그런식으로 풀면 쉽게 답은 구할 수 있다.

근데 시간 복잡도가 O(N^2)인데 혹시 더 좋은 방법이 없을까..

```c++
#include<iostream>
using namespace std;
int expressions(int testCase)
    {
        int count = 0;
        int sum = 0;
        for(int i=1; i<=testCase; i++) {
            sum = 0;
            for(int j=i; j<=testCase; j++) {
                sum += j;
                if(sum > testCase) break;
                if(sum == testCase) count++;
            }
        }
        return count;
    }

int main()
{
	int testNo = 15;
	int testAnswer = expressions(testNo);
// 아래는 테스트로 출력해 보기 위한 코드입니다.
	cout<<testAnswer;
}

```
