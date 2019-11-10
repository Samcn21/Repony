<template>
    <div id="app">
        <MazeInit v-on:get-mazeData="saveMazeData" />
        <MazeDisplay v-bind:mazeID="mazeID"
                     v-bind:gameplayType="gameplayType"
                     v-bind:ponyName="ponyName" />
    </div>
</template>

<script>
import MazeInit from '../src/components/MazeInit.vue'
import MazeDisplay from '../src/components/MazeDisplay.vue'

export default {
    name: 'app',
    components: {
        MazeInit,
        MazeDisplay
    },
    data() {
        return {
            mazeID: '',
            ponyName: '',
            gameplayType: ''
        }
    },
    methods: {
        saveMazeData(mazeID, ponyName, gameplayType) {
            this.mazeID = mazeID,
            this.ponyName = ponyName,
            this.gameplayType = gameplayType;
        }
    }
}
</script>

<style>
    * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    body {
        font-family: Arial, Helvetica sans-serif;
        line-height: 1.4;
        font-size: 14px;
        width: 1280px;
        margin: 0 auto;
    }

    #app {
        text-align: center;
        margin-top: 1rem;
    }

    input[type="submit"] {
        position: relative;
        display: block;
        padding: 19px 39px 18px 39px;
        color: #FFF;
        background-color: #1abc9c;
        font-size: 18px;
        text-align: center;
        font-style: normal;
        width: 25%;
        border: 1px solid #16a085;
        border-width: 1px 1px 3px;
        cursor: pointer;
    }

    input[type="submit"]:hover {
        background: #109177;
    }

    input[type="submit"].disabled {
        background-color: #a3e4d7;
        cursor: auto;
        color: #ebf7f1;
        border: 1px solid #a2d9ce;
        border-width: 1px 1px 3px;
    }

    input[type="submit"]:focus {
        outline: none;
    }
</style>
