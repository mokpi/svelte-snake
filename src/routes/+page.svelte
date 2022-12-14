<script lang="ts">
    import { onMount } from "svelte"

    // position
    type SnakePosition = number[] // [x, y]
    let snake: SnakePosition[] = [[50, 50], [50, 50], [50,50]]
    const MIN_POSITION = 0
    const GAME_SIZE = 20

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

    // events
    const onKeyDown = (ev: KeyboardEvent) => {
        if (ev.code == "KeyD" || ev.code == "ArrowRight") {
            if (velocity[0] < 0) return
            Go.right()
        }
        else if (ev.code == "KeyA" || ev.code == "ArrowLeft") {
            if (velocity[0] > 0) return
            Go.left()
        }
        else if (ev.code == "KeyW" || ev.code == "ArrowUp") {
            if (velocity[1] > 0) return
            Go.up()
        }
        else if (ev.code == "KeyS" || ev.code == "ArrowDown") {
            if (velocity[1] < 0) return
            Go.down()
        }
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

    // speed
    function pow2(n: number) { return 2 << (n-1); }
    const BASE_TIME = 1000 // ms
    let speed = 3
    $: speed = speed > 0 ? speed : 1
    $: updateTime = BASE_TIME / pow2(speed)

    // game loop
    const gameLoop = () => {
        const nextPos = [
            wrapPosition(snake[0][0] + velocity[0]),
            wrapPosition(snake[0][1] + velocity[1]),
        ]

        const isMoving = velocity[0] != 0 || velocity[1] != 0

        // collision detection
        if (isMoving) {
            for (let i = 0; i < snake.length; i++) {
                if (nextPos[0] == snake[i][0] &&
                    nextPos[1] == snake[i][1])
                {
                    document.removeEventListener("keydown", onKeyDown)
                    return // no updates and no setTimeout
                }
            }
        }

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

        // loop
        setTimeout(gameLoop, updateTime)
    }

    // start the game
    onMount(() => {
        document.addEventListener("keydown", onKeyDown)
        gameLoop()
    })
</script>

<div>
    <div>Press WASD or Arrow keys to start the game.</div>
    <div>If the game froze, a collision has happened and game over.</div>
</div>

<div class="container" style={
`width: ${10*GAME_SIZE}px; height:${10*GAME_SIZE}px;`
}>
    <div class="food" style={
    `left: ${10*food[0]}px; top:${10*food[1]}px;`
    }></div>
    {#each snake as snakePart}
        <div class="snakePart" style={
        `left: ${10*snakePart[0]}px; top:${10*snakePart[1]}px;`
        }></div>
    {/each}
</div>

<div>
    <div>Head Position: {snake[0][0]}, {snake[0][1]}</div>
    <div>Length (points): {snake.length}</div>
    <div>Speed:
        <button on:click={() => speed--}>-</button>
        {speed}
        <button on:click={() => speed++}>+</button>
        (update every {updateTime} ms)
    </div>
</div>

<style>
    :global(body) {
        background: #333;
        color: white;
        font-size: 1.2em;
        line-height: 150%;
        padding: 20px;
    }
    .container {
        margin: 20px;
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
        border-radius: 50%;
        box-shadow: 0 0 10px white;
    }
    .food {
        background: yellowgreen;
        box-shadow: 0 0 10px yellowgreen;
    }
</style>
