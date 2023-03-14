<script lang=ts>
    import { onMount } from "svelte";
    import { fly, fade } from 'svelte/transition';
    let grid: any[] = [];
    let brushColor: string = '#000000';
    let size: number = 16;
    let sizeArray = [4,8,16,32];
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

    let showModal = false;

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
            type=color 
            bind:value={brushColor} 
            bind:this={colorPicker}
            style={colorPickerStyle}
        >
        <button
        class=newButton
        on:click={() => {
            showModal = true;
        }}
        >New</button>
    </div>
</div>
{#if showModal}
<div class=modal transition:fade>
    <div class=modalContent transition:fly={{y: 500}}>
        <button class=closeButton on:click={() => showModal = false}>X</button>
        <div class=gridButtonContainer>

            {#each sizeArray as gridSize}
                <button
                    class=gridButton
                    on:click={() => {
                        let newGrid = [];
                        for (let i = 0; i < gridSize; i++) {
                            let row = [];
                            for (let j = 0; j < gridSize; j++) {
                                row.push({ color: 'white'})
                            }
                            newGrid.push(row);
                        }
                        grid = newGrid;
                        size = gridSize;
                        showModal = false;
                }}>{gridSize}x</button>
            {/each}
        </div>
    </div>
</div>
{/if}

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

        .newButton {
            height: 50%;
            aspect-ratio: 1/1;
        }
    }

    @media (orientation: portrait) {
        .canvas {
           height: 90vw;
           width: 90vw;
        }

        .newButton {
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

    input[type=color] {
        transition: .5s;
    }

    button {
        border: none;
        background-color: black;
        color: white;
        border-radius: 5px;
        outline: 2px solid white;
        color: white;
    }

    .newButton {
        background-color: transparent;
        min-width: 3em;
    }
    
    .gridButton {
        padding: 1em;
        margin: 0 1em 1em;
        min-width: 4em;
    }
    
    .gridButtonContainer {
        display: flex;
        flex-wrap: wrap;
        /* row-gap: 1em; */
        margin-top: 3em;
        justify-content: space-evenly;

    }

    .gridLines {
        outline: 1px solid black;
    }

    .modal {
        height: 100%;
        width: 100%;
        background-color: rgb(0,0,0,.6);
        position: absolute;
        top:0;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .modalContent {
        background-color: white;
        display: flex;
        flex-wrap: wrap;
        justify-content: space-evenly;
        flex-direction: row;
        border-radius: .5em;
        position: relative;
    }

    .closeButton {
        color: black;
        width: fit-content;
        border: none;
        outline: none;
        background-color: transparent;
        border-radius: 1em;
        position: absolute;
        right: 10px;
        top: 10px;
    }
</style>