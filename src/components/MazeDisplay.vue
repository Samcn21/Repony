<template>
    <div class="maze-container" v-bind:class="{ 'container--hidden': isHidden }">
        <!-- <div>
            <form @submit="refreshMaze" class="">
                <input type="submit" value="Refresh" class="" />
            </form>
        </div> -->
        <ManualPanel class="manual-panel"
                        v-bind:class="{'container--hidden': hideManualPanel}"
                        v-on:get-ponyMove="movePony"
                        v-bind:ponyDirections="ponyDirections"
                        />

        <AutoPanel class="auto-panel"
                        v-bind:class="{'container--hidden': hideAutoPanel}"
                        v-bind:escapePath="escapePath"
                        v-on:get-ponyMove="movePony"
                        />

        <div class="container--hidden">
            <form @submit="testMethod" class="">
                <label>
                    test:
                </label>
                <input type="text" placeholder="25" v-model="testValue" class="" />
                <input type="submit" value="test!!!" class="" />
            </form>
        </div>
        <br/>
        <pre>{{mazePrint}}</pre>
    </div>
</template>

<script>
import ManualPanel from '../components/ManualPanel.vue'
import AutoPanel from '../components/AutoPanel.vue'
import axios from 'axios';
import fetch from 'node-fetch';

export default {
    name: 'MazeDisplay',
    props: ['mazeID', 'gameplayType'],
    components: {
        ManualPanel,
        AutoPanel
    },
    data() {
        return {
            isHidden: true,
            hideManualPanel: true,
            hideAutoPanel: true,
            mazeCells: [],
            mazeWalls: [],
            cellsPossibleDirections: [],
            escapePath: [],
            passedPath: [],
            domokunCell: 0,
            ponyCell: 0,
            endCell: 0,
            mazeWidth: 0,
            mazeHeight: 0,
            mazePrint: '',
            testValue: 0,
            ponyDirections: {
                north: false,
                south: false,
                east: false,
                west: false
            },
            isOver: false
        }
    },
    watch: {
        mazeID: async function (value) {
            if (value) {
                this.isHidden = false;
                const mazeData = await this.sendGetRequest(value);
                this.extractData(mazeData);
                this.mazePrint = await this.sendGetRequest(value, true);

                if (this.gameplayType === 'auto') {
                    this.hideAutoPanel = false;
                    this.saveThePony();
                } else {
                    this.hideManualPanel = false;
                    this.updatePonyDirections();
                }
            }
        }
    },
    methods: {
        async refreshMaze(e) {
            if (e) {
                e.preventDefault();
            }
            const mazeId = this.mazeID;
            const mazeData = await this.sendGetRequest(mazeId);
            this.extractData(mazeData, true);
            this.mazePrint = await this.sendGetRequest(mazeId, true);
        },
        async sendGetRequest(mazeId, isPrint) {
            let baseURL = `https://ponychallenge.trustpilot.com/pony-challenge/maze/${mazeId}`;
            const url = isPrint ? `${baseURL}/print` : baseURL;

            try {
                const response = await axios.get(url);
                return response.data;
            } catch (error) {
                console.error(error);
            }
        },
        async sendPOSTRequest(mazeId, direction) {
            const url = `https://ponychallenge.trustpilot.com/pony-challenge/maze/${mazeId}`;
            try {
                const response = await fetch(url, {
                    method: 'POST',
                    body: JSON.stringify({
                        "direction": direction
                    }),
                    headers: {
                        'Content-Type': 'application/json'
                    },
                });
                return await response.json();
            } catch (error) {
                console.log('an error occured fetching maze id: ' + error);
                return '';
            }
        },
        async movePony(direction) {
            const gameStateData = await this.sendPOSTRequest(this.mazeID, direction);
            const gameResult = this.getGameResult(gameStateData);

            if (gameResult === 'active') {
                await this.refreshMaze();
                await this.updatePonyDirections();
            } else {
                this.updatePonyDirections(true);
                this.isOver = true;
            }
        },
        getGameResult(data) {
            if (data.state === 'won') {
                console.log('you saved the pony: ' + data['hidden-url']);
                return 'win';
            }

            if (data.state === 'over') {
                console.log('pony is dead: ' + data['hidden-url']);
                return 'lost';
            }

            return 'active';
        },
        extractData(mazeData, isRefresh) {
            //getting Domokun, Pony and End-point cell index
            this.domokunCell = mazeData.domokun[0];
            this.ponyCell = mazeData.pony[0];
            this.endCell = mazeData['end-point'][0];

            //getting the maze data for the first time
            if (!isRefresh) {
                // set data
                this.mazeCells = mazeData.data;
                this.mazeWidth = mazeData.size[0];
                this.mazeHeight = mazeData.size[1];

                // finding all walls beside north and west which are East and South
                // finding the possible moves for each cell (directions with no walls for each cell)
                this.processMazeCells();
            }
        },
        processMazeCells() {
            //let cloneMazeCells = this.mazeCells.slice(0); //https://davidwalsh.name/javascript-clone-array
            let cloneMazeCells = JSON.parse(JSON.stringify(this.mazeCells)); //https://stackoverflow.com/questions/31344041/how-to-unbind-an-array-copy-in-vue-js
            let cellsPossibleDirections = [];
            const width = this.mazeWidth;
            const height = this.mazeHeight;
            const directions = ['north', 'south', 'east', 'west'];

            // Looping through all cells
            for (let i = 0; i < cloneMazeCells.length; i++) {
                // there is a wall in the south direction
                if (!this.canGoSouth(i, cloneMazeCells)) {
                    cloneMazeCells[i].push('south');
                }

                // there is a wall in the east direction
                if (!this.canGoEast(i, cloneMazeCells)) {
                    cloneMazeCells[i].push('east');
                }

                // when the cell missing any wall from the 4 directions it means that direction is a path
                // those directions that are not in the list will be added to the cell as possible directions
                const possibleMoves = this.getPossibleMoves(cloneMazeCells[i]);
                cellsPossibleDirections.push(possibleMoves);
            }

            this.mazeWalls = cloneMazeCells;
            this.cellsPossibleDirections = cellsPossibleDirections;
        },
        getPossibleMoves(mazeCell) {
            const directions = ['north', 'south', 'east', 'west'];
            const result = directions.filter(e => !mazeCell.includes(e));
            return result;
        },
        updatePonyDirections(isOver) {
            const ponyLocation = this.ponyCell;
            const mazeWalls = this.mazeWalls;

            this.ponyDirections.north = isOver ? false : this.canGoNorth(ponyLocation, mazeWalls);
            this.ponyDirections.south = isOver ? false : this.canGoSouth(ponyLocation, mazeWalls);
            this.ponyDirections.east = isOver ? false : this.canGoEast(ponyLocation, mazeWalls);
            this.ponyDirections.west = isOver ? false : this.canGoWest(ponyLocation, mazeWalls);
        },
        updateMazeWall(index) {
            this.mazeWalls[index] = [];

            // there is a wall in the north direction
            if (!this.canGoNorth(index, this.mazeWalls)) {
                this.mazeWalls[index].push('north');
            }

            // there is a wall in the south direction
            if (!this.canGoSouth(index, this.mazeWalls)) {
                this.mazeWalls[index].push('south');
            }

            // there is a wall in the east direction
            if (!this.canGoEast(index, this.mazeWalls)) {
                this.mazeWalls[index].push('east');
            }

            // there is a wall in the west direction
            if (!this.canGoWest(index, this.mazeWalls)) {
                this.mazeWalls[index].push('west');
            }

            this.cellsPossibleDirections[index] = this.getPossibleMoves(this.mazeWalls[index]);
        },
        saveThePony() {
            // looping through all maze cells to apply dead end filling
            for (let i = 0; i < this.mazeCells.length; i++) {
                this.deadEndFilling(i);
            }

            // finding the escape path for the Pony
            this.findEscapePath(this.ponyCell);
        },
        findEscapePath(index) {
            // adding the cell to the blocked list
            this.passedPath.push(index);

            const cellsPossibleDirections = this.cellsPossibleDirections[index];

            // looping through all possible directions (first cell is 1 possible direction and the rest are 2 possible directions)
            for (let i = 0; i < cellsPossibleDirections.length; i++) {
                const nextDestinationCell = this.getDestinationCell(index, cellsPossibleDirections[i]);

                // the path that is not passed yet
                if (!this.hasElement(this.passedPath, nextDestinationCell)) {
                    const ponyPath = {
                        index: index,
                        direction: cellsPossibleDirections[i],
                        request: false
                    };
                    this.escapePath.push(ponyPath);

                    // invoking the same function (recursively) for the next cell
                    this.findEscapePath(nextDestinationCell);
                }
            }
        },
        deadEndFilling(index) {
            // this cell is Pony cell or End-point cell
            if (index === this.ponyCell || index === this.endCell) {
                return;
            }

            const cellsPossibleDirections = this.cellsPossibleDirections[index];

            // there is no possible way to move (all 4 directions are walls)
            if (cellsPossibleDirections.length === 0) {
                return;
            }

            // if the cell is a dead end (there is only one possible direction to move)
            if (cellsPossibleDirections.length === 1) {
                const nextDestinationCell = this.getDestinationCell(index, cellsPossibleDirections[0]);

                // adding the wall (the possible direction) to the cell, because we don't need this cell anymore (blocked)
                this.mazeWalls[index].push(cellsPossibleDirections[0]);

                // updating the possible directions of this cell (which is no direction)
                this.cellsPossibleDirections[index] = [];

                // updating the walls and the possible directions of the next destination cell
                this.updateMazeWall(nextDestinationCell);

                // invoking the same function (recursively) for the next cell
                this.deadEndFilling(nextDestinationCell);
            }
        },
        isValidCellIndex(cell, mazeCellsNumber) {
            return mazeCellsNumber - 1 < cell || cell < 0 ? false : true;
        },
        hasElement(array, element) {
            return array.indexOf(element) === -1 ? false : true;
        },
        getDestinationCell(currentCell, direction) {
            let result = -1;

            // if cannot move to the destination cells it returns -1
            if (!this.hasElement(this.cellsPossibleDirections[currentCell], direction)) {
                return result;
            }

            // find the destination from the direction of the movement
            switch(direction) {
                case 'north':
                    // symmetric cell of the previous row
                    result = currentCell - this.mazeWidth;
                    break;
                case 'south':
                    // symmetric cell of the next row
                    result = currentCell + this.mazeWidth;
                    break;
                case 'east':
                    // next cell
                    result = currentCell + 1;
                    break;
                case 'west':
                    // previous cell
                    result = currentCell - 1;
                    break;
            }

            return result;
        },
        canGoNorth(cell, mazeCells) {
            const mazeCellsNumber = this.mazeCells.length;
            const width = this.mazeWidth;

            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            if (this.isFirstRow(cell, width)) {
                return false;
            }

            return this.hasPreRowSouthWall(cell, mazeCells, width) ? false : true;
        },
        canGoSouth(cell, mazeCells) {
            const mazeCellsNumber = this.mazeCells.length;
            const width = this.mazeWidth;
            const height = this.mazeHeight;

            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            if (this.isLastRow(cell, mazeCellsNumber, width, height)) {
                return false;
            }

            // if the next row same column cell has north wall
            return this.hasNextRowNorthWall(cell, mazeCells, width) ? false : true;
        },
        canGoEast(cell, mazeCells) {
            const mazeCellsNumber = this.mazeCells.length;
            const width = this.mazeWidth;

            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            if (this.isLastColumn(cell, width, mazeCellsNumber)) {
                return false;
            }

            const nextCellIndex = cell + 1;

            // if the next cell has west wall
            return this.hasElement(mazeCells[nextCellIndex], 'west') ? false : true;
        },
        canGoWest(cell, mazeCells) {
            const mazeCellsNumber = this.mazeCells.length;
            const width = this.mazeWidth;

            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            if (this.isFirstColumn(cell, width)) {
                return false;
            }

            const previousCellIndex = cell - 1;

            // if the previous cell has east wall
            return this.hasElement(mazeCells[previousCellIndex], 'east') ? false : true;
        },
        hasPreRowSouthWall(cell, mazeCells, width) {
            const preRowSouthWallIndex = cell - width;
            return this.hasElement(mazeCells[preRowSouthWallIndex], 'south');
        },
        hasNextRowNorthWall(cell, mazeCells, width) {
            const nextRowNorthWallIndex = cell + width;
            return this.hasElement(mazeCells[nextRowNorthWallIndex], 'north');
        },
        isFirstRow(cell, width) {
            return cell < width;
        },
        isLastRow(cell, cellNumbers, width, height) {
            const lastRowFirstCellIndex = (width * (height - 1)) -1;
            return cell > lastRowFirstCellIndex && cell < cellNumbers ? true : false;
        },
        isFirstColumn(cell, width) {
            return cell % width === 0;
        },
        isLastColumn(cell, width, cellNumbers) {
            return width - 1 === cell % width && cell < cellNumbers;
        },
        testMethod(e) {
            e.preventDefault();
            let value = parseInt(this.testValue, 10);
            const h = Math.floor(value / this.mazeWidth) + 1;
            const v = value % this.mazeWidth + 1;
            console.log('cell: ' + value + ' => row: ' + h + ' ,column: ' + v);
            //console.log(this.canGoWest(value, this.mazeWalls));

            //console.log(this.getDestinationCell(value, 'west'));
            this.saveThePony();
        }
    }
}
</script>

<style lang="scss" scoped>
    .maze-container {
        margin-top: 3rem;
    }

    .container--hidden {
        display: none;
    }
</style>