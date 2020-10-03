<template>
    <div class="led-box" v-bind:style="{ backgroundColor: this.currentColor }">

    </div>
</template>

<script>
export default {
    name: "LED",
    props: {
        lightColor: String,
        powerOn: Boolean
    },
    computed: {
        currentColor: function () { 
            return this.powerOn ? this.lightColor : "grey"
        }
    }
}
</script>

<style scoped>
.led-box {
    border-style: solid;
    border-color: black;
    border-width: 2px;
    border-radius: 1em;
    width: 1em;
    height: 1em;
    background-color: grey;
}
</style>