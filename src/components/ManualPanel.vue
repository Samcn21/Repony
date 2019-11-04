<template>
    <div class="main-container">
        <div class="first-col">
            <div>
                <input v-on:click="movePony('west')" type="submit" value="west" :disabled="disableButton('west')" />
            </div>
        </div>
        <div class="second-col">
            <div>
                <input v-on:click="movePony('north')" type="submit" value="north" v-bind:class="{ 'disabled': disableButton('north') }" :disabled="disableButton('north')" />
            </div>
            <div>
                <input v-on:click="movePony('south')" type="submit" value="south" :disabled="disableButton('south')" />
            </div>
        </div>
        <div class="third-col">
            <input v-on:click="movePony('east')" type="submit" value="east" :disabled="disableButton('east')" />
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

    .disabled {
        border: 2px dashed red;
    }
</style>