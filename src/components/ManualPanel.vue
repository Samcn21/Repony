<template>
    <div class="main-container">
        <div class="first-col">
            <input v-on:click="movePony('west')"
                   v-bind:class="{ 'disabled': disableButton('west') }"
                   :disabled="disableButton('west')"
                   type="submit"
                   value="west"
                   class="direction-button" />
        </div>
        <div class="second-col">
            <div>
                <input v-on:click="movePony('north')"
                       v-bind:class="{ 'disabled': disableButton('north') }"
                       :disabled="disableButton('north')"
                       type="submit"
                       value="north"
                       class="direction-button" />
            </div>
            <div>
                <input v-on:click="movePony('south')"
                       v-bind:class="{ 'disabled': disableButton('south') }"
                       :disabled="disableButton('south')"
                       type="submit"
                       value="south"
                       class="direction-button" />
            </div>
        </div>
        <div class="third-col">
            <input v-on:click="movePony('east')"
                   v-bind:class="{ 'disabled': disableButton('east') }"
                   :disabled="disableButton('east')"
                   type="submit"
                   value="east"
                   class="direction-button" />
        </div>
    </div>
</template>

<script>
export default {
    name: 'ManualPanel',
    props: ['ponyDirections'],
    methods: {
        movePony(direction) {
            this.$emit('get-ponyMove', direction);
        },
        disableButton(direction) {
            return !this.ponyDirections[direction];
        }
    }
}
</script>

<style lang="scss" scoped>
    .main-container {
        display: flex;
        flex-direction: row;
        //width: 500px;
        justify-content: space-around;
    }

    .direction-button {
        cursor: pointer;
        width: 100px;
        height: 40px;
    }

    .disabled {
        cursor: auto;
        //border: 2px dashed red;
    }
</style>