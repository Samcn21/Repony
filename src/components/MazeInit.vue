<template>
    <div>
        <form @submit="createMaze" class="">
            <div class="">
                <div v-bind:class="{ 'maze-creation__form--hidden': isMazeCreated }" >
                    <div>
                        <label>
                            Maze width:
                        </label>
                        <select v-model="mazeWidth">
                            <option v-for="width in widthHeight" :value="width" :key="width" :selected="width === mazeWidth">{{ width }}</option>
                        </select>
                    </div>
                    <div>
                        <label>
                            Maze Height:
                        </label>
                        <select v-model="mazeHeight">
                            <option v-for="height in widthHeight" :value="height" :key="height" :selected="height === mazeHeight">{{ height }}</option>
                        </select>
                    </div>
                    <div>
                        <label>
                            Maze Difficulty:
                        </label>
                        <select v-model="mazeDifficulty">
                            <option v-for="difficulty in difficultyOptions" :value="difficulty" :key="difficulty" :selected="difficulty === 0 ? 'selected' : ''">{{ difficulty }}</option>
                        </select>
                    </div>
                    <div>
                        <label>
                            Pony Name:
                        </label>
                        <select v-model="ponyName">
                            <option v-for="name in ponyNames" :value="name" :key="name" :selected="name === ponyName">{{ name }}</option>
                        </select>
                    </div>
                    <div>
                        <label>
                            Gameplay Type:
                        </label>
                        <select v-model="gameplayType">
                            <option value="manual">I will save {{ ponyName }} myself</option>
                            <option value="auto">I will let the app saves {{ ponyName }}</option>
                        </select>
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
import fetch from 'node-fetch';

export default {
    name: 'MazeInit',
    data() {
        return {
            widthHeight: [],
            difficultyOptions: [],
            ponyNames: [
                'Twilight Sparkle',
                'Pinkie Pie',
                'Rarity',
                'Rainbow Dash',
                'Fluttershy',
                'Applejack',
                'Princess Celestia',
                'Apple Bloom',
                'Sweetie Belle',
                'Princess Luna',
                'Spike',
                'Derpy Hooves',
                'Cheerilee',
            ],
            gameplayType: 'auto',
            minWH: 15,
            maxWH: 25,
            mazeWidth: 15,
            mazeHeight: 15,
            ponyName: 'Applejack',
            mazeDifficulty: 0,
            mazeID: '',
            isMazeCreated: false,
            mazeIDtest: {
                maze_id: '' //'3128cc30-a257-459f-bacc-70df2edeed57'
            }
        }
    },
    mounted() {
        for (let i = this.minWH; i <= this.maxWH; i++) {
            this.widthHeight.push(i);
        }

        for (let i = 0; i <= 10; i++) {
            this.difficultyOptions.push(i);
        }

        this.ponyNames.sort();
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
            this.$emit('get-mazeData', this.mazeID, this.ponyName, this.gameplayType);
        }
    }
}

</script>

<style lang="scss" scoped>
    .maze-creation__form--hidden {
        display: none;
    }
</style>