<script lang="ts">
import { onMount } from "svelte";

    // binds
    let container: HTMLDivElement

    // position
    type SnakePosition = number[] // [x, y]
    let snake: SnakePosition[] = [[50, 50], [50, 50], [50,50]]
    const MIN_POSITION = 0
    const GAME_SIZE = 100

    // velocity
    type SnakeVelocity = SnakePosition
    let velocity: SnakeVelocity = [0, 0]
    function goX(x: number) { velocity = [x, 0] }
    function goY(y: number) { velocity = [0, y] }

    const Go = {
        left: () => goX(-1),
        right: () => goX(+1),
        up: () => goY(-1),
        down: () => goY(+1),
    }

    // for adding
    function wrap(num: number, min: number, max: number) {
        // return Math.min(Math.max(num, min), max);
        return num >= max ? min + (num - max) : (
            num < min ? max - (min - num) : num
        )
    }
    function wrapPosition(num: number) {
        return wrap(num, MIN_POSITION, MIN_POSITION + GAME_SIZE);
    }

    // food
    function randomIntInclusive(min: number, max: number) { // min and max included 
        return Math.floor(Math.random() * (max - min + 1) + min)
    }
    function randomFood() {
        return [
            randomIntInclusive(MIN_POSITION, MIN_POSITION + GAME_SIZE - 1),
            randomIntInclusive(MIN_POSITION, MIN_POSITION + GAME_SIZE - 1),
        ]
    }
    let food: SnakePosition = randomFood()

    // start the game
    onMount(() => {
        document.addEventListener("keydown", (ev) => {
            if (ev.code == "KeyD" || ev.code == "ArrowRight") {
                Go.right()
            }
            if (ev.code == "KeyA" || ev.code == "ArrowLeft") {
                Go.left()
            }
            if (ev.code == "KeyW" || ev.code == "ArrowUp") {
                Go.up()
            }
            if (ev.code == "KeyS" || ev.code == "ArrowDown") {
                Go.down()
            }
        })

        // game loop
        setInterval(() => {
            const nextPos = [
                wrapPosition(snake[0][0] + velocity[0]),
                wrapPosition(snake[0][1] + velocity[1]),
            ]

            // food update
            if (nextPos[0] == food[0] && 
                nextPos[1] == food[1])
            {
                snake[snake.length] = []
                // add a new entry at the end.
                // Loop below will populate the position automatically.

                food = randomFood()
            }

            // position update
            for (let i = snake.length - 1; i >= 1; i--) {
                snake[i][0] = snake[i-1][0]
                snake[i][1] = snake[i-1][1]
            }
            snake[0][0] = nextPos[0]
            snake[0][1] = nextPos[1]
        }, 100)
    })
</script>

<div bind:this={container} class="container">
    <div class="food" style={
    `left: ${10*food[0]}px; top:${10*food[1]}px;`
    }></div>
    {#each snake as snakePart}
        <div class="snakePart" style={
        `left: ${10*snakePart[0]}px; top:${10*snakePart[1]}px;`
        }></div>
    {/each}
</div>

<style>
    :global(body) {
        background: #666;
    }
    .container {
        position: relative;
        background: black;
        width: 1000px;
        height: 1000px;
    }
    .snakePart, .food {
        position: absolute;
        background: white;
        width: 10px;
        height: 10px;
    }
    .food {
        background: yellowgreen;
    }
</style>
