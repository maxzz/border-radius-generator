<template>
    <main>
        <div class="blob-show" :style="{borderRadius: generated}"></div>
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
</style>
