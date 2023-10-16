# [Snake game solution](https://frontendeval.com/questions/snake)

## [Stackblitz playground](https://stackblitz.com/github/willnguyen1312/snake-game)

### Constants

- `SIZE` - 15 - the size of the board
- `SPEED` - 200 milliseconds - the time between each move

### State

- snakeCells: `string[]` - the cells that the snake is currently occupying
- appleCell: `string` - the cell that the apple is currently occupying
- direction: `string` - the direction that the snake is moving, can be `up`, `down`, `left`, `right`
- gameState: `string` - the state of the game, can be `idle`, `playing`, `gameover`

Note: the cells are represented by the string of the row and column, e.g. `1-1` is the top left cell. This eases the calculation of the next cell when the snake moves.

### Rendering

The board is generated using a nested for loops with a size of 15. The board has 15 rows and 15 columns as per requirement. Each row is styled by a simple flexbox to lay out its cells horizontally.

As the game starts, the snake will move to the right and the apple will be randomly generated on the board after being eaten by the snake. The snake move is controlled by WASD and arrow keys. The snake will die if it hits the wall or itself. The game will be reset if the snake dies.

The snake and apple cells are rendered using the `snake` and `apple` classes respectively.

### Snake game logic

- `checkSnakeCell`: As our app is re-rendering upon snake move, we can check whether a cell is occupied by the snake by checking if a cell is in the `snakeCells` array.

- `checkAppleCell`: Similar to `checkSnakeCell`, we can check whether a cell is occupied by the apple by checking if a cell is equal to the `appleCell`.

- `handleKeyDown`: This function handles the keydown event and updates the snake direction accordingly. The snake direction is updated only if the key pressed is valid and the snake is not moving in the opposite direction.

- `getNewHead`: This function calculates the new head based on the current head and the direction. The new head is calculated by adding the current head with the current direction vector.

- `checkValidNewHead`: This function checks if the new head is valid. The new head is valid if it is not occupied by the snake and it is not out of bound.

- `generateNextApple`: This function generates the next apple cell by randomly generating a row and column and checking if the cell is not occupied by the snake.

- `moveSnake`: This function moves the snake to the next cell based on the current direction. It makes use of the `getNewHead` and `checkValidNewHead` functions. The snake will die if it hits the wall or itself. The game states will be reset if the snake dies. If the new cell is valid, add new head cell to the snake cells. If the new head is same as the apple cell, update the current apple cell with the new one by calling `generateNextApple`. Otherwise, remove the tail cell.

- `intervalId`: The interval is set up by using `setInterval` to call `moveSnake` every `SPEED` milliseconds. As the game state changes, this will be cleared to start a new section. It will also be cleaned up on the unmount lifecycle to avoid memory leaks.

### Nice to have

- [ ] Add test coverage
- [ ] Add more configuration for speed, size, pause, etc.
