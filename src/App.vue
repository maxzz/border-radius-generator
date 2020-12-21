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
                <div class="markers"
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
                    <template v-if="options.showCssRects">
                        <div class="css-marker bm-tl" :title="`top-left\n${corners[0]}, ${corners[4]}`"> </div>
                        <div class="css-marker bm-tr" :title="`top-right\n${corners[1]}, ${corners[5]}`"></div>
                        <div class="css-marker bm-br" :title="`bottom-right\n${corners[2]}, ${corners[6]}`"></div>
                        <div class="css-marker bm-bl" :title="`bottom-left\n${corners[3]}, ${corners[7]}`"></div>
                    </template>

                    <template v-if="options.showSvgFrame">
                        <svg class="svg-marker bm-tl" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse :cx="`100%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-tr" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse :cx="`0%`" :cy="`100%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-br" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse :cx="`0%`" :cy="`0%`" :rx="`100%`" :ry="`100%`"/> </svg>
                        <svg class="svg-marker bm-bl" :style="{fill: options.showSvgRects ? 'rgba(0, 255, 0, .2)' : 'none'}"> <ellipse :cx="`100%`" :cy="`0%`" :rx="`100%`" :ry="`100%`"/> </svg>
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
    
                    <div class="control-row">
                        <label>Show CSS corner rectangles<input type="checkbox" v-model="options.showCssRects"></label>
                    </div>
    
                    <div class="control-row">
                        <label>Show SVG corner rectangles<input type="checkbox" v-model="options.showSvgRects"></label>
                    </div>
    
                    <div class="control-row">
                        <label>Show SVG frame<input type="checkbox" v-model="options.showSvgFrame"></label>
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

<script>
    import { computed, reactive, ref, toRaw } from 'vue';
    //import { hexaToRgba, rgbaToHsla } from './colors.ts';

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

    const paletteTeal = [
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
        '#a7ffeb',
        '#64ffda',
        '#1de9b6',
        '#00bfa5',
    ];

    export default {
        setup() {
            const generatedCss = ref('23% 77% 82% 18% / 59% 21% 79% 41%'); // ref(generateShape(true)); // TODO: allow edit: '10% 90% 28% 72% / 20% 58% 42% 80%'
            const generatedTxt = computed(() => generatedCss.value ? `border-radius: ${generatedCss.value}` : '');

            const defaultOptions = {
                showControls: true,
                shapes: 1,
                borderWidth: 1,
                scale: .1,
                shiftX: 20,
                shiftY: 20,
                symmetrical: true,
                showCssRects: true,
                showSvgRects: true,
                showBorder: true,
                showSvgFrame: true,
                animate: false,
                demoMode: false,
            };

            const demoOptions = {
                showControls: true,
                shapes: 10,
                borderWidth: 1,
                scale: .1, // scale: .0043,
                shiftX: 9,
                shiftY: 7,
                symmetrical: true,
                showCssRects: true,
                showSvgRects: true,
                showBorder: true,
                showSvgFrame: true,
                animate: true,
                demoMode: true,
            };

            const demoOptions2 = {
                showControls: true,
                shapes: 18,
                borderWidth: 1,
                scale: .0947,
                shiftX: 20,
                shiftY: 6,
                symmetrical: true,
                showCssRects: false,
                showSvgRects: true,
                showBorder: true,
                showSvgFrame: true,
                animate: true,
                demoMode: true,
            };

            let options = reactive(defaultOptions);

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
                if (options.shapes == 1) {
                    return 'red';
                }

                return options.shapes == 1 ? 'red' : paletteTeal[idx % paletteTeal.length];
            }

            function onGenerate() {
                generatedCss.value = generateShape(options.symmetrical);
            }

            let corners = computed(() => generateCorners(generatedCss.value));

            let timerID;

            function onAnimate() {
                if (options.animate) {
                    if (timerID) {
                        clearInterval(timerID);
                    }
                    timerID = setInterval(onGenerate, 1500); // This shoild be more then transion on .bubba
                } else {
                    if (timerID) {
                        clearInterval(timerID);
                        timerID = null;
                    }
                }
            }

            let currentOptions = defaultOptions;

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
                generatedCss,
                generatedTxt,
                getBubbaTransform,
                getBubbaBackground,
                onGenerate,
                onAnimate,
                onDemoMode,
                corners,
                woPersent,
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

        //background-color: red;
        //background-color: pink;
        //background-color: transparent;
        //fill: rgba(0, 255, 0, .2);

        & ellipse {
            // fill: yellow;
            // fill: none;
            // fill: rgba(0, 255, 0, .2);
            stroke: green;
            stroke-width: 1;
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
