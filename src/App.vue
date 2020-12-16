<template>
    <main>
        <!-- <div class="blob-show blob-borders-0" :style="{borderRadius: generatedCss}"></div> -->

        <div class="blob-wrap" :style="{'--border-width': `${options.borderWidth}px`}">
            <div v-for="item of Number(options.shapes)" :key="item" class="blob-show2" 
                :style="{
                    borderRadius: generatedCss,
                    transform: getItemTransform(item)
                }"
                :class="{'blob-borders': options.showBorder}"
            >
            <span v-if="options.showBorder">{{item}}</span>
            </div>
        </div>

        <input type="button" value="Generate" @click="onGenerate">

        <input type="text" :value="generatedTxt" readonly>

        <div class="control-row">
            <span>Shift:</span>
            <label>X: <input type="range" min="-50" max="50" step="1" v-model="options.shiftX">{{options.shiftX}}</label>
            <label>Y: <input type="range" min="-50" max="50" step="1" v-model="options.shiftY">{{options.shiftY}}</label>
        </div>

        <div class="control-row">
            <label>N shapes: <input type="range" min="1" max="20" step="1" v-model="options.shapes">{{options.shapes}}</label>
        </div>

        <div class="control-row">
            <label>Scale: <input type="range" min=".001" max="2" step=".001" v-model="options.scale">{{options.scale}}</label>
        </div>

        <div class="control-row">
            <label>Boder: <input type="range" min="1" max="40" step="1" v-model="options.borderWidth">{{options.borderWidth}}</label>
        </div>

        <div class="control-row">
            <label>Show border<input type="checkbox" v-model="options.showBorder"></label>
        </div>

    </main>
</template>

<script>
    import { computed, reactive, ref } from 'vue';

    function random(min, max){
        return Math.floor(min + Math.random() * (max - min));
    }

    function generateShape() {
        var r1 = random(5, 96);
        var r2 = random(5, 96);
        var r3 = random(5, 96);
        var r4 = random(5, 96);

        var r11 = 100 - r1;
        var r22 = 100 - r2;
        var r33 = 100 - r3;
        var r44 = 100 - r4;

        return `${r1}% ${r11}% ${r22}% ${r2}% / ${r3}% ${r4}% ${r44}% ${r33}%`;
    }

    export default {
        setup() {
            const generatedCss = ref('23% 77% 82% 18% / 59% 21% 79% 41%'); // ref(generateShape());
            const generatedTxt = computed(() => generatedCss.value ? `border-radius: ${generatedCss.value}` : '');

            const options = reactive({
                shapes: 3,
                borderWidth: 1,
                scale: .1,
                shiftX: 20,
                shiftY: 20,
                showBorder: true,
            });

            function getItemCss(num) {
                return ``;
            }

            function getItemTransform(num) {
                let idx = num - 1;
                return `scale(${1 - options.scale * idx})  translate(${options.shiftX * idx}px, ${options.shiftY * idx}px)`;
            }

            function onGenerate() {
                generatedCss.value = generateShape();
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
        display: grid;
        place-items: center;
    }

    main {
        display: grid;
        row-gap: 1em;
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

    .control-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
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

    .blob-borders {
        border-top: var(--border-width) solid hsl(0, 75%, 50%);
        border-right: var(--border-width) solid hsl(90, 75%, 50%);
        border-bottom: var(--border-width) solid hsl(180, 75%, 50%);
        border-left: var(--border-width) solid hsl(270, 75%, 50%);
    }

    .blob-wrap {
        position: relative;
        width: 400px;
        height: 400px;
        border: 1px dashed red;
    }

    .blob-show2 {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;

        background-color: oldlace;
        transition: 1s border-radius;
    }
</style>
