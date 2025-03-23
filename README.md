#include <iostream>
#include <cstdlib>
#include <ctime>

using namespace std;

class Maze {
public:
    Maze(int width, int height, int difficulty);
    ~Maze(); // Destructor to free memory
    void printMaze();
    bool movePlayer(char direction);
    bool isExit(int x, int y);
    void startGame();
    
private:
    int width, height;
    int playerX, playerY;
    int exitX, exitY;
    char **maze;
    int difficulty; // Difficulty level to control the wall density
    
    void generateMaze();
};

Maze::Maze(int w, int h, int diff) {
    width = w;
    height = h;
    playerX = 1; // Start position
    playerY = 1;
    difficulty = diff;
    maze = new char*[height];
    for (int i = 0; i < height; ++i) {
        maze[i] = new char[width];
    }
    generateMaze();
}

Maze::~Maze() {
    for (int i = 0; i < height; ++i) {
        delete[] maze[i]; // Free each row
    }
    delete[] maze; // Free the maze itself
}

void Maze::generateMaze() {
    srand(time(0));  // Seed for randomness

    // Fill maze with walls
    for (int i = 0; i < height; ++i) {
        for (int j = 0; j < width; ++j) {
            if (i == 0 || j == 0 || i == height - 1 || j == width - 1) {
                maze[i][j] = '#'; // Walls around the border
            } else {
                maze[i][j] = (rand() % difficulty == 0) ? '#' : '.'; // Adjust wall density based on difficulty
            }
        }
    }

    // Ensure starting position is not on a wall
    maze[playerY][playerX] = '.';

    // Place the exit (random location within the maze)
    do {
        exitX = rand() % (width - 2) + 1;
        exitY = rand() % (height - 2) + 1;
    } while (maze[exitY][exitX] == '#'); // Ensure the exit is not in a wall
    maze[exitY][exitX] = 'E'; // Mark the exit
}

void Maze::printMaze() {
    for (int i = 0; i < height; ++i) {
        for (int j = 0; j < width; ++j) {
            if (i == playerY && j == playerX) {
                cout << 'P';  // Player is marked as P
            } else {
                cout << maze[i][j];
            }
        }
        cout << endl;
    }
}

bool Maze::movePlayer(char direction) {
    int newX = playerX;
    int newY = playerY;

    if (direction == 'w') { newY--; } // Move up
    else if (direction == 's') { newY++; } // Move down
    else if (direction == 'a') { newX--; } // Move left
    else if (direction == 'd') { newX++; } // Move right

    // Check if the move is within bounds and not a wall
    if (newX >= 1 && newX < width - 1 && newY >= 1 && newY < height - 1 && maze[newY][newX] != '#') {
        playerX = newX;
        playerY = newY;
    } else {
        cout << "Invalid move! You can't walk into a wall or out of bounds." << endl;
        return false; // Invalid move (hit a wall)
    }

    return true;
}

bool Maze::isExit(int x, int y) {
    return (x == exitX && y == exitY);
}

void Maze::startGame() {
    char move;
    bool gameOver = false;

    // Game instructions
    cout << "\nWelcome to Maze Explorer!" << endl;
    cout << "Find the exit (E) while avoiding walls (#)." << endl;
    cout << "Use the following keys to move: " << endl;
    cout << "W - Up\nA - Left\nS - Down\nD - Right\nQ - Quit" << endl;
    
    while (!gameOver) {
        printMaze();
        cout << "\nEnter your move: ";
        cin >> move;

        // Validate the move
        if (move != 'w' && move != 'a' && move != 's' && move != 'd' && move != 'q') {
            cout << "Invalid input! Please enter W, A, S, D, or Q." << endl;
            continue;
        }

        if (move == 'q') {
            cout << "You quit the game." << endl;
            break;
        }

        if (movePlayer(move)) {
            if (isExit(playerX, playerY)) {
                cout << "Congratulations! You reached the exit!" << endl;
                gameOver = true;
            }
        }
    }
}

int main() {
    int width, height, difficulty;
    cout << "Enter maze width (5-20): ";
    cin >> width;
    cout << "Enter maze height (5-20): ";
    cin >> height;

    // Ensure the maze is large enough to have walls
    if (width < 5 || height < 5) {
        cout << "Maze dimensions are too small. Minimum size is 5x5." << endl;
        return 1;
    }

    cout << "Select difficulty level (1 - Easy, 2 - Medium, 3 - Hard): ";
    cin >> difficulty;

    // Ensure difficulty is within a valid range
    if (difficulty < 1 || difficulty > 3) {
        cout << "Invalid difficulty level. Setting to Medium (2)." << endl;
        difficulty = 2;
    }

    // Difficulty levels: lower values mean fewer walls
    int wallDensity = (difficulty == 1) ? 5 : (difficulty == 2) ? 3 : 2;

    Maze game(width, height, wallDensity);
    game.startGame();
    return 0;
}
