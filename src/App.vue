<template>
    <div id="app">
        <MazeInit v-on:get-mazeData="saveMazeData" />
        <MazeDisplay v-bind:mazeID="mazeID"
                     v-bind:gameplayType="gameplayType" />
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
</style>
