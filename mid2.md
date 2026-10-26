# 문제2: SQlite를 활용한 간단한 DB를 설계하고 구현하시오. (3명 이상의 데이터를 넣고 검색을 포함한다)

## 답안2

- 소스코드:

```
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"

// SELECT 쿼리 결과를 깔끔하게 출력하기 위한 콜백 함수
static int callback(void* data, int argc, char** argv, char** azColName) {
    int i;
    printf("----------------------------------------\n");
    for (i = 0; i < argc; i++) {
        printf("%s: %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    return 0;
}

int main() {
    sqlite3* db;
    char* err_msg = 0;
    int rc;

    // 1. 데이터베이스 파일 열기
    rc = sqlite3_open("student.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터베이스를 열 수 없습니다: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    // 2. 기존 테이블 삭제 후 재생성 (초기화)
    const char* sql_drop = "DROP TABLE IF EXISTS students;";
    rc = sqlite3_exec(db, sql_drop, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "기존 테이블 삭제 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
    }

    // 3. 학생 테이블 생성
    const char* sql_create = "CREATE TABLE students ("
        "student_id TEXT PRIMARY KEY, "
        "name TEXT, "
        "department TEXT);";
    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "테이블 생성 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // 4. 5명의 초기 학생 데이터 삽입
    const char* sql_insert =
        "INSERT INTO students (student_id, name, department) VALUES ('2026001', '김철수', '컴퓨터공학과');"
        "INSERT INTO students (student_id, name, department) VALUES ('2026002', '이영희', '전자공학과');"
        "INSERT INTO students (student_id, name, department) VALUES ('2026003', '박민수', '소프트웨어학과');"
        "INSERT INTO students (student_id, name, department) VALUES ('2026004', '정지원', '경영학과');"
        "INSERT INTO students (student_id, name, department) VALUES ('2026005', '최유진', '인공지능학과');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터 삽입 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // 5. [추가] 전체 학생 목록 먼저 출력하기
    printf("========================================\n");
    printf("          [ 전체 학생 목록 ]\n");
    printf("========================================\n");
    const char* sql_select_all = "SELECT student_id AS '학번', name AS '이름', department AS '소속학과' FROM students;";
    rc = sqlite3_exec(db, sql_select_all, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "전체 목록 조회 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
    }
    printf("----------------------------------------\n\n");

    // 6. 검색 기능 구현 (이름으로 검색)
    char search_name[50];
    printf("========================================\n");
    printf("         [ 학생 검색 프로그램 ]\n");
    printf("========================================\n");
    printf("검색할 학생의 이름을 입력하세요: ");
    scanf("%s", search_name);

    // SQL 쿼리 동적 생성 (LIKE를 이용해 부분 검색 허용)
    char sql_select[256];
    sprintf(sql_select, "SELECT student_id AS '학번', name AS '이름', department AS '소속학과' FROM students WHERE name LIKE '%%%s%%';", search_name);

    printf("\n[검색 결과]\n");
    rc = sqlite3_exec(db, sql_select, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "검색 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
    }

    // 7. 데이터베이스 연결 종료
    sqlite3_close(db);
    return 0;
}
```


- 사진: <img width="1107" height="880" alt="스크린샷 2026-10-26 154953" src="https://github.com/user-attachments/assets/d590d451-9416-464a-b2bb-5a501d2e3b84" />

