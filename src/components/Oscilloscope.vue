<template>
    <canvas ref="my-canvas" :width="width" :height="height"> </canvas>
</template>

<script>
export default {
    props: {
        width: Number,
        height: Number,
        range: Array,
        data: Array,
        backgroundColor: {
            type: String,
            default: "black"
        },
        lineColor: {
            type: String,
            default: "#5F5"
        },
        powerOn: {
            type: Boolean,
            default: true
        }
    },
    methods: {
        render () {
            const ctx = this.$refs['my-canvas'].getContext('2d');
            ctx.beginPath();
            ctx.rect(0, 0, this.width, this.height);
            ctx.fillStyle = this.backgroundColor;
            ctx.fill();
            ctx.closePath();

            const len = this.data.length
            const spacing = this.width / (len-1);

            if (len == 0) return;
            if (! this.powerOn) return;

            const mapPoint = ([x, y]) => {
                const x_ = x * spacing;
                const y_ = this.height - (y - this.range[0]) * (this.height/(this.range[1] - this.range[0]));
                //console.log(`${x_}, ${y_}`);
                return [x_, y_];
            };

            // start
            ctx.beginPath();
            ctx.strokeStyle = this.lineColor;
            ctx.moveTo(...mapPoint([0, this.data[0]]))

            for (let i = 0; i < len; ++ i) {
                ctx.lineTo(...mapPoint([i, this.data[i]]));
            }
            ctx.stroke();
            ctx.closePath();
        }
    },

    mounted() {this.render()},

    watch: {
        range() {this.render()},
        width() {this.render()},
        height() {this.render()},
        data() {this.render()},
        background() {this.render()},
        lineColor() {this.render()},
        powerOn() {this.render()},
    }
}
</script>