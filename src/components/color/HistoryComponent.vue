<template>
    <div class="history-wrap">
        <div class="history" v-for="color in colors" :key="color" :style="{backgroundColor: color}"></div>
    </div>
</template>

<script>
export default {
    props: {
        colors: {
            type: Array,
            default: () => ['#daadda']
        }
    },
    data() {
        return {
            colorHistory: JSON.parse(localStorage.getItem(this.colorsHistoryKey)) || []
        }
    }
}
</script>

<style lang="scss">
.history-wrap {
    // display: flex;
    width: 90%;
    margin: auto;
    .history {
        display: inline-block;
        width: 16px;
        height: 16px;
        border-radius: 3px;
        margin-left: 10px;
        margin-right: 10px;
        margin-top: 10px;
    }
}

</style>
