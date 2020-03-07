# Kotlin from Start to Somewhere
## Part 3

### What will we cover?
In this session, we'll put (almost) everything you've learned into making Conway's Game of Life!

Conway's Game of Life is an example of a _cellular automaton_, a grid of cells each with a finite number of states. In this model, cells represent a 'population', and cells can be either 'alive' or 'dead' based on rules that occur between iteration steps. 

- Any living cell with 2 or 3 living neighbors stays alive.
- Any dead cell with 3 living neigbors becomes alive.
- All other cells are dead. 

If you need help at any point, then raise your hand or ask a mentor - they'll be more than happy to help!

# Getting Started
First things first, lets get construct the skeleton that we'll use to build the game.

In an empty document, create your `main` function, and above that, an empty class called `Game`.

```kotlin
class Game() {

}

fun main() {

}

```

Our `Game` needs some parameters, one for width, one for height, and one for the percentage of the starting board that will be 'alive'. Let's make these member variables because they will get used beyond the constructor.

```kotlin
class Game(val width: Int = 20, val height: Int = 10, val percentageFilledStart: Int = 50) {

}
```

Now, let's write the functions that we need in our main function!
- `initGrid`, to set up the grid at the start
- `printGrid`, to print the current iteration of the grid to the console
- `updateGrid`, to iterate the grid to the next step.

We can leave them blank for now!

```kotlin
class Game(val width: Int = 20, val height: Int = 10, val percentageFilledStart: Int = 50) {
    fun initGrid() {
    }
    
    fun printGrid() {
    }
    
    fun updateGrid() {
    }
}

fun main() {
    val game = Game()
    game.initGrid()
    game.printGrid()
    for (i in 0 until 20) {
        game.updateGrid()
        game.printGrid()
    }
}
```

Now let's start filling these in!

`initGrid` requires a random number generator, so we can randomly decide if a cell should be alive or not at the start. Thankfully, Java has one we can steal, so just shove `import java.util.Random` right at the top of the file. For now, we need to create two _member variables_ of our `Game` class: `rand`, the random number generator instance, and `grid`, the grid of characters representing the game board:

```kotlin
class Game(val width: Int = 20, val height: Int = 10, val percentageFilledStart: Int = 50) {
    val rand = Random()
    var grid = List<List<Char>>(height) { " ".repeat(width).toList() }
...
}
...
```

Now we can define our function! We use the `map` function over a collection to map an input to an output (in this case, old row to new row, and old character to new character after the random number generation).

Because having multiple `it`s can be confusing, we explicitly name the input parameters of these lambdas to `row` and `char`.

```kotlin
    fun initGrid() {
        grid = grid.map { row -> row.map { char -> if (rand.nextInt() % 100 <= percentageFilledStart) ' ' else '#' } }
    }
```

Now `printGrid` is simple enough. We want to print the iteration count for our reference, and then print the grid row by row. We can achieve that using a simple for loop and a helper function `joinToString`. We also need a new member variable `iter`, which is the iteration counter.
```kotlin
    var iter = 0

    fun printGrid() {
        println("Iteration $iter")
    	for (str in grid) println(str.joinToString(separator="")) // Separator empty string, otherwise would be comma.
        println("") // New line for clarity
    }
```

## Progress Update

At this point you should have something like this:

```kotlin
import java.util.Random
class Game(val width: Int = 20, val height: Int = 10, val percentageFilledStart: Int = 50) {
    val rand = Random()
    var grid = List<List<Char>>(height) { " ".repeat(width).toList() }
    var iter = 0
    
    fun initGrid() {
        grid = grid.map { row -> row.map { char -> if (rand.nextInt() % 100 <= percentageFilledStart) ' ' else '#' } }
    }
    
    fun printGrid() {
        println("Iteration $iter")
    	for (str in grid) println(str.joinToString(separator=""))
        println("")
    }
}

fun main() {
    val game = Game()
    game.initGrid()
    game.printGrid()
    for (i in 0 until 20) {
        game.updateGrid()
        game.printGrid()
    }
}
```

If you now run this, we should just see a random grid being printed a bunch of times... If you want you can put a call to `initGrid` in `updateGrid` but that just resets the grid... We need to perform the iteration step!

We want to make three class helper functions that will assist us on our mission, `isAlive` and `countAliveNeighbors`

- `isAlive` will tell us if the given cell is alive or not
- `countAliveNeighbors` will tell us how many neighbors are alive.
- `updateCell` will return the new alive/dead value of any given cell.

First, let's work on `isAlive`. This can be a simple expression function, that determines if the current cell is alive or dead with an `if` expression (also some simple bounds checking, as it simplifies our code later).

```kotlin
    fun isAlive(row: Int, col: Int) = if (row in 0 until height && col in 0 until width && grid[row][col] == '#') 1 else 0
```

O represents a dead cell, and 1 represents an alive cell.

Now, we can write `countAliveNeighbors`, as this uses `isAlive`. This one is complicated, so don't worry if you don't get what this is doing on first glance:

```kotlin
    fun countAliveNeighbors(row: Int, col: Int) = listOf(-1 to -1, -1 to 0, -1 to 1, 0 to -1, 0 to 1, 1 to -1, 1 to 0, 1 to 1)
            .map { isAlive(it.first + row, it.second + col) }
            .sum()
```
First, we construct a list of all of the 'coordinate displacements' for each neighbor of the main cell (which in this system would be 0,0). These are essentially relative coordinates. We use the `to` keyword to construct a `Pair`, which lets us do `.first` and `.second` to get the first and second values.

From this list, we call `map` to convert these displacements into their `isAlive()` values, by adding the relative coordinate to the coordinate of the current cell, to get the neighbor cell's coordinates in global space. We then pass these coordinates to `isAlive`, so our list becomes a list of 1s and 0s, representing alive or dead cells. We then `sum` these values to get the number of alive neighbor cells.

Finally, we want `updateCell`.
```kotlin
    fun updateCell(row: Int, col: Int, current: Char) = countAliveNeighbors(row, col).let { // it ->
        if (current == '#') {
                if (it in 2..3) '#' else ' '
            } else {
                if (it == 3) '#' else ' '
            }
    }   
```

This one is again a bit complicated. We need to call the function `countAliveNeighbors()`, but doing that requires multiple lines, and means we can't use our lovely one expression functions. So we use the `let` extension function on our output of `countAliveNeighbors()`, which if you remember, means that the resulting value from the function call only exists in that scope, as the lambda variable (by default called `it`). The final expression in that block is then returned, which means the function takes the value of the `if` expression. The `if` expression checks if we are alive currently, and then sees if we can stay alive, otherwise it checks if we should _become_ alive. These are all based on the rules above, so I won't go too much into those.

## Progress Update

At this point we're almost done, and you should have something like this:
```kotlin
import java.util.Random

class Game(val width: Int = 20, val height: Int = 10, val percentageFilledStart: Int = 50) {
    val rand = Random()
    var grid = List<List<Char>>(height) { " ".repeat(width).toList() }
    var iter = 0
    
    fun initGrid() {
        grid = grid.map { row -> row.map { char -> if (rand.nextInt() % 100 <= percentageFilledStart) ' ' else '#' } }
    }
    
    fun printGrid() {
        println("Iteration $iter")
    	for (str in grid) println(str.joinToString(separator=""))
        println("")
    }
    
    fun updateGrid() {
        //Todo
    }
    
    fun updateCell(row: Int, col: Int, current: Char) = countAliveNeighbors(row, col).let {
        if (current == '#') {
                if (it in 2..3) '#' else ' '
            } else {
                if (it == 3) '#' else ' '
            }
    }    
    fun countAliveNeighbors(row: Int, col: Int) = listOf(-1 to -1, -1 to 0, -1 to 1, 0 to -1, 0 to 1, 1 to -1, 1 to 0, 1 to 1)
            .map { isAlive(it.first + row, it.second + col) }
            .sum()
    
    fun isAlive(row: Int, col: Int) = if (row in 0 until height && col in 0 until width && grid[row][col] == '#') 1 else 0
}

fun main() {
    val game = Game()
    game.initGrid()
    game.printGrid()
    for (i in 0 until 20) {
        game.updateGrid()
        game.printGrid()
    }
}
```

Lastly we now need to implement or `updateGrid()` function. Now, similar to `map`, `mapIndexed` is another function that operates on containers, that maps values to other values based on a lambda. However, this lambda also has a parameter for the index! This is incredibly useful as it allows us to pass our `rowIndex` and `colIndex` into `updateCell()`.

The code below is similar to `initGrid()` in that we map over each row, and then for each row we map over each char, but in this case we use `mapIndexed`, and our new cell value comes from `updateCell()`.

```kotlin
    fun updateGrid() {
        grid = grid.mapIndexed { 
            rowIndex, row -> row.mapIndexed { 
            	colIndex, char -> updateCell(rowIndex, colIndex, char)
        	} 
        }
        iter++
    }
```

And that's it. If you run the script, you should see 20 iterations of Conway's Game of Life!

# Possible Improvements
- Better initial grid setup.
- Caching that displacement list of pairs.
- Actually having it work in a real terminal so you can see it change in real time!
- Whatever else you can think of!
# Most important tip of all
- Don't be afraid if you don't know what something is!
- This was not all-encompassing, Kotlin is a *big* language.

If you come across a piece of syntax that you don't understand, google it! The internet is an amazing resource for programmers, don't expect to memorise everything you've learned today!