<template>
    <div>
        <form @submit="createMaze" class="">
            <div class="">
                <div v-bind:class="{ 'maze-creation__form--hidden': isMazeCreated }" >
                    <div>
                        <label>
                            Maze width:
                        </label>
                        <input type="text" placeholder="15" v-model="mazeWidth" class="" />
                    </div>
                    <div>
                        <label>
                            Maze Height:
                        </label>
                        <input type="text" placeholder="25" v-model="mazeHeight" class="" />
                    </div>
                    <div>
                        <label>
                            Pony Name:
                        </label>
                        <input type="text" placeholder="applejack" v-model="ponyName" class="" />
                    </div>
                    <div>
                        <label>
                            Maze Difficulty:
                        </label>
                        <input type="text" placeholder="0" v-model="mazeDifficulty" class="" />
                    </div>

                    <input type="submit" value="Create A Maze!" class="" />
                </div>

                <div v-bind:class="{ 'maze-creation__form--hidden': !isMazeCreated }">
                    <div>
                        <label>
                            Maze ID:
                        </label>
                        {{mazeID}}
                    </div>
                </div>
            </div>
        </form>

    </div>
</template>

<script>
//import axios from 'axios';
import fetch from 'node-fetch';

export default {
    name: 'MazeInit',
    async created() {
        //await this.createMaze();
    },
    data() {
        return {
            mazeWidth: 15,
            mazeHeight: 25,
            ponyName: 'applejack',
            mazeDifficulty: 0,
            mazeID: '',
            isMazeCreated: false,
            mazeIDtest: {
                maze_id: '3128cc30-a257-459f-bacc-70df2edeed57'
            }
        }
    },
    methods: {
        async createMaze(e) {
            e.preventDefault();

            // for testing already existing maze id
            if (this.mazeIDtest.maze_id) {
                this.getMazeId(this.mazeIDtest);
                return;
            }

            const url = 'https://ponychallenge.trustpilot.com/pony-challenge/maze';
            try {
                const response = await fetch(url, {
                    method: 'POST',
                    body: JSON.stringify({
                        "maze-width": parseInt(this.mazeWidth, 10),
                        "maze-height": parseInt(this.mazeHeight, 10),
                        "maze-player-name": this.ponyName,
                        "difficulty": parseInt(this.mazeDifficulty, 10)
                    }),
                    headers: {
                        'Content-Type': 'application/json'
                    },
                });
                this.getMazeId(await response.json());
            } catch (error) {
                console.log('an error occured fetching maze id: ' + error);
                return '';
            }
        },
        getMazeId(value) {
            this.mazeID = value.maze_id;
            this.isMazeCreated = true;
            this.$emit('get-mazeData', this.mazeID, this.ponyName);
        }
    }
}

</script>

<style lang="scss" scoped>
    .maze-creation__form--hidden {
        display: none;
    }
</style>