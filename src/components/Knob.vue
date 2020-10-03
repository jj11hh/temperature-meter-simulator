<template>
    <div 
        class="container" 
        v-bind:style="containerStyle" 
        v-on:mousedown="mousedown"
        v-on:mousemove="mousemove"
        v-on:wheel="handleWheel"
    >
        <div class="handle" v-bind:style="handleStyle"></div>
    </div>
</template>

<script>

function getRotation(event) {
    let rect = event.target.getBoundingClientRect();
    let [x, y] = [event.clientX - rect.left - (rect.width/2), event.clientY - rect.top - (rect.height/2)];
    return Math.atan2(y, x)
}

export default {
    name: 'Knob',
    data: function () {
        return {
            isRotating: false,
            startAngle: 0,
            lastRotation: 0
        }
    },
    methods: {
        mousedown: function (event) {
            if (event.buttons & 0x01) {
                this.isRotating = true;
                this.startAngle = getRotation(event);
                this.lastRotation = this.rotation;
            }
        },
        mousemove: function (event) {
            if (! (event.buttons & 0x01)) {
                this.isRotating = false;
            }
            if (this.isRotating) {
                const oldvalue = this.value;
                let rotation = getRotation(event) - this.startAngle;

                if (rotation < Math.PI * 2) {
                    rotation += Math.PI * 2;
                }

                rotation += this.lastRotation;
                rotation %= Math.PI * 2;

                const rough_value = (rotation / (Math.PI * 2)) * this.span
                const steps = Math.round(rough_value / this.step) % Math.round(this.span / this.step)
                const value = steps * this.step + this.min;

                if (value != oldvalue)
                    this.$emit('change', value);
            }
        },
        handleWheel: function (event) {
            const wheelY = event.deltaY;
            let step = Math.round((this.value - this.min) / this.step);
            if (wheelY > 0) {
                step = (step + 1) % (Math.round(this.span / this.step) + 1);
            }
            else if (wheelY < 0) {
                step --;
                if (step < 0) step = Math.round(this.span / this.step);
            }
            this.$emit('change', step * this.step + this.min);
            event.preventDefault();
        }
    },
    props: {
        min: Number,
        max: Number,
        step: Number,
        value: Number,
        disabled: {
            type: Boolean,
            default: false
        },
        color: {
            type: String,
            default: "#4444FF"
        }
    },
    computed: {
        span: function () {
            return this.max - this.min;
        },
        containerStyle: function () {
            return {
                backgroundColor: this.color,
                transform: `rotate(${this.rotation/(2*Math.PI)*360}deg)`
            }
        },
        handleStyle: function() {
            return {
                backgroundColor: this.color
            }
        },
        rotation: function () {
            return ((this.value - this.min) / (this.max - this.min + this.step)) * 2*Math.PI;
        }
    }
}
</script>

<style scoped>

.container {
    border-width: 4px;
    width: 50px;
    height: 50px;
    border-radius: 25px;
    position: relative;
}

.handle {
    position: absolute;
    top: 5px;
    left: 20px;
    width: 10px;
    height: 10px;
    border-radius: 5px;

    filter:brightness(300%);
}

</style>