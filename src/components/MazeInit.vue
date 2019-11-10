<template>
    <div class="maze-init__container" v-bind:class="{ 'maze-creation__form--hidden': isMazeCreated }">
        <form @submit="createMaze" class="maze-init__form">
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Maze width:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="mazeWidth" class="select-field">
                        <option v-for="width in widthHeight" :value="width" :key="width" :selected="width === mazeWidth">{{ width + ' cells' }}</option>
                    </select>
                </div>
            </div>
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Maze Height:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="mazeHeight" class="select-field">
                        <option v-for="height in widthHeight" :value="height" :key="height" :selected="height === mazeHeight">{{ height + ' cells' }}</option>
                    </select>
                </div>
            </div>
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Maze Difficulty:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="mazeDifficulty" class="select-field">
                        <option v-for="difficulty in difficultyOptions" :value="difficulty" :key="difficulty" :selected="difficulty === 0 ? 'selected' : ''">{{ difficulty }}</option>
                    </select>
                </div>
            </div>
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Maze Visualization Type:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="visualizationType" class="select-field">
                        <option value="non-graphical">Non graphical maze (printed maze from the API)</option>
                        <option value="graphical">Graphical maze</option>
                    </select>
                </div>
            </div>
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Pony Name:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="ponyName" class="select-field">
                        <option v-for="name in ponyNames" :value="name" :key="name" :selected="name === ponyName">{{ name }}</option>
                    </select>
                </div>
            </div>
            <div class="maze-init__form-item">
                <div class="maze-init__label">
                    <label>
                        Gameplay Type:
                    </label>
                </div>
                <div class="maze-init__select-container">
                    <select v-model="gameplayType" class="select-field">
                        <option value="manual">I will save {{ ponyName }} myself!</option>
                        <option value="auto">I will let the app saves {{ ponyName }}</option>
                    </select>
                </div>
            </div>

            <div class="maze-init__form-item maze-init__form-button">
                <input type="submit" value="Create A Maze!" class="" />
            </div>

        </form>
                <div v-bind:class="{ 'maze-creation__form--hidden': !isMazeCreated }">
                    <div>
                        <label>
                            Maze ID:
                        </label>
                        {{mazeID}}
                    </div>
                </div>
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
            visualizationType: 'graphical',
            minWH: 15,
            maxWH: 25,
            mazeWidth: 15,
            mazeHeight: 25,
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

    .maze-init__container {
        //border: 1px red solid;
        background-color: #f4f7f8;
    }

    .maze-init__form {
        border: 1px black solid;
    }

    .select-field {
        box-sizing: border-box;
        -webkit-box-sizing: border-box;
        -moz-box-sizing: border-box;
        border: 1px solid #C2C2C2;
        box-shadow: 1px 1px 4px #EBEBEB;
        -moz-box-shadow: 1px 1px 4px #EBEBEB;
        -webkit-box-shadow: 1px 1px 4px #EBEBEB;
        border-radius: 3px;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
        padding: 7px;
        outline: none;
        height: 3rem;
        background-color: #d2e6e2;
    }

    .select-field {
        width: 100%;
    }

    .select-field:focus {
        border: 1px solid #30ff00;
    }

    .maze-init__form {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .maze-init__form-item {
        width: 60%;
        display: flex;
        padding: 1%;
    }

    .maze-init__form-button {
        width: 100%;
        justify-content: space-around;
    }

    .maze-init__select-container {
        width: 70%;
    }

    .maze-init__label {
        width: 30%;
        text-align: left;
        padding-top: 0.5rem;
    }


</style>
