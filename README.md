#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <cstdlib>
#include <conio.h> // for _kbhit and _getch
#include <chrono>

using namespace std;

const int width = 30;
const int height = 20;

bool gameOver;
int x, y, fruitX, fruitY, score;
vector<pair<int, int>> snake;
enum Direction { STOP = 0, LEFT, RIGHT, UP, DOWN };
Direction dir;

mutex mtx;

void Setup() {
    gameOver = false;
    dir = STOP;
    x = width / 2;
    y = height / 2;
    snake.clear();
    snake.push_back({x, y});
    fruitX = rand() % width;
    fruitY = rand() % height;
    score = 0;
}

void Draw() {
    system("cls"); // clear screen

    for (int i = 0; i < width + 2; i++) cout << "#";
    cout << endl;

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            if (j == 0) cout << "#"; // wall
            bool printed = false;
            mtx.lock();
            for (auto segment : snake) {
                if (segment.first == j && segment.second == i) {
                    cout << "O";
                    printed = true;
                }
            }
            mtx.unlock();

            if (fruitX == j && fruitY == i) {
                cout << "F";
                printed = true;
            }

            if (!printed) cout << " ";

            if (j == width - 1) cout << "#";
        }
        cout << endl;
    }

    for (int i = 0; i < width + 2; i++) cout << "#";
    cout << endl;

    cout << "Score: " << score << endl;
}

void Input() {
    if (_kbhit()) {
        switch (_getch()) {
        case 'a':
            dir = LEFT;
            break;
        case 'd':
            dir = RIGHT;
            break;
        case 'w':
            dir = UP;
            break;
        case 's':
            dir = DOWN;
            break;
        case 'x':
            gameOver = true;
            break;
        }
    }
}

void MoveSnake() {
    while (!gameOver) {
        mtx.lock();
        if (dir != STOP) {
            pair<int, int> prev = snake[0];
            pair<int, int> newHead = prev;

            switch (dir) {
            case LEFT:
                newHead.first--;
                break;
            case RIGHT:
                newHead.first++;
                break;
            case UP:
                newHead.second--;
                break;
            case DOWN:
                newHead.second++;
                break;
            }

            if (newHead.first < 0 || newHead.first >= width || newHead.second < 0 || newHead.second >= height)
                gameOver = true;

            for (auto s : snake) {
                if (s == newHead) {
                    gameOver = true;
                    break;
                }
            }

            snake.insert(snake.begin(), newHead);
            if (newHead.first == fruitX && newHead.second == fruitY) {
                score += 10;
                fruitX = rand() % width;
                fruitY = rand() % height;
            } else {
                snake.pop_back();
            }
        }
        mtx.unlock();
        this_thread::sleep_for(chrono::milliseconds(150));
    }
}

void FruitSpawner() {
    while (!gameOver) {
        this_thread::sleep_for(chrono::seconds(10)); // every 10 seconds
        mtx.lock();
        fruitX = rand() % width;
        fruitY = rand() % height;
        mtx.unlock();
    }
}

int main() {
    Setup();
    thread snakeThread(MoveSnake);
    thread fruitThread(FruitSpawner);

    while (!gameOver) {
        Draw();
        Input();
        this_thread::sleep_for(chrono::milliseconds(100));
    }

    snakeThread.join();
    fruitThread.join();

    cout << "Game Over! Final Score: " << score << endl;
    return 0;
}
