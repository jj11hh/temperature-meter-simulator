<template>
    <div class="container-three">
        <Knob class="knob" :max="9" :min="0" :step="1" :value="hundreds" @change="setHundreds"/>
        <Knob class="knob" :max="9" :min="0" :step="1" :value="tens" @change="setTens"/>
        <Knob class="knob" :max="9" :min="0" :step="1" :value="ones" @change="setOnes"/>
        <div class="dot-symbol"></div>
        <Knob class="knob" :max="9" :min="0" :step="1" :value="dotOnes" @change="setDotOnes"/>
        <h4> {{ `${hundreds}${tens}${ones}.${dotOnes}${unit}` }} </h4>
    </div>
</template>

<script>
import Knob from './Knob'
export default {
    props: {
        value: Number,
        unit: {
            type: String,
            default: ""
        }
    },
    methods: {
        setHundreds (hundreds) {
            this.$emit('change', +`${hundreds}${this.tens}${this.ones}.${this.dotOnes}`);
        },
        setTens (tens) {
            this.$emit('change', +`${this.hundreds}${tens}${this.ones}.${this.dotOnes}`);
        },
        setOnes (ones) {
            this.$emit('change', +`${this.hundreds}${this.tens}${ones}.${this.dotOnes}`);
        },
        setDotOnes (dotOnes) {
            this.$emit('change', +`${this.hundreds}${this.tens}${this.ones}.${dotOnes}`);
        }
    },
    components: {
        Knob
    },
    computed: {
        hundreds () {
            return Math.floor(this.value / 100) % 10;
        },
        tens () {
            return Math.floor(this.value / 10) % 10;
        },
        ones () {
            return Math.floor(this.value) % 10;
        },
        dotOnes () {
            return Math.floor(this.value * 10) % 10;
        }
    }
}
</script>

<style scoped>

.container-three {
    width: 246px;
    height: 80px;
    border: solid 2px black;
}

.knob {
    float: left;
    margin: 5px;
}

.dot-symbol {
    margin-top: 40px;
    width: 6px;
    height: 6px;
    background: black;
    border-radius: 3px;
    float: left;
}

</style>