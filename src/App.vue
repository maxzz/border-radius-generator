<template>
    <main>
        <div class="bubbas" :style="{'--border-width': `${options.borderWidth}px`}">
            <div v-for="item of Number(options.shapes)" :key="item" class="bubba" 
                :style="{
                    borderRadius: generatedCss,
                    backgroundColor: options.shapes == 1 ? 'red' : 'oldlace',
                    transform: getItemTransform(item)
                }"
                :class="{'bubba-borders': options.showBorder}"
            >
                <div class="bubba-marker" :style="{width: corners[0], height: corners[4], top: '0', left: '0'}" :title="`${corners[0]}, ${corners[4]}`"> </div>
                <div class="bubba-marker" :style="{width: corners[1], height: corners[5], top: '0', right: '0'}"></div>
                <div class="bubba-marker" :style="{width: corners[2], height: corners[6], bottom: '0', right: '0'}"></div>
                <div class="bubba-marker" :style="{width: corners[3], height: corners[7], bottom: '0', left: '0'}"></div>

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
                    <label>Step scale: <input type="range" min=".001" max="2" step=".001" v-model="options.scale"><span class="value-disp">{{options.scale}}</span></label>
                </div>
        
                <div class="control-row double-range">
                    <span>Step shift:</span>
                    <label>x: <input type="range" min="-50" max="50" step="1" v-model="options.shiftX"><span class="value-disp">{{options.shiftX}}</span></label>
                    <label>y: <input type="range" min="-50" max="50" step="1" v-model="options.shiftY"><span class="value-disp">{{options.shiftY}}</span></label>
                </div>
        
                <div class="control-row">
                    <label>Generate symmetrical corners<input type="checkbox" v-model="options.symmetrical"></label>
                </div>

                <div class="control-row">
                    <label class="toggle-showborder">Show border<input type="checkbox" v-model="options.showBorder">
                        <div v-show="options.showBorder" class="legend">
                            <span class="legend-tl" title="top-left">t-l</span>
                            <span class="legend-tr" title="top-right">t-r</span>
                            <span class="legend-br" title="bottom-right">b-r</span>
                            <span class="legend-bl" title="bottom-left">b-l</span>
                        </div>
                    </label>
                </div>
            </template>
        </div>

    </main>
</template>

<script>
    import { computed, reactive, ref } from 'vue';

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

    export default {
        setup() {
            const generatedCss = ref('23% 77% 82% 18% / 59% 21% 79% 41%'); // ref(generateShape(true));
            const generatedTxt = computed(() => generatedCss.value ? `border-radius: ${generatedCss.value}` : '');

            const options = reactive({
                shapes: 2,
                borderWidth: 1,
                scale: .1,
                shiftX: 20,
                shiftY: 20,
                showBorder: true,
                showControls: true,
                symmetrical: true,
            });

            function getItemCss(num) {
                return ``;
            }

            function getItemTransform(num) {
                let idx = num - 1;
                return `scale(${1 - options.scale * idx})  translate(${options.shiftX * idx}px, ${options.shiftY * idx}px)`;
            }

            function onGenerate() {
                generatedCss.value = generateShape(options.symmetrical);
            }

            let corners = computed(() => generateCorners(generatedCss.value));

            return {
                options,
                generatedCss,
                generatedTxt,
                getItemTransform,
                onGenerate,
                corners,
            };
        }
    };
</script>

<style lang="scss">
    $border-tl: hsl(0, 75%, 50%);
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
        border: 1px dashed red;
        background-color: #fff;
    }

    .bubba {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;

        transition: 1s border-radius;
        margin: 1rem;

        outline: 1px dashed green;
    }

    .bubba-marker {
        position: absolute;
        //background-color: rgba(238, 130, 238, 0.226);
        outline: 1px dashed darkblue;
    }

    $marker-opacity: .9;

    .bubba-marker:nth-child(1) {
        background-color: transparentize($border-tl, $marker-opacity);
    }

    .bubba-marker:nth-child(2) {
        background-color: transparentize($border-tr, $marker-opacity);
    }

    .bubba-marker:nth-child(3) {
        background-color: transparentize($border-br, $marker-opacity);
    }

    .bubba-marker:nth-child(4) {
        background-color: transparentize($border-bl, $marker-opacity);
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
    }

    .bubba-borders {
        border-top: var(--border-width) solid $border-tl;
        border-right: var(--border-width) solid $border-tr;
        border-bottom: var(--border-width) solid $border-br;
        border-left: var(--border-width) solid $border-bl;
    }

    .bubba-borders-alt {
        border-top: 5px solid hsl(120, 100%, 25%);
        border-right: 5px solid hsl(120, 100%, 50%);
        border-bottom: 5px solid hsl(120, 100%, 25%);
        border-left: 5px solid hsl(120, 100%, 50%);
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
