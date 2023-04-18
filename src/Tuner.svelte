<script>
    import { onMount } from "svelte";
    import { fade, fly, scale } from "svelte/transition";

    export let instrument;
    export let callback;

    let bar;
    let Frequency = 0;
    let Note = "--";
    let Octave = "-";

    let Stream;

    let MeterElem;
    let MeterPointerElem;

    let isListening = false;

    function start() {}

    function stop() {
        isListening = false;
        Stream?.getTracks()?.forEach((track) => track.stop());
    }


    onMount(() => {
        history.pushState({}, '', '');
        history.pushState({}, '', '');
        history.pushState({}, '', '');
        history.pushState({}, '', '');
        history.pushState({}, '', '');
        history.pushState({}, '', '');
        console.log('Pushed state');
        window.onpopstate = ()=>{
            history.forward();
            callback();
        };
    });
</script>

<div class="container">
    <div class="instrument">{instrument}</div>
    <div class="tuner">
        <div class="note">
            <div class="normal">
                <div class="meter" bind:this={MeterElem}>
                    <div class="noteName">
                        {Note}{Octave} | {Math.round(Frequency)}Hz
                    </div>
                    <div class="meter-dot" />
                    <div class="meter-pointer" bind:this={MeterPointerElem}></div>
                    <div class="range">
                        <div class="meter-scale meter-scale-strong" in:fly={{ y: 10, delay: 100 }}>♭</div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 150 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 200 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 250 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 300 }}></div>
                        <div class="meter-scale meter-scale-strong" in:fly={{ y: 10, delay: 350 }}>♮</div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 400 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 450 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 500 }}></div>
                        <div class="meter-scale" in:fly={{ y: 10, delay: 550 }}></div>
                        <div class="meter-scale meter-scale-strong" in:fly={{ y: 10, delay: 600 }}>♯</div>
                    </div>

                </div>
            </div>
        </div>
        {#if isListening}
            <button class="listenActionButton" on:click={stop}>Stop</button>
        {:else}
            <button class="listenActionButton" on:click={start}>Start</button>
        {/if}
    </div>
    <div class="barContainer">
        <div class="bar" bind:this={bar} />
    </div>
</div>

<style>



    .back{
        position: fixed;
        top: 0;
        left: 0;
        padding: 10px;
        cursor: pointer;
    }
    .meter {
        position: fixed;
        left: 0;
        right: 0;
        top: 50%;
        transform: translateY(-50%);
        width: 400px;
        height: 33%;
    }

    .meter-scale {
        width: 1px;
        height: 100%;
        transform-origin: bottom;
        transition: transform 0.2s;
        box-sizing: border-box;
        border-top: 10px solid;
        position: absolute;
        right: 50%;
    }

    .meter-scale:nth-child(1) {
        transform: rotate(-45deg);
    }

    .meter-scale:nth-child(2) {
        transform: rotate(-36deg);
    }

    .meter-scale:nth-child(3) {
        transform: rotate(-27deg);
    }

    .meter-scale:nth-child(4) {
        transform: rotate(-18deg);
    }

    .meter-scale:nth-child(5) {
        transform: rotate(-9deg);
    }

    .meter-scale:nth-child(6) {
        transform: rotate(0deg);
    }

    .meter-scale:nth-child(7) {
        transform: rotate(9deg);
    }

    .meter-scale:nth-child(8) {
        transform: rotate(18deg);
    }

    .meter-scale:nth-child(9) {
        transform: rotate(27deg);
    }

    .meter-scale:nth-child(10) {
        transform: rotate(36deg);
    }

    .meter-scale:nth-child(11) {
        transform: rotate(45deg);
    }

    .meter-scale-strong{
        width: 2px;
        border-top-width: 20px;
    }

    .meter-dot {
        width: 10px;
        height: 10px;
        background: #2c3e50;
        border-radius: 50%;
        position: absolute;
        bottom: -5px;
        right: 50%;
        margin-right: -4px;
    }

    .meter-pointer {
        width: 2px;
        height: 100%;
        background: #2c3e50;
        transform: rotate(45deg);
        transform-origin: bottom;
        transition: transform 0.5s;
        position: absolute;
        right: 50%;
    }

    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        width: min(100vw, 500px);
    }

    .barContainer {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 5px;
        height: 100%;
        background: #ffffff00;
    }

    .bar {
        position: absolute;
        bottom: 0;
        width: 100%;
        height: 0;
        background: #2196f3;
        transition: 50ms;
    }

    .listenActionButton {
        border: none;
        outline: none;
        padding: 10px;
        border-radius: 10px;
        background: #266dfa;
        cursor: pointer;
        transition: 100ms;
    }

    .listenActionButton:hover {
        filter: brightness(0.97);
    }

    .instrument {
        font-size: 1.4rem;
        color: #fff;
        font-weight: 700;
        margin-bottom: 20px;
        padding: 10px;
        width: 100%;
        text-align: right;
    }

    .tuner {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-end;
        width: 100%;
        height: 100%;
        gap: 20px;
        padding-bottom: 50px;
    }

    .note {
        font-size: 1.4rem;
        font-weight: 700;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        width: 100%;
        padding: 50px;
    }

    .noteName {
        font-size: 1.4rem;
        color: #fff;
        font-weight: 700;
        position: absolute;
        bottom: 0;
        left: 50%;
        transform: translate(-50%, 100%);
    }

    @media (max-width: 768px) {
        .meter {
            width: 100%;
        }
    }
</style>
