<template>
    <div class="main-container">
        <div class="first-col">
            <input v-on:click="movePony()"
                   :disabled="isButtonDisabled"
                   type="submit"
                   value="next"
                   class="direction-button" />
        </div>
    </div>
</template>

<script>
export default {
    name: 'AutoPanel',
    props: ['escapePath'],
    data() {
        return {
            moveCounter: 0,
            isButtonDisabled: false
        }
    },
    methods: {
        movePony() {
            if (this.escapePath.length <= this.moveCounter) {
                this.isButtonDisabled = true;
                return;
            }

            const direction = this.escapePath[this.moveCounter].direction;
            this.$emit('get-ponyMove', direction);
            this.moveCounter++
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