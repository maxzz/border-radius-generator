<template>
    <main>
        <div class="bubbas" :style="{'--border-width': `${options.borderWidth}px`}">
            <div 
                v-for="item of Number(options.shapes)" :key="item"
                class="bubba" 
                :class="{'bubba-borders': options.showBorder}"
                :style="{
                    borderRadius: generatedCss,
                    transform: getBubbaTransform(item),
                    backgroundColor: getBubbaBackground(item),
                    outline: options.showCssRects ? '1px dashed green' : 'none'
                }"
            >
                <div class="markers" v-if="!options.showOnlyOneRect || item === Number(options.shapes)"
                    :style="{
                        '--w0': corners[0],
                        '--w1': corners[1],
                        '--w2': corners[2],
                        '--w3': corners[3],
                        '--h0': corners[4],
                        '--h1': corners[5],
                        '--h2': corners[6],
                        '--h3': corners[7],
                    }"
                >
                    <template v-if="optionsShowRect.showCssRects">
                        <div class="css-marker bm-tl" :title="`top-left\n${corners[0]}, ${corners[4]}`"> </div>
                        <div class="css-marker bm-tr" :title="`top-right\n${corners[1]}, ${corners[5]}`"></div>
                        <div class="css-marker bm-br" :title="`bottom-right\n${corners[2]}, ${corners[6]}`"></div>
                        <div class="css-marker bm-bl" :title="`bottom-left\n${corners[3]}, ${corners[7]}`"></div>
                    </template>

                    <template v-if="optionsShowRect.showSvgRects">
                        <!-- options.showSvgFrame -->
                        <svg class="svg-marker bm-tl" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse pathLength="4" :cx="`100%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-tr" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse pathLength="4" :cx="`0%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-br" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse pathLength="4" :cx="`0%`" :cy="`0%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-bl" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse pathLength="4" :cx="`100%`" :cy="`0%`" :rx="`100%`" :ry="`100%`"/> </svg>
                    </template>

                    <template v-if="optionsShowRect.showSvgRects && animateStokes ">
                        <!-- options.showSvgFrame -->
                        <svg class="ani-marker bm-tl" :style="{'--seg-b': 2, '--seg-e': 1, '--delay': '0s'}"> <ellipse pathLength="4" :cx="`100%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="ani-marker bm-tr" :style="{'--seg-b': 1, '--seg-e': 0, '--delay': '.5s'}"> <ellipse pathLength="4" :cx="`0%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="ani-marker bm-br" :style="{'--seg-b': 4, '--seg-e': 3, '--delay': '1s'}"> <ellipse pathLength="4" :cx="`0%`" :cy="`0%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="ani-marker bm-bl" :style="{'--seg-b': 3, '--seg-e': 2, '--delay': '1.5s'}"> <ellipse pathLength="4" :cx="`100%`" :cy="`0%`" :rx="`100%`" :ry="`100%`" @animationend="animateStokes = false"/> </svg>
                    </template>
                </div>

                <span v-if="options.showBorder">{{item}}</span>
            </div>
        </div>

        <input type="text" :value="generatedTxt" readonly>

        <input type="button" value="Generate" @click="onGenerate">

        <div class="control-panel">
            <input type="button" class="controls-header" @click="options.showControls = !options.showControls" :value="options.showControls ? 'Hide controls' : 'Show controls'" />

            <template v-if="options.showControls">
                <div class="control-row">
                    <label>Number of shapes: <input type="range" min="1" max="20" step="1" v-model="options.shapes"><span class="value-disp">{{options.shapes}}</span></label>
                </div>
        
                <div class="control-row">
                    <label>Boder: <input type="range" min="1" max="100" step="1" v-model="options.borderWidth"><span class="value-disp">{{options.borderWidth}}</span></label>
                </div>

                <div class="control-row">
                    <label>Step scale: <input type="range" min=".0001" max="2" step=".0001" v-model="options.scale"><span class="value-disp">{{options.scale}}</span></label>
                </div>
        
                <div class="control-row double-range">
                    <span>Step shift:</span>
                    <label>x: <input type="range" min="-50" max="50" step="1" v-model="options.shiftX"><span class="value-disp">{{options.shiftX}}</span></label>
                    <label>y: <input type="range" min="-50" max="50" step="1" v-model="options.shiftY"><span class="value-disp">{{options.shiftY}}</span></label>
                </div>

                <div class="control-row">
                    <label>Generate symmetrical corners<input type="checkbox" v-model="options.symmetrical"></label>
                </div>

                <fieldset>
                    <legend>Show options</legend>

                    <div class="control-row" style="justify-content: start">
                        Show corners:
                        <div class="row-show-corners">
                            <label>None<input type="radio" value="0" name="showCorners" v-model="options.showRects"></label>
                            <label>CSS <input type="radio" value="1" name="showCorners" v-model="options.showRects"></label>
                            <label>SVG <input type="radio" value="2" name="showCorners" v-model="options.showRects"></label>
                        </div>
                    </div>

                    <!-- <div class="control-row">
                        <label>Show CSS corner rectangles<input type="checkbox" v-model="options.showCssRects"></label>
                    </div>
    
                    <div class="control-row">
                        <label>Show SVG corner rectangles<input type="checkbox" v-model="options.showSvgRects"></label>
                    </div>
    
                    <div class="control-row">
                        <label>Show SVG frame<input type="checkbox" v-model="options.showSvgFrame"></label>
                    </div> -->

                    <div class="control-row">
                        <label>Show corners only on the last rectangle<input type="checkbox" v-model="options.showOnlyOneRect"></label>
                    </div>

                    <div class="control-row">
                        <label class="toggle-showborder">Show borders<input type="checkbox" v-model="options.showBorder">
                            <div v-show="options.showBorder" class="legend">
                                <span class="legend-tl" title="top-left">T-L</span>
                                <span class="legend-tr" title="top-right">T-R</span>
                                <span class="legend-br" title="bottom-right">B-R</span>
                                <span class="legend-bl" title="bottom-left">B-L</span>
                            </div>
                        </label>
                    </div>
                </fieldset>

                <div class="control-row">
                    <label>Animate<input type="checkbox" v-model="options.animate" @change="onAnimate"></label> <!-- TODO: store the last 12 steps of animation in history -->
                    <label>Demo options<input type="checkbox" v-model="options.demoMode" @change="onDemoMode"></label>
                </div>
            </template>
        </div>
    </main>
</template>

<style lang="scss">
    .row-show-corners {
        display: flex;
        justify-content: space-between;

        & > label {
            margin: 0 .8em;
        }
    }
</style>

<script lang="ts">
    import { computed, nextTick, reactive, ref, toRaw } from 'vue';

    //#region types
    const enum ShowRects {
        none,
        css,
        svg
    }

    type Options = {
        showControls: boolean;
        shapes: number,
        borderWidth: number,
        scale: number,
        shiftX: number,
        shiftY: number,
        symmetrical: boolean;
        showRects: ShowRects, // 0 - none; 1 - CSS; 2 - SVG
        showCssRects: boolean;
        showSvgRects: boolean;
        showBorder: boolean;
        showSvgFrame: boolean;
        showOnlyOneRect: boolean;
        animate: boolean,
        demoMode: boolean,
    };

    const defaultOptions: Options = {
        showControls: true,
        shapes: 1,
        borderWidth: 1,
        scale: .1,
        shiftX: 20,
        shiftY: 20,
        symmetrical: true,
        showRects: 0, // 0 - none; 1 - CSS; 2 - SVG
        showCssRects: true,
        showSvgRects: true,
        showBorder: true,
        showSvgFrame: true,
        showOnlyOneRect: true,
        animate: false,
        demoMode: false,
    };

    const demoOptions: Options = {
        showControls: true,
        shapes: 10,
        borderWidth: 1,
        scale: .1, // scale: .0043,
        shiftX: 9,
        shiftY: 7,
        symmetrical: true,
        showRects: 0,
        showCssRects: true,
        showSvgRects: true,
        showBorder: true,
        showSvgFrame: true,
        showOnlyOneRect: false,
        animate: true,
        demoMode: true,
    };

    // const demoOptions2: Options = {
    //     showControls: true,
    //     shapes: 18,
    //     borderWidth: 1,
    //     scale: .0947,
    //     shiftX: 20,
    //     shiftY: 6,
    //     symmetrical: true,
    //     showRects: 0,
    //     showCssRects: false,
    //     showSvgRects: true,
    //     showBorder: true,
    //     showSvgFrame: true,
    //     showOnlyOneRect: true,
    //     animate: true,
    //     demoMode: true,
    // };

    //#endregion types

    //#region local utils

    function assignToReactive(toObj, fromObj) {
        for (const [k, v] of Object.entries(fromObj)) {
            toObj[k] = v;
        }
    }

    function nonReactive(reactiveObj) {
        return JSON.parse(JSON.stringify(reactiveObj));
    }

    function filterKeys(obj, keys) {
        keys = keys || [];
        return Object.fromEntries(Object.entries(obj).filter(([k, v]) => !keys.includes(k)));
    }

    function random(min, max){
        return Math.floor(min + Math.random() * (max - min));
    }

    function generateCorners(borderRadius) {
        return [...(borderRadius || '').matchAll(/(\d\d*%)/g)].map(m => m[1]);
    }

    function generateShape(symmetrical) {
        let wTL = random(5, 96);
        let wBL = random(5, 96);
        let hTL = random(5, 96);
        let hTR = random(5, 96);

        let wTR, wBR, hBL, hBR;

        if (symmetrical) {
            wTR = 100 - wTL;
            wBR = 100 - wBL;
            hBL = 100 - hTL;
            hBR = 100 - hTR;
        } else {
            wTR = random(5, 96);
            wBR = random(5, 96);
            hBL = random(5, 96);
            hBR = random(5, 96);
        }

        return `${wTL}% ${wTR}% ${wBR}% ${wBL}% / ${hTL}% ${hTR}% ${hBR}% ${hBL}%`;
    }

    const paletteTeal: string[] = [
        '#e0f2f1',
        '#b2dfdb',
        '#80cbc4',
        '#4db6ac',
        '#26a69a',
        '#009688',
        '#00897b',
        '#00796b',
        '#00695c',
        '#004d40',
    ];

    //#endregion local utils

    export default {
        setup() {
            const generatedCss = ref('23% 77% 82% 18% / 59% 21% 79% 41%'); // ref(generateShape(true)); // TODO: allow edit: '10% 90% 28% 72% / 20% 58% 42% 80%'
            const generatedTxt = computed(() => generatedCss.value ? `border-radius: ${generatedCss.value}` : '');

            let options: Options = reactive(defaultOptions);

            let optionsShowRect = reactive({
                showCssRects: computed(() => options.showRects == ShowRects.css),
                showSvgRects: computed(() => options.showRects == ShowRects.svg),
            });

            options.demoMode && (options = reactive(demoOptions));

            function getItemCss(num) {
                return ``;
            }

            function getBubbaTransform(num) {
                let idx = num - 1;
                return `scale(${1 - options.scale * idx})  translate(${options.shiftX * idx}px, ${options.shiftY * idx}px)`;
            }

            function getBubbaBackground(num) {
                let idx = num - 1;
                return options.shapes == 1 ? paletteTeal[0] : paletteTeal[idx % paletteTeal.length];
            }

            const animateStokes = ref(false);

            function showAnimatedStrokes() {
                animateStokes.value = false; // this is to cancel prev animation
                nextTick(() => animateStokes.value = true);
            }

            function onGenerate() {
                generatedCss.value = generateShape(options.symmetrical);
                showAnimatedStrokes();
            }

            let corners = computed(() => generateCorners(generatedCss.value));

            let timerID;
            function onAnimate() {
                if (options.animate) {
                    if (timerID) {
                        clearInterval(timerID);
                    }
                    onGenerate();
                    timerID = setInterval(onGenerate, 2200); // This shoild be more then transion on .bubba
                } else {
                    if (timerID) {
                        clearInterval(timerID);
                        timerID = null;
                    }
                }
            }

            let currentOptions: any = defaultOptions;
            function onDemoMode(event) {
                if (options.demoMode) {
                    currentOptions = filterKeys(nonReactive(options), ['demoMode']);
                    assignToReactive(options, demoOptions);
                } else {
                    assignToReactive(options, currentOptions);
                }
                onAnimate();
            }

            function woPersent(v) {
                return v.replace('%', '');
            }

            return {
                options,
                optionsShowRect,
                generatedCss,
                generatedTxt,
                getBubbaTransform,
                getBubbaBackground,
                onGenerate,
                onAnimate,
                onDemoMode,
                corners,
                woPersent,
                animateStokes,
            };
        }
    };
</script>

<style lang="scss">
    $border-tl: hsl(30, 75%, 50%);
    $border-tr: hsl(90, 75%, 50%);
    $border-br: hsl(180, 75%, 50%);
    $border-bl: hsl(270, 75%, 50%);

    *, *::before, *::after {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    body {
        height: 100vh;
        width: calc(100vw - 32px); // - scrollbar
        margin-left: 16px;
        display: grid;
        justify-content: center;
        background-color: #f9f9f9;
    }

    main {
        display: grid;
        row-gap: 1rem;
        margin-top: 1rem;
    }

    .bubbas {
        position: relative;
        width: 400px;
        height: 400px;
        background-color: white;
        border: 10px solid #f9f9f9;
        box-shadow: 0 0 4px 1px rgba(0, 0, 0, 0.1);
        
        justify-self: center;
        overflow: auto;
        resize: both;
        min-width: 100px;
        min-height: 100px;
    }

    .bubba {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        user-select: none;
        transition: 1s border-radius;
    }

    .markers {
        .bm-tl {
            top: 0;
            left: 0;
            width: var(--w0);
            height: var(--h0);
        }

        .bm-tr {
            top: 0;
            right: 0;
            width: var(--w1);
            height: var(--h1);
        }

        .bm-br {
            bottom: 0;
            right: 0;
            width: var(--w2);
            height: var(--h2);
        }

        .bm-bl {
            bottom: 0;
            left: 0;
            width: var(--w3);
            height: var(--h3);
        }
    }

    .css-marker {
        position: absolute;
        outline: 1px dashed yellowgreen;

        $marker-opacity: .6;
        
        &.bm-tl {
            background-color: transparentize($border-tl, $marker-opacity);
        }

        &.bm-tr {
            background-color: transparentize($border-tr, $marker-opacity);
        }

        &.bm-br {
            background-color: transparentize($border-br, $marker-opacity);
        }

        &.bm-bl {
            background-color: transparentize($border-bl, $marker-opacity);
        }
    }

    .svg-marker {
        position: absolute;
        outline: 1px dashed yellowgreen;

        $marker-opacity: .6;

        fill: none;

        & ellipse {
            // fill: yellow;
            // fill: none;
            // fill: rgba(0, 255, 0, .2);
            stroke: green;
            stroke-width: .5;
            //transition: all 2s;
        }
    }

    input[type=button], input[type=text] {
        padding: 1rem;
    }

    .control-panel {
        display: grid;
        row-gap: .4rem;
        font-size: .9rem;

        label {
            display: grid;
            grid-auto-flow: column;
            align-items: center;
            column-gap: .4rem;
        }

        .value-disp {
            width: 1rem;
            font-size: .7rem;
        }

        .control-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            user-select: none;
        }

        .double-range input {
            width: 7rem;
        }

        fieldset {
            padding: .4em 1em .4em 1.4em;
            border: 1px solid #888;
            border-radius: 3px;

            & legend {
                padding: 0 .4em;
                margin-left: -.4em;
            }
        }
    }

    .bubba-borders {
        border: var(--border-width) solid;
        border-color: $border-tl $border-tr $border-br $border-bl;
    }

    .bubba-borders-alt {
        border: var(--border-width) solid;
        border-color: hsl(120, 100%, 25%) hsl(120, 100%, 50%);
    }

    .toggle-showborder {
        .legend {
            display: none;
            margin-left: 2rem;

            column-gap: .2rem;
            grid-template-columns: repeat(4, 3em);
            text-align: center;
            font-size: .8rem;
            color: #e4e4e4;

            .legend-tl {
                background-color: $border-tl;
                color: #f1f1ff;
            }
            .legend-tr {
                background-color: $border-tr;
                color: #444444;
            }
            .legend-br {
                background-color: $border-br;
                color: #444444;
            }
            .legend-bl {
                background-color: $border-bl;
            }
        }

        &:hover .legend {
            display: grid;
        }
    }
</style>

<style lang="scss">
    .ani-marker {
        position: absolute;

        & ellipse {
            fill: transparent;
            stroke: darkmagenta;
            stroke-width: 0;

            stroke-dashoffset: 0;
            stroke-dasharray: 4;
            animation: 1s enter-ani-marker;
            animation-delay: var(--delay);
        }
    }

    @keyframes enter-ani-marker {
        from {
            stroke-dashoffset: var(--seg-b);
            opacity: 0;
            stroke-width: 1;
        }
        50% {
            opacity: 1;
            stroke-dashoffset: var(--seg-e);
            stroke-width: 10;
        }
        70% {
            opacity: 0;
            stroke-width: 1;
        }
        100% {
            opacity: 0;
        }
    }
</style>
