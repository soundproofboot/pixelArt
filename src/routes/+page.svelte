<script lang=ts>
    import { onMount } from "svelte";

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
    let colorPicker: HTMLElement;
    $: colorPickerStyle = '';

    onMount(() => {
        let buttonEl = document.querySelector('button');
        let dimensions = buttonEl?.getBoundingClientRect();
        colorPickerStyle = `width: ${dimensions?.width}px; height: ${dimensions?.height}px`;
    })
</script>

<svelte:window 
    on:mousedown={(e) => {
        if (e.target !== colorPicker) {
            e.preventDefault();
        }
        down = true
        }}
    on:mouseup={() => down = false}
    on:touchstart={(e) => {e.stopPropagation(); down = true;}}
    on:touchend={()=> down = false}
    on:touchmove={(e) => {
        let x = e.touches[0].clientX;
        let y = e.touches[0].clientY;

        let copy = [...grid];
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
<div style="height: 100%; display: flex; flex-direction: column; justify-content: space-evenly;">
    <div class=canvas style={containerStyle}>
        {#each grid as row, i}
        {#each row as box, j}
        <div 
        style="background-color: {box.color};"
        class=gridLines
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
    <div class=toolbar >
        <input 
            in:fade={{duration: 1000}}
            type=color 
            bind:value={brushColor} 
            bind:this={colorPicker}
            style={colorPickerStyle}
        >
        <button
        class=resetButton
        on:click={() => {
            grid = grid.map(row => {
                let newRow = row.map(box => {
                    return { color: 'white'}
                });
                return newRow;
            })
        }}
        >Reset</button>
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
    
    :global(body) {
        overflow: hidden;
        position: fixed;
    }

    .canvas {
        background-color: white;
        display: grid;
        margin: 1em auto;
        outline: 5px solid gray;
        border-radius: 5px;
    }

    @media (orientation: landscape) {
        .canvas {
           height: 80vh;
           width: 80vh;
        }

        .resetButton {
            height: 50%;
            aspect-ratio: 1/1;
        }
    }

    @media (orientation: portrait) {
        .canvas {
           height: 90vw;
           width: 90vw;
        }

        .resetButton {
            width: 20vw;
            aspect-ratio: 1/1;
        }
    }

    .toolbar {
        display: flex;
        justify-content: center;
        align-items: center;
        column-gap: 2em;
        flex-grow: 1;
    }

    .resetButton {
        background-color: transparent;
        border: none;
        color: white;
        outline: 2px solid white;
        border-radius: 5px;
    }

    .gridLines {
        outline: 1px solid black;
    }
</style>