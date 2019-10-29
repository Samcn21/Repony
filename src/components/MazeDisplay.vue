<template>
    <div class="maze-container" v-bind:class="{ 'maze-container--hidden': isHidden }">
        <div>
            <form @submit="refreshMaze" class="">
                <input type="submit" value="Refresh" class="" />
            </form>
        </div>
        <div>
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
import axios from 'axios';

export default {
    name: 'MazeDisplay',
    props: ['mazeID'],
    components: {

    },
    data() {
        return {
            isHidden: false,
            mazeCells: [],
            mazeWalls: [],
            cellsPossibleDirections: [],
            deadEndCells: [],
            blockCells: [],
            domokunCell: 0,
            ponyCell: 0,
            endCell: 0,
            mazeWidth: 0,
            mazeHeight: 0,
            mazePrint: '',
            testValue: 0
        }
    },
    watch: {
        mazeID: async function (value) {
            if (value) {
                this.isHidden = false;
                const mazeData = await this.sendGetRequest(value);
                this.extractData(mazeData);
                this.mazePrint = await this.sendGetRequest(value, true);
            }
        }
    },
    methods: {
        async refreshMaze(e) {
            e.preventDefault();
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
        extractData(mazeData, isRefresh) {
            //getting Domokun, Pony and End-point cell index
            this.domokunCell = mazeData.domokun[0];
            this.ponyCell = mazeData.pony[0];
            this.endCell = mazeData['end-point'][0];

            //getting the maze data for the first time
            //if (!isRefresh) {
                // set data
                this.mazeCells = mazeData.data;
                this.mazeWidth = mazeData.size[0];
                this.mazeHeight = mazeData.size[1];

                // finding all walls beside north and west which are East and South
                this.mazeWalls = this.getEastAndSouthWalls();

                // finding the possible moves for each cell (directions with no walls for each cell)
                this.cellsPossibleDirections = this.getCellsPossibleDirections();

                // finding the dead end cells (cells with 3 walls except Pony and Endpoint cells)
                this.deadEndCells = this.getDeadEndCells();

                // a clone of dead end cells for they are already in the block list (no way to go)
                // more data will be added to this array
                this.blockCells = this.deadEndCells.slice(0); //https://davidwalsh.name/javascript-clone-array
            //}
        },
        getEastAndSouthWalls() {
            let cloneMazeCells = JSON.parse(JSON.stringify(this.mazeCells)); //https://stackoverflow.com/questions/31344041/how-to-unbind-an-array-copy-in-vue-js
            const width = this.mazeWidth;
            const height = this.mazeHeight;
            for (let i = 0; i < cloneMazeCells.length; i++) {
                if (!this.canGoSouth(i, cloneMazeCells, cloneMazeCells.length, width, height)) {
                    cloneMazeCells[i].push('south');
                }

                if (!this.canGoEast(i, cloneMazeCells, cloneMazeCells.length, width)) {
                    cloneMazeCells[i].push('east');
                }
            }

            return cloneMazeCells;
        },
        getCellsPossibleDirections() {
            let result = [];
            const width = this.mazeWidth;
            const height = this.mazeHeight;
            const mazeWalls = this.mazeWalls;
            const directions = ['north', 'south', 'east', 'west'];
            for (let i = 0; i < mazeWalls.length; i++) {
                // when the cell missing any wall from the 4 directions it means that direction is a path
                // those directions that are not in the list will be added to the cell as possible directions
                let possibleMoves = directions.filter(e => !mazeWalls[i].includes(e));
                result.push(possibleMoves);
            }

            return result;
        },
        updateCellPossibleDirections(index) {
            let cellPossibleDirections = this.cellsPossibleDirections[index];
            const cellPossibleDirectionNumber = cellPossibleDirections.length;

            for (let i = 0; i < cellPossibleDirectionNumber; i++) {
                // the destination cell that can go from the current cell
                const destinationCell = this.getDestinationCell(index, cellPossibleDirections[i]);

                // if the destination cell is blocked we remove the direction toward the destination cell from the current cell
                if (this.hasElement(this.blockCells, destinationCell)) {
                    const arrayIndex = this.cellsPossibleDirections[index].indexOf(cellPossibleDirections[i]);
                    if (arrayIndex !== -1) {
                        this.cellsPossibleDirections[index].splice(arrayIndex, 1);
                    }
                }
            }
        },
        getDeadEndCells() {
            let result = [];
            const mazeWalls = this.mazeWalls;
            for (let i = 0; i < mazeWalls.length; i++) {
                // deadend cells are the cells with 3 walls.
                // if the Pony cell or End point cell are located in a deadend cell, that cell should be excluded.
                // the rest are the dead end cells.
                if (mazeWalls[i].length > 2 && (i !== this.ponyCell || i !== this.endCell)) {
                    result.push(i);
                }
            }

            return result;
        },
        getDeadEndNeighborCell(index) {
            console.log(index);
            // make a function to test if this is a dead end
            if (this.cellsPossibleDirections[index].length === 1) {
                const nextDestinationCell = this.getDestinationCell(index, this.cellsPossibleDirections[index][0]);

                //console.log(nextDestinationCell);
                if (nextDestinationCell === -1) {
                    console.log('destination is impossible');
                }

                return nextDestinationCell;
            }
        },
        getPossibleMoves(cell) {
            const mazeCells = this.mazeCells;
            const mazeCellsNumber = mazeCells.length;
            const width = this.mazeWidth;
            const height = this.mazeWidth;

            let canGoNorth = this.canGoNorth(cell, mazeCells, mazeCellsNumber);
            console.log('can go north: ' + canGoNorth);

            let canGoSouth = this.canGoSouth(cell, mazeCells, mazeCellsNumber, width, height);
            console.log('can go south: ' + canGoSouth);

            let canGoEast = this.canGoEast(cell, mazeCells, mazeCellsNumber, width);
            console.log('can go east: ' + canGoEast);

            let canGoWest = this.canGoWest(cell, mazeCells, mazeCellsNumber);
            console.log('can go west: ' + canGoWest);
        },
        findThePath() {

            // adding pony cell to the block cells
            if (!this.hasElement(this.blockCells, this.ponyCell)) {
                this.blockCells.push(this.ponyCell);
            }

            // adding pony cell to the block cells
            if (!this.hasElement(this.blockCells, this.endCell)) {
                this.blockCells.push(this.endCell);
            }
            //this.blockCells.push(15);
            //this.updateCellPossibleDirections(16);
            for (let i = 0; i < this.deadEndCells.length; i++) {
                this.getDeadEndPath(this.deadEndCells[i]);

                // update neighbor
                this.updateCellPossibleDirections(this.getDeadEndNeighborCell(this.deadEndCells[i]));
            }

            // for (let i = 0; i < this.mazeCells.length; i++) {
            //     this.getDeadEndPath(i);
            // }
        },
        getDeadEndPath(index) {
            console.log('-------------------');
            console.log('get dead end ' + index);
            // if there is one way to continue

            //console.log('where cell ' + index + ' can go: ' + this.cellsPossibleDirections[index]);
            if (this.cellsPossibleDirections[index].length === 1) {
                const nextDestinationCell = this.getDestinationCell(index, this.cellsPossibleDirections[index][0]);

                //console.log(nextDestinationCell);
                if (nextDestinationCell === -1) {
                    console.log('destination is impossible');
                }

                if (!this.hasElement(this.blockCells, nextDestinationCell)) {
                    console.log('destination cell is not blocked, will be blocked and call the function again: ', nextDestinationCell);
                    this.blockCells.push(nextDestinationCell);
                    this.updateCellPossibleDirections(nextDestinationCell);
                    this.getDeadEndPath(nextDestinationCell);
                } else {
                    console.log('destination cell is blocked, continue with other cells' , nextDestinationCell);
                }
            // it is a junstion maybe put in junction array
            } else {
                console.log('this is a junction, stop here');
            }
        },

        isValidCellIndex(cell, mazeCellsNumber) {
            return mazeCellsNumber - 1 < cell || cell < 0 ? false : true;
        },
        hasElement(array, element) {
            return array.indexOf(element) === -1 ? false : true;
        },
        removeArrayElement(array, value) {
            return array.filter((element) => {element !== value});
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
                    // symmetric cell of the upper row
                    result = currentCell - this.mazeWidth;
                    break;
                case 'south':
                    // symmetric cell of the lower row
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
        canGoNorth(cell, mazeCells, mazeCellsNumber) {
            return this.canGoNorthOrWest(cell, mazeCells, mazeCellsNumber, true);
        },
        canGoSouth(cell, mazeCells, mazeCellsNumber, width, height) {
            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            if (this.isLastRow(cell, mazeCellsNumber, width, height)) {
                return false;
            }

            // if the next row same column cell has north wall
            return this.hasNextRowNorthWall(cell, mazeCells, width) ? false : true;
        },
        canGoEast(cell, mazeCells, mazeCellsNumber, width) {
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
        canGoWest(cell, mazeCells, mazeCellsNumber) {
            return this.canGoNorthOrWest(cell, mazeCells, mazeCellsNumber);
        },
        canGoNorthOrWest(cell, mazeCells, mazeCellsNumber, isNorth) {
            const direction = isNorth ? 'north' : 'west';

            if (!this.isValidCellIndex(cell, mazeCellsNumber)) {
                return false;
            }

            return !this.hasElement(mazeCells[cell], direction);
        },
        hasNextRowNorthWall(cell, mazeCells, width) {
            const nextRowNorthWallIndex = cell + width;
            return this.hasElement(mazeCells[nextRowNorthWallIndex], 'north');
        },
        isLastRow(cell, cellNumbers, width, height) {
            const lastRowFirstCellIndex = (width * (height - 1)) -1;
            return cell > lastRowFirstCellIndex && cell < cellNumbers ? true : false;
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

            //let possibleMoves = this.getPossibleMoves(value);
            //console.log(this.getDestinationCell(value, 'west'));
            this.findThePath();
        }
    }
}
</script>

<style lang="scss" scoped>
    .maze-container {
        margin-top: 3rem;
    }

    .maze-container--hidden {
        display: none;
    }
</style>