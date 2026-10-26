#문제4: 높이 N을 입력받아 중앙 정렬된 피라미드 형태의 문자 패턴을 출력하시오.

```
#include <stdio.h>

int main() {

	int N;
	printf("숫자 입력: ");
	scanf_s("%d", &N);

	for (int i = 1; i <= N; i++) {
		for (int k = 0; k < N - i; k++)printf(" ");
		for (int j = 0; j < (i * 2) - 1; j++) {
			if (i % 2 == 0) printf("#");
			else printf("*");
		}
		printf("\n");
	}

	
}

```
