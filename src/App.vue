<template>
    <main>
        <!-- <div class="blob-show blob-borders-0" :style="{borderRadius: generatedCss}"></div> -->

        <div class="blob-wrap">
            <div v-for="item of 4" :key="item" class="blob-show2" 
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
            <span>Shift</span>
            <label>X: <input type="range"></label>
            <label>Y: <input type="range"></label>
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
            const generatedCss = ref(generateShape());
            const generatedTxt = computed(() => generatedCss.value ? `border-radius: ${generatedCss.value}` : '');

            const options = reactive({
                centerX: 0,
                centerY: 0,
                showBorder: true,
            });

            function getItemCss(num) {
                return ``;
            }

            function getItemTransform(num) {
                let idx = num - 1;
                return `scale(${1 - .1 * idx})  translate(${20 * idx}px, ${20 * idx}px)`;
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

    .blob-wrap {
        position: relative;
        width: 400px;
        height: 400px;
    }

    .blob-borders {
        border-top: 15px solid hsl(0, 75%, 50%);
        border-right: 15px solid hsl(90, 75%, 50%);
        border-bottom: 15px solid hsl(180, 75%, 50%);
        border-left: 15px solid hsl(270, 75%, 50%);
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
