# 문제4: 높이 N을 입력받아 중앙 정렬된 피라미드 형태의 문자 패턴을 출력하시오.

# 답안4:

- 소스코드:
  
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
- 사진:
  <img width="1119" height="476" alt="image" src="https://github.com/user-attachments/assets/8dea67b1-b861-412a-9cb9-d7ce22e40f18" />
