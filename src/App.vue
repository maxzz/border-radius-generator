<template>
    <main>
        <!-- <div class="blob-show blob-borders-0" :style="{borderRadius: generatedCss}"></div> -->

        <div class="blob-wrap" :style="{'--border-width': `${options.borderWidth}px`}">
            <div v-for="item of Number(options.shapes)" :key="item" class="blob-show2" 
                :style="{
                    borderRadius: generatedCss,
                    backgroundColor: options.shapes == 1 ? 'red' : 'oldlace',
                    transform: getItemTransform(item)
                }"
                :class="{'blob-borders': options.showBorder}"
            >
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
                    <label>Scale: <input type="range" min=".001" max="2" step=".001" v-model="options.scale"><span class="value-disp">{{options.scale}}</span></label>
                </div>
        
                <div class="control-row">
                    <span>Shift:</span>
                    <label>X: <input type="range" min="-50" max="50" step="1" v-model="options.shiftX"><span class="value-disp">{{options.shiftX}}</span></label>
                    <label>Y: <input type="range" min="-50" max="50" step="1" v-model="options.shiftY"><span class="value-disp">{{options.shiftY}}</span></label>
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

    function generateShape(symmetrical) {
        let r1 = random(5, 96);
        let r2 = random(5, 96);
        let r3 = random(5, 96);
        let r4 = random(5, 96);

        let r11, r22, r33, r44;

        if (symmetrical) {
            r11 = 100 - r1;
            r22 = 100 - r2;
            r33 = 100 - r3;
            r44 = 100 - r4;
        } else {
            r11 = random(5, 96);
            r22 = random(5, 96);
            r33 = random(5, 96);
            r44 = random(5, 96);
        }

        return `${r1}% ${r11}% ${r22}% ${r2}% / ${r3}% ${r4}% ${r44}% ${r33}%`;
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

            return {
                options,
                generatedCss,
                generatedTxt,
                getItemTransform,
                onGenerate,
            };
        }
    };
</script>

<style lang="scss">
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
        row-gap: 1em;
        margin-top: 1em;
    }

    .control-panel {
        display: grid;
        row-gap: .4em;
    }

    input[type=button], input[type=text] {
        padding: 1em;
    }

    label {
        display: grid;
        grid-auto-flow: column;
        align-items: center;
        column-gap: .4em;
    }

    .value-disp {
        width: 1em;
        font-size: .7em;
    }

    .control-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
        user-select: none;
    }

    .blob-show {
        position: relative;
        width: 400px;
        height: 400px;
        background-color: red;
        transition: 1s border-radius;
    }

    .blob-borders-0 {
        border-top: 5px solid hsl(120, 100%, 25%);
        border-right: 5px solid hsl(120, 100%, 50%);
        border-bottom: 5px solid hsl(120, 100%, 25%);
        border-left: 5px solid hsl(120, 100%, 50%);
    }

    $border-tl: hsl(0, 75%, 50%);
    $border-tr: hsl(90, 75%, 50%);
    $border-br: hsl(180, 75%, 50%);
    $border-bl: hsl(270, 75%, 50%);

    .blob-borders {
        border-top: var(--border-width) solid $border-tl;
        border-right: var(--border-width) solid $border-tr;
        border-bottom: var(--border-width) solid $border-br;
        border-left: var(--border-width) solid $border-bl;
    }

    .toggle-showborder {
        .legend {
            display: none;
            margin-left: 2em;

            column-gap: .2em;
            grid-template-columns: repeat(4, 3em);
            text-align: center;
            font-size: .8em;
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

    .blob-wrap {
        position: relative;
        width: 400px;
        height: 400px;
        border: 1px dashed red;
        background-color: #fff;
    }

    .blob-show2 {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;

        transition: 1s border-radius;
        margin: 1em;
    }
</style>
