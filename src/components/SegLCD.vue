<template>
    <div class="container">
        <div 
            class="segment"
            v-for="i in range(8)"
            v-bind:key="`${i}`"
            v-bind:class="{'light-on': segsOn[i]}">
        </div>
    </div>
</template>

<script>
export default {
    name: 'SegLCD',
    props: {
        digit: {
            type: String,
            default: '-'
        },
        dot: {
            type: Boolean,
            default: false,
        },
        powerOn: {
            type: Boolean,
            default: true
        },
    },
    methods: {
        range: function (n) {
            let r = [];
            for (let i = 0; i < n; ++ i) {
                r.push(i);
            }
            return r;
        }
    },
    computed: {
        segsOn: function () {
            if (! this.powerOn) {
                return [false] * 8;
            }

            /**
             *    0
             *  -----
             * |1 3  |2
             *  -----
             * |4 5  |6
             *  -----
             *       . 7
             */

            const digits = [
                [true, true, true, false, true, true, true],
                [false, false, true, false, false, false, true],
                [true, false, true, true, true, true, false],
                [true, false, true, true, false, true, true],
                [false, true, true, true, false, false, true],
                [true, true, false, true, false, true, true],
                [true, true, false, true, true, true, true],
                [true, false, true, false, false, false, true],
                [true, true, true, true, true, true, true],
                [true, true, true, true, false, true, true],
            ];

            const symbols = {
                'A': [true, true, true, true, true, false, true],
                'B': [true, true, false, true, true, true, true],
                'C': [true, true, false, false, true, true, false],
                'D': [true, false, true, true, true, true, true],
                'E': [true, true, false, true, true, true, false],
                'F': [true, true, false, true, false, false, false],
                'G': [true, true, true, true, false, true, true],
                'H': [false, true, true, true, true, false, true],
                'I': [false, false, true, false, false, false, true],
                'J': [false, false, true, false, true, true, false],
                'L': [false, true, false, false, true, true, false],
                'O': [false, false, false, true, true, true, true],
                'P': [true, true, true, true, true, false, false],
                'Q': [true, true, true, true, false, false, true],
                'R': [false, false, false, true, true, false, false],
                'S': digits[5],
                'U': [false, true, true, true, true, true, true],
                'V': [false, false, false, true, true, true, true],
                '-': [false, false, false, true, false, false, false],
                ' ': [false, false, false, false, false, false, false],
            };

            let segs;

            if (/^[0-9]/.test(this.digit)) {
                const index = this.digit.charCodeAt(0) - '0'.charCodeAt(0);
                segs = digits[index];
            }
            else if (this.digit in symbols) {
                segs = symbols[this.digit];
            }
            else {
                segs = symbols['-'];
            }

            return segs.concat([this.dot]);
        }
    }
}
</script>

<style scoped>
.container {
    position: relative;
    width: 50px;
    height: 70px;
    background-color: black;
}
.segment {
    position: absolute;
    background-color: rgb(47, 47, 47);
}

.segment:nth-child(1) {
    top: 5px;
    left: 10px;
    width: 30px;
    height: 5px;
}

.segment:nth-child(2) {
    top: 10px;
    left: 5px;
    width: 5px;
    height: 20px;
}

.segment:nth-child(3) {
    top: 10px;
    left: 40px;
    width: 5px;
    height: 20px;
}

.segment:nth-child(4) {
    top: 30px;
    left: 10px;
    width: 30px;
    height: 5px;
}

.segment:nth-child(5) {
    top: 35px;
    left: 5px;
    width: 5px;
    height: 20px;
}

.segment:nth-child(6) {
    top: 55px;
    left: 10px;
    width: 30px;
    height: 5px;
}

.segment:nth-child(7) {
    top: 35px;
    left: 40px;
    width: 5px;
    height: 20px;
}

.segment:nth-child(8) {
    top: 60px;
    left: 45px;
    width: 5px;
    height: 5px;
}

.light-on {
    background-color: rgb(107, 160, 209) !important;
}
</style>