# 문제3: 숫자야구 판정 게임을 작성하시오

- 소스코드:

```
#include <stdio.h>

int main() {

	
	int num[] = { 1,3,5 };

	int usr[3];
	int str = 0, ball = 0;

	printf("숫자 3개 입력(중복X): ");
	scanf_s("%d %d %d", &usr[0], &usr[1], &usr[2]);

	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			if (num[i] == usr[j]) {
				if (i == j)
					str++;
				else
					ball++;
			}
		}
	}

	printf("결과: %d strike, %d ball\n", str, ball);
	if (str == 3) printf("HOMERUN");

	
}

```
