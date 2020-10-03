<template>
    <div class="container">
        <SegLCD 
            class="in-a-row" 
            v-for="i in digits.length" 
            :key="i" 
            :digit="digits[i-1].digit" 
            :dot="digits[i-1].dot" 
        />    
    </div>
</template>

<script>
import SegLCD from './SegLCD.vue'
export default {
    name: 'LCDBar',
    props: {
        content: {
            type: String,
            required: true
        },
        powerOn: {
            type: Boolean,
            default: true,
        }
    },
    components: {
        SegLCD
    },

    computed: {
        digits: function () {
            if (! this.powerOn) {
                return Array(5).fill({digit: " ", dot: false});
            }
            let digits = [];
            for (const char of this.content) {
                if (char != '.'){
                    digits.push({
                        digit: char,
                        dot: false
                    })
                }
                else if (digits.length > 0) {
                    digits[digits.length - 1].dot = true;
                }
            }

            return digits;
        }
    }
}
</script>

<style scoped>
.in-a-row {
    display: inline-block;
}
</style>