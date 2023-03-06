<script lang=ts>
    function getRandomColor() {
        let stringChars = ['0','1','2','3','4','5','6','7','8','9','0','a','b','c','d','e','f']
        let colorString = '#';
        for (let i = 0; i < 6; i++) {
            colorString += stringChars[Math.floor(Math.random() * stringChars.length)];
        }
        return colorString;
    }

    let grid: any[] = [];
    let brushColor: string = '#000000';
    let size: number = 16;
    for (let i = 0; i < size; i++) {
        let row = [];
        for (let j = 0; j < size; j++) {
            row.push({ color: 'white'})
        }
        grid.push(row);
    }

    $: containerStyle = `grid-template-columns: repeat(${size}, 1fr); grid-template-rows: repeat(${size}, 1fr);`
    $: down = false;
    $: showGrid = true;
</script>

<svelte:window 
    on:mousedown={(e) => {e.preventDefault(); down = true}}
    on:mouseup={() => down = false}
    on:touchstart={(e) => {e.stopPropagation(); down = true;}}
    on:touchend={()=> down = false}
    on:touchmove={(e) => {
        let x = e.touches[0].clientX;
        let y = e.touches[0].clientY;

        let copy = [...grid];
                let el = copy[0][0].el;
                let { top, bottom, left, right } = copy[0][0].el.getBoundingClientRect();
                el.getBoundingClientRect().top;
                for (let i = 0; i < copy.length; i++) {
                    for (let j = 0; j < copy[i].length; j++) {
                        let {top, bottom, left, right } = copy[i][j].el.getBoundingClientRect();
                if (y < bottom && y > top && x > left && x < right) {
                            if (down) copy[i][j].color = brushColor;
                        }
                    }
                 }
                grid = copy;
    }}
    />
<div class=page style={containerStyle}>
    {#each grid as row, i}
        {#each row as box, j}
        <div 
            style="background-color: {box.color}"
            class={showGrid ? 'gridLines' : ''}
            id='row{i}box{j}'
            bind:this={box.el}
            on:mousedown={() => box.color = brushColor}
            on:touchstart={() => box.color = brushColor}
            on:mouseenter={() => {
                if (down) {
                    box.color = brushColor;
                }
            }}
            >
        </div>
        {/each}
    {/each}
</div>
<div class=toolbar>
    <div>
        <span style="color: white;">Color: </span>
        <input type=color bind:value={brushColor}>
    </div>
    <div>
        <button
            on:click={() => {
                grid = grid.map(row => {
                    let newRow = row.map(box => {
                        return { color: 'white'}
                    });
                    return newRow;
                })
            }}
        >Reset</button>
        <button
            on:click={() => {
                showGrid = !showGrid
            }}
        >
            Show Grid
        </button>
        <button
        on:click={() => {
        
            grid = grid.map(row => {
                return row.map(box => {
                    return { color: getRandomColor()}
                })
            })
        }}
        >Random</button>
    </div>
    <div>
        <span style="color: white;">Grid size: {size}</span>
        <button on:click={() => {
            grid = grid.map(row => [...row, { color: 'white'}]);
            let newRow = [];
            for (let i = 0; i < size +1; i++) {
                newRow.push({ color: 'white'})
            }
            grid = [...grid, newRow]
            size++
            }}>up</button>
        <button on:click={() => {
            grid = grid.map(row => row.slice(0, -1))
            grid=grid.slice(0, -1)
            size--
            }}>down</button>
    </div>
</div>
<style>
    :global(body, html) {
        background-color: black;
        height: 100%;
        width: 100%;
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    .page {
        background-color: white;
        height: 90%;
        width: 100%;
        display: grid;
    }

    .toolbar {
        height: 10%;
        display: flex;
        justify-content: space-evenly;
        align-items: center;
    }

    .gridLines {
        outline: 1px solid black;
    }
</style>