<template>
    <main>
        <!-- <div class="blob-show" :style="{borderRadius: generated}"></div> -->

        <div class="blob-wrap">
            <div v-for="item of [0,1,2,3]" :key="item" class="blob-show2" :style="{
                borderRadius: generated,
                transform: `scale(${1 - .2 * item})`
                }"></div>
        </div>


        <input type="button" value="Generate" @click="onGenerate">
        <input :value="generatedOutput" readonly>
    </main>
</template>

<script>
    import { computed, ref } from 'vue';

    function random(min, max){
        return Math.floor(min + Math.random() * (max - min));
    }

    function generateShape() {
        var r1 = random(5, 100);
        var r2 = random(5, 100);
        var r3 = random(5, 100);
        var r4 = random(5, 100);

        var r11 = 100 - r1;
        var r22 = 100 - r2;
        var r33 = 100 - r3;
        var r44 = 100 - r4;

        return `${r1}% ${r11}% ${r22}% ${r2}% / ${r3}% ${r4}% ${r44}% ${r33}%`;
    }

    export default {
        setup() {
            const generated = ref(generateShape());
            const generatedOutput = computed(() => generated.value ? `border-radius: ${generated.value}` : '');

            function onGenerate() {
                generated.value = generateShape();
            }

            return {
                generated,
                generatedOutput,
                onGenerate,
            };
        }
    };
</script>

<style>
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

    input {
        padding: 1em;
    }

    .blob-show {
        position: relative;
        width: 400px;
        height: 400px;
        background-color: red;
        transition: 1s border-radius;

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

    .blob-show2 {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;

        background-color: oldlace;
        transition: 1s border-radius;

        border-top: 15px solid hsl(0, 75%, 50%);
        border-right: 15px solid hsl(90, 75%, 50%);
        border-bottom: 15px solid hsl(180, 75%, 50%);
        border-left: 15px solid hsl(270, 75%, 50%);
    }


</style>
