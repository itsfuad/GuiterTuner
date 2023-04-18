<script>
    import { onMount } from "svelte";
    import { fade, fly } from "svelte/transition";

    export let instrument;

    let bar;
    let Frequency = 0;
    let Note = "--";
    let Octave = "-";

    let Stream;


    let MeterElem;
    let MeterPointerElem;

    class Tuner {
        constructor(a4) {
            this.middleA = a4 || 440;
            this.bufferSize = 4096;
            this.noteStrings = [
                "C",
                "C♯",
                "D",
                "D♯",
                "E",
                "F",
                "F♯",
                "G",
                "G♯",
                "A",
                "A♯",
                "B",
            ];

            this.initGetUserMedia();
        }
        initGetUserMedia() {
            window.AudioContext =
                window.AudioContext || window.webkitAudioContext;
            if (!window.AudioContext) {
                return alert("AudioContext not supported");
            }

            // Older browsers might not implement mediaDevices at all, so we set an empty object first
            if (navigator.mediaDevices === undefined) {
                navigator.mediaDevices = {};
            }

            // Some browsers partially implement mediaDevices. We can't just assign an object
            // with getUserMedia as it would overwrite existing properties.
            // Here, we will just add the getUserMedia property if it's missing.
            if (navigator.mediaDevices.getUserMedia === undefined) {
                navigator.mediaDevices.getUserMedia = function (constraints) {
                    // First get ahold of the legacy getUserMedia, if present
                    const getUserMedia =
                        navigator.webkitGetUserMedia ||
                        navigator.mozGetUserMedia;

                    // Some browsers just don't implement it - return a rejected promise with an error
                    // to keep a consistent interface
                    if (!getUserMedia) {
                        alert(
                            "getUserMedia is not implemented in this browser"
                        );
                    }

                    // Otherwise, wrap the call to the old navigator.getUserMedia with a Promise
                    return new Promise(function (resolve, reject) {
                        getUserMedia.call(
                            navigator,
                            constraints,
                            resolve,
                            reject
                        );
                    });
                };
            }
        }
        startRecord() {
            const self = this;
            navigator.mediaDevices
                .getUserMedia({ audio: true })
                .then(function (stream) {
                    Stream = stream;
                    self.audioContext
                        .createMediaStreamSource(stream)
                        .connect(self.analyser);
                    self.analyser.connect(self.scriptProcessor);
                    self.scriptProcessor.connect(self.audioContext.destination);
                    self.scriptProcessor.addEventListener(
                        "audioprocess",
                        function (event) {
                            const frequency = self.pitchDetector.do(
                                event.inputBuffer.getChannelData(0)
                            );
                            if (frequency && self.onNoteDetected) {
                                self.onNoteDetected(frequency);
                            }
                        }
                    );
                })
                .catch(function (error) {
                    alert(error.name + ": " + error.message);
                });
        }
        init() {
            this.audioContext = new window.AudioContext();
            this.analyser = this.audioContext.createAnalyser();
            this.scriptProcessor = this.audioContext.createScriptProcessor(
                this.bufferSize,
                1,
                1
            );

            const self = this;

            aubio().then(function (aubio) {
                self.pitchDetector = new aubio.Pitch(
                    "default",
                    self.bufferSize,
                    1,
                    self.audioContext.sampleRate
                );
                self.startRecord();
            });
        }
    }

    let isListening = false;
    let tuner = new Tuner(440);
    let cent = 0;

    function start() {
        isListening = true;
        tuner.init();
        tuner.onNoteDetected = (frequency) => {
            Frequency = frequency;
            let index = Math.round((Math.log(frequency / 440) / Math.log(2)) * 12 - 3);
            index = (index + tuner.noteStrings.length) % tuner.noteStrings.length;
            Octave = Math.round(Math.log2(frequency / 440) + 4);
            Note = tuner.noteStrings[index];
            cent = Math.round((Math.log(frequency / 440) / Math.log(2)) * 1200);
            bar.style.height = `${(frequency / 1000) * 100}%`;
            //rotate meter
            if (cent > 45) {
                cent = 45;
            }
            if (cent < -45) {
                cent = -45;
            }
            
            MeterPointerElem.style.transform = `rotate(${cent}deg)`;
        };
    }

    function stop() {
        isListening = false;
        tuner.audioContext.close();
        Stream.getTracks().forEach((track) => track.stop());
    }


    onMount(() => {
        for (var i = 0; i <= 10; i += 1) {
            const $scale = document.createElement("div");
            $scale.className = "meter-scale";
            $scale.style.transform = "rotate(" + (i * 9 - 45) + "deg)";
            if (i % 5 === 0) {
            $scale.classList.add("meter-scale-strong");
            }
            MeterElem.appendChild($scale);
        }
    });

</script>

<div class="container">
    <div class="instrument">{instrument}</div>
    <div class="tuner">
        <div class="note">
            <div class="flat" in:fly={{ y: 10, delay: 300 }}>♭</div>
            <div class="normal" in:fly={{ y: 10, delay: 400 }}>
                <div class="noteName">
                    {Note}{Octave} | {Math.round(Frequency)}Hz
                </div>
                <div class="meter" bind:this={MeterElem}>
                    <div class="meter-dot" />
                    <div class="meter-pointer" bind:this={MeterPointerElem} />
                </div>
            </div>
            <div class="sharp" in:fly={{ y: 10, delay: 500 }}>♯</div>
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
    .meter {
        position: fixed;
        left: 0;
        right: 0;
        bottom: 50%;
        width: 400px;
        height: 33%;
        margin: 0 auto 5vh auto;
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

    .meter-pointer{
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
    }

    @media (max-width: 768px) {
        .meter {
            width: 100%;
        }
    }
</style>
