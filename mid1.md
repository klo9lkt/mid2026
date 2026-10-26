# 문제1: 무지개 블록깨기 게임을 C언어로 작성하시오.

## 답안1:

- 소스코드
```

#include "raylib.h"
#include <math.h>
#include <stdbool.h>
#include <stdio.h>

#define SCREEN_WIDTH 900
#define SCREEN_HEIGHT 700

#define ROWS 6
#define COLS 10

#define BLOCK_WIDTH 76
#define BLOCK_HEIGHT 25
#define BLOCK_GAP 7

#define PADDLE_WIDTH 120
#define PADDLE_HEIGHT 16

#define BALL_RADIUS 9
#define MAX_LIVES 3

typedef enum {
    PLAYING,
    GAME_OVER,
    WIN
} GameState;

typedef struct {
    Rectangle rect;
    Color color;
    bool active;
} Block;

static Block blocks[ROWS][COLS];

static Vector2 ball;
static Vector2 ballVelocity;
static Rectangle paddle;

static int lives;
static int score;
static int remainingBlocks;

static GameState gameState;

// 무지개 색상
static Color rainbowColors[ROWS] = {
    {255, 65, 65, 255},    // 빨강
    {255, 145, 40, 255},   // 주황
    {255, 220, 55, 255},   // 노랑
    {65, 210, 100, 255},   // 초록
    {60, 145, 255, 255},   // 파랑
    {170, 90, 255, 255}    // 보라
};

void ResetGame(void)
{
    int totalWidth = COLS * BLOCK_WIDTH
                   + (COLS - 1) * BLOCK_GAP;

    int startX = (SCREEN_WIDTH - totalWidth) / 2;
    int startY = 100;

    // 블록 생성
    for (int row = 0; row < ROWS; row++)
    {
        for (int col = 0; col < COLS; col++)
        {
            blocks[row][col].rect = (Rectangle){
                startX + col * (BLOCK_WIDTH + BLOCK_GAP),
                startY + row * (BLOCK_HEIGHT + BLOCK_GAP),
                BLOCK_WIDTH,
                BLOCK_HEIGHT
            };

            blocks[row][col].color = rainbowColors[row];
            blocks[row][col].active = true;
        }
    }

    // 바 초기화
    paddle = (Rectangle){
        SCREEN_WIDTH / 2.0f - PADDLE_WIDTH / 2.0f,
        SCREEN_HEIGHT - 65,
        PADDLE_WIDTH,
        PADDLE_HEIGHT
    };

    // 공 초기화
    ball = (Vector2){
        SCREEN_WIDTH / 2.0f,
        SCREEN_HEIGHT - 100
    };

    ballVelocity = (Vector2){ 260.0f, -300.0f };

    lives = MAX_LIVES;
    score = 0;
    remainingBlocks = ROWS * COLS;

    gameState = PLAYING;
}

void UpdateGame(void)
{
    if (gameState != PLAYING)
        return;

    float dt = GetFrameTime();

    // 좌우 방향키로 바 이동
    if (IsKeyDown(KEY_LEFT))
        paddle.x -= 500.0f * dt;

    if (IsKeyDown(KEY_RIGHT))
        paddle.x += 500.0f * dt;

    // 화면 밖으로 나가지 않도록 제한
    if (paddle.x < 0)
        paddle.x = 0;

    if (paddle.x + paddle.width > SCREEN_WIDTH)
        paddle.x = SCREEN_WIDTH - paddle.width;

    // 공의 이전 위치 저장
    float previousX = ball.x;
    float previousY = ball.y;

    // 공 이동
    ball.x += ballVelocity.x * dt;
    ball.y += ballVelocity.y * dt;

    // 좌우 벽 충돌
    if (ball.x - BALL_RADIUS <= 0)
    {
        ball.x = BALL_RADIUS;
        ballVelocity.x = fabsf(ballVelocity.x);
    }

    if (ball.x + BALL_RADIUS >= SCREEN_WIDTH)
    {
        ball.x = SCREEN_WIDTH - BALL_RADIUS;
        ballVelocity.x = -fabsf(ballVelocity.x);
    }

    // 천장 충돌
    if (ball.y - BALL_RADIUS <= 0)
    {
        ball.y = BALL_RADIUS;
        ballVelocity.y = fabsf(ballVelocity.y);
    }

    // 바 충돌
    Rectangle ballRect = {
        ball.x - BALL_RADIUS,
        ball.y - BALL_RADIUS,
        BALL_RADIUS * 2,
        BALL_RADIUS * 2
    };

    if (ballVelocity.y > 0 &&
        CheckCollisionCircleRec(ball, BALL_RADIUS, paddle))
    {
        // 바의 어느 부분에 맞았는지에 따라 반사 각도 변경
        float paddleCenter = paddle.x + paddle.width / 2.0f;

        float hitPosition =
            (ball.x - paddleCenter) / (paddle.width / 2.0f);

        if (hitPosition < -1.0f) hitPosition = -1.0f;
        if (hitPosition > 1.0f) hitPosition = 1.0f;

        ball.y = paddle.y - BALL_RADIUS;

        ballVelocity.x = hitPosition * 380.0f;
        ballVelocity.y = -sqrtf(260.0f * 260.0f
                               + 300.0f * 300.0f);

        // 공 속도를 일정하게 유지
        float speed = sqrtf(
            ballVelocity.x * ballVelocity.x +
            ballVelocity.y * ballVelocity.y
        );

        ballVelocity.x *= 400.0f / speed;
        ballVelocity.y *= 400.0f / speed;
    }

    // 블록 충돌
    for (int row = 0; row < ROWS; row++)
    {
        for (int col = 0; col < COLS; col++)
        {
            Block *block = &blocks[row][col];

            if (!block->active)
                continue;

            if (CheckCollisionCircleRec(
                    ball, BALL_RADIUS, block->rect))
            {
                block->active = false;
                score += 10;
                remainingBlocks--;

                // 위/아래에서 충돌했는지 확인
                if (previousY + BALL_RADIUS <= block->rect.y ||
                    previousY - BALL_RADIUS >=
                    block->rect.y + block->rect.height)
                {
                    ballVelocity.y = -ballVelocity.y;
                }
                else
                {
                    ballVelocity.x = -ballVelocity.x;
                }

                // 한 번에 블록 하나만 제거
                row = ROWS;
                break;
            }
        }
    }

    // 모든 블록을 깨면 승리
    if (remainingBlocks <= 0)
    {
        gameState = WIN;
        return;
    }

    // 공을 놓치면 목숨 감소
    if (ball.y - BALL_RADIUS > SCREEN_HEIGHT)
    {
        lives--;

        if (lives <= 0)
        {
            gameState = GAME_OVER;
        }
        else
        {
            // 공과 바 위치 재설정
            paddle.x = SCREEN_WIDTH / 2.0f
                     - PADDLE_WIDTH / 2.0f;

            ball.x = SCREEN_WIDTH / 2.0f;
            ball.y = SCREEN_HEIGHT - 100;

            ballVelocity = (Vector2){ 260.0f, -300.0f };
        }
    }
}

void DrawGame(void)
{
    BeginDrawing();

    // 배경
    ClearBackground((Color){ 18, 22, 45, 255 });

    // 상단 UI
    DrawText("RAINBOW BREAKER", 25, 20, 27, RAYWHITE);

    DrawText(TextFormat("SCORE: %d", score),
             340, 25, 22, GOLD);

    DrawText(TextFormat("LIVES: %d", lives),
             710, 25, 22, RED);

    // 블록 그리기
    for (int row = 0; row < ROWS; row++)
    {
        for (int col = 0; col < COLS; col++)
        {
            if (!blocks[row][col].active)
                continue;

            Rectangle r = blocks[row][col].rect;

            DrawRectangleRounded(r, 0.25f, 6,
                                 blocks[row][col].color);

            // 블록 위쪽의 밝은 테두리
            DrawLineEx(
                (Vector2){ r.x + 5, r.y + 3 },
                (Vector2){ r.x + r.width - 5, r.y + 3 },
                2.0f,
                Fade(WHITE, 0.5f)
            );
        }
    }

    // 바 그리기
    DrawRectangleRounded(paddle, 0.7f, 8, SKYBLUE);

    // 공 그리기
    DrawCircleV(ball, BALL_RADIUS, RAYWHITE);
    DrawCircleLines((int)ball.x, (int)ball.y,
                    BALL_RADIUS, SKYBLUE);

    // 하단 안내
    DrawText("LEFT / RIGHT : MOVE",
             20, SCREEN_HEIGHT - 30, 16, LIGHTGRAY);

    // 게임 오버
    if (gameState == GAME_OVER)
    {
        DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT,
                      Fade(BLACK, 0.75f));

        DrawText("GAME OVER",
                 280, 260, 55, RED);

        DrawText(TextFormat("FINAL SCORE: %d", score),
                 330, 335, 25, RAYWHITE);

        DrawText("PRESS R TO RESTART",
                 300, 400, 22, GOLD);
    }

    // 승리
    if (gameState == WIN)
    {
        DrawRectangle(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT,
                      Fade(BLACK, 0.75f));

        DrawText("YOU WIN!",
                 330, 260, 55, GOLD);

        DrawText(TextFormat("FINAL SCORE: %d", score),
                 330, 335, 25, RAYWHITE);

        DrawText("PRESS R TO PLAY AGAIN",
                 275, 400, 22, SKYBLUE);
    }

    EndDrawing();
}

int main(void)
{
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT,
               "Rainbow Block Breaker - raylib");

    SetTargetFPS(60);

    ResetGame();

    while (!WindowShouldClose())
    {
        // R 키로 게임 재시작
        if (IsKeyPressed(KEY_R) &&
            gameState != PLAYING)
        {
            ResetGame();
        }

        UpdateGame();
        DrawGame();
    }

    CloseWindow();

    return 0;
}


```

- 사진
<img width="899" height="722" alt="스크린샷 2026-10-26 150954" src="https://github.com/user-attachments/assets/38852c1d-e2cf-4b3f-8779-103f775ce976" />
