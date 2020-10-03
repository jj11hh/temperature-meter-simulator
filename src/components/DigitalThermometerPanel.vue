<template>
    <div class="panel">
        <h1 class="title">XXX型仿真精密数字测温仪</h1>
        <div class="nameplate">
            <p>CMC</p>
            <p>202009</p>
        </div>
        <div class="display">
            <h4 class="display-label">示值</h4>
            <LCDBar class="lcd-bar" :content="lcdify(f)" :powerOn="power"/>
            <h4 class="display-label">误差</h4>
            <LCDBar class="lcd-bar" :content="lcdify(Math.abs(f - T))" :powerOn="power"/>
        </div>
        <Oscilloscope
            class="oscilloscope"
            :width="250"
            :height="100"
            :range="[Math.min(...wave) , Math.max(...wave)]"
            :data="wave"
            :powerOn="power"
        />
        <LCDBar class="lcd-resistance" :content="lcdify(R_t)" :powerOn="power"/>
        <ThreePointOneKnob class="three-point-one" :value="T" unit="℃" @change="T=$event"/>
        <div class="button-set">
            <LED :powerOn="calibrating" lightColor="#5F5"/>
            <input type="button" value="校准" @click="calibrate">
            <input type="button" value="下载" @click="saveParameter">
            <input type="file" value="上传" @change="loadParameter">
        </div>

        <div class="power-indicator">
            <LED :powerOn="power" :lightColor="'#5F5'"/>
            <input type="button" value="电源" @click="powerSwitch">
        </div>

        <div class="knob-panel">
            <h4 class="knob-panel-title">参数设定</h4>

            <div class="knob-group">
                <span class="knob-label">M</span>
                <Knob class="knob" :min="1" :max="100" :step="1" :value="M" @change="M=$event"/>
                <span class="knob-value">{{M}}</span>
            </div>

            <div class="knob-group">
                <span class="knob-label">r</span>
                <Knob class="knob" :min="0" :max="10" :step="0.1" :value="r" @change="r=$event"/>
                <span class="knob-value">{{r.toFixed(1)}}Ω</span>
            </div>
            <div class="knob-group">
                <span class="knob-label">R <sub>0</sub> </span>
                <Knob class="knob" :min="0" :max="100" :step="0.1" :value="R_0" @change="R_0=$event"/>
                <span class="knob-value">{{R_0.toFixed(1)}}Ω</span>
            </div>
            <div class="knob-group">
                <span class="knob-label">R <sub>1</sub></span>
                <Knob class="knob" :min="0" :max="100" :step="0.1" :value="R_1" @change="R_1=$event"/>
                <span class="knob-value">{{R_1.toFixed(1)}}Ω</span>
            </div>
            <div class="knob-group">
                <span class="knob-label">R <sub>2</sub></span>
                <Knob class="knob" :min="0" :max="100" :step="0.1" :value="R_2" @change="R_2=$event"/>
                <span class="knob-value">{{R_2.toFixed(1)}}Ω</span>
            </div>
            <div class="knob-group">
                <span class="knob-label">I <sub>ref</sub></span>
                <Knob class="knob" :min="1" :max="10" :step="0.1" :value="I_ref" @change="I_ref=$event"/>
                <span class="knob-value">{{I_ref.toFixed(1)}}mA</span>
            </div>
            <div class="knob-group">
                <span class="knob-label">Gain</span>
                <Knob class="knob" :min="0" :max="5000" :step="10" :value="Gain" @change="Gain=$event"/>
                <span class="knob-value">{{Gain.toFixed(1)}}</span>
            </div>
        </div>

        <p class="name-label">黑龙江科技大学 测控17-3班 蒋易恒</p>
        <div class="text-v-ab">
            R <sub>t</sub>:{{R_t.toFixed(2)}}
            V <sub>AB</sub>:{{V_AB.toFixed(2)}}
            V <sub>(t)</sub>:{{(Gain * V_AB).toFixed(2)}}
            <div>
                T={{model[0].toFixed(3)}}v<sup>3</sup>+{{model[1].toFixed(3)}}v<sup>2</sup>+{{model[2].toFixed(3)}}v+{{model[3].toFixed(3)}}={{f.toFixed(3)}}
            </div>
        </div>
    </div>
</template>

<script>
import LED from './LED.vue'
import LCDBar from './LCDBar.vue'
import Oscilloscope from './Oscilloscope.vue'
import ThreePointOneKnob from './ThreePointOneKnob.vue'
import Knob from './Knob.vue'

import regression from 'regression';
import saveAs from '../saveFile.js';

function getDefaultData () {
    return {
        M: 10,
        r: 1.0,
        R_0: 100,
        R_1: 100,
        R_2: 100,
        I_ref: 1.0,
        Gain: 1000,
        T: 0,
        power: false,
        calibrating: false,
        model: [1, 1, 1, 1],
        wave: [...Array(11).keys()].map(x => Math.sin(((x-5)/10)*2*Math.PI))
    }
}

export default {
    data: getDefaultData,
    
    components: {
        LED,
        LCDBar,
        Oscilloscope,
        ThreePointOneKnob,
        Knob,
    },

    methods: {
        powerSwitch () {
            const power = ! this.power;

            if (! power) {
                Object.assign(this.$data, getDefaultData());
            }

            this.power = power;
        },
        calibrate () {
            if (! this.power) return;

            const asyncSleep = (time) => new Promise((resolve, _) => {
                setTimeout(resolve, time);
                _;
            });

            (async () => {
                const step = 300 / (this.M - 1);

                this.calibrating = true;

                const data = [];

                for (let i = 0; i < this.M; ++ i) {
                    this.T = i * step;
                    data.push([this.V_t, this.T]);
                    await asyncSleep(300);
                }

                const result = regression.polynomial(data, { 
                    order: 3,
                    precision: 18
                });

                console.log(result);
                console.log(data);

                this.model = result.equation;

                const graph = data.map(x => x[0]);
                this.wave = graph;

                this.calibrating = false;
            })();
        },
        lcdify (number) {
            if (isNaN(number)) {
                return "-----"
            }
            const width = number > 0 ? 6 : 5;
            const sign = number > 0 ? "" : "-";
            number = number > 0 ? number : -number;

            let str = `${number.toFixed(2)}`;
            if (str.length > width) {
                return "ERROR";
            }
            else if (str.length == width) {
                return sign + str;
            }
            else {
                return sign + "0".repeat(width - str.length) + str;
            }
        },
        saveParameter () {
            console.log(JSON.stringify(this.$data));
            saveAs("parameters.json", JSON.stringify(this.$data));
        },
        loadParameter ( event ) {
            const file = event.target.files[0];
            if (! file) return;

            const reader = new FileReader();
            reader.onload = (e) => {
                const contents = e.target.result;
                const parameter = JSON.parse(contents);

                Object.assign(this.$data, parameter);
            };

            reader.readAsText(file);
        }
    },

    computed: {
        R_t () {
            const A = 3.9083e-3;
            const B = -5.775e-7;
            return this.R_0 * (1 + A*this.T + B*this.T*this.T);
        },
        V_AB () {
            const R_N = 100.0;
            
            const R_left = this.r + R_N + this.R_1;
            const R_right = this.R_t + this.r + this.R_2;
            const R_both = 1 / (R_left + R_right);
            const V_1 = this.I_ref * R_both;
            const V_A = V_1 * (this.R_1 / R_left);
            const V_B = V_1 * (this.R_2 / R_right);
            return V_A - V_B;

        },

        V_t () {
            return this.Gain * this.V_AB;
        },

        f () {
            const [C, B, A, K] = this.model;
            return C * (this.V_t**3) + B * (this.V_t**2) + A * this.V_t + K;
        }
    }
}
</script>

<style scoped>
.panel {
    background-color: rgb(184, 222, 255);
    width: 400px;
    height: 650px;
    border: solid 2px black;
    position: relative;
}

h1.title {
    font-family: serif;
    font-size: 18pt;
}

.nameplate {
    position: absolute;
    background-color: white;
    width: 60px;
    height: 60px;
    left: 20px;
    top: 70px;
    border-radius: 30px;
    border: solid 2px black;
}

.nameplate p {
    margin: 0;
}

.display {
    position: absolute;
    left: 100px
}

h4.display-label {
    float: left;
    clear: both;
    margin-right: 0.5em;

}

.oscilloscope {
    position: absolute;
    left: 140px;
    top: 220px;
}

.lcd-resistance {
    position: absolute;
    left: 140px;
    top: 330px;
}

.lcd-bar {
    float: left;
}

.three-point-one {
    position: absolute;
    left: 140px;
    top: 410px;
}

.button-set {
    position: absolute;
    left: 140px;
    top: 500px;
    border: solid 2px black;
    width: 186px;
}

.power-indicator {
    position: absolute;
    left: 344px;
    top: 500px;
    border: solid 2px black;
}

.name-label {
    position: absolute;
    left: 140px;
    top: 570px;
}

.text-v-ab {
    position: absolute;
    left: 140px;
    top:600px;
}

.knob-panel {
    position: absolute;
    left: 10px;
    top: 190px;
    width: 120px;
    height: 450px;
    background-color: rgb(141, 183, 251);
    border: solid 2px black;
}

.knob-panel-title {
    margin-top: 0;
    margin-bottom: 0;
}

.knob-panel .knob {
    display: inline-block;
}

.knob-group {
    text-align: left;
}

.knob-label {
    width: 1.6em;
    font-size: 10pt;
    display: inline-block;
    text-align: right;
}

.knob-value {
    width: 2em;
    font-size: 8pt;
}
</style>