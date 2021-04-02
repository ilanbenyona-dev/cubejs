<template>

    <div class="cube" :style="style">
        <div class="cube__face front">front</div>
        <div class="cube__face back">back</div>
        <div class="cube__face right">right</div>
        <div class="cube__face left">left</div>
        <div class="cube__face top">top</div>
        <div class="cube__face bottom">bottom</div>
    </div>


</template>

<script>

import lerp from 'gl-vec3/lerp';
import recomposeMat4 from 'mat4-recompose';
import decomposeMat4 from 'mat4-decompose';
import determinant from 'gl-mat4/determinant';
import slerp from 'quat-slerp';

function decompose(m, state) {
    if (determinant(m) === 0)
        return false
    return decomposeMat4(m, state.translate, state.scale, state.skew, state.perspective, state.quaternion)
}

function state() {
    return {
        translate: vec3(),
        scale: vec3(1),
        skew: vec3(),
        perspective: vec4(),
        quaternion: vec4()
    }
}

function vec3(n) {
    return [n||0,n||0,n||0]
}

function vec4() {
    return [0,0,0,1]
}

var state0 = state();
var state1 = state();
var tmpState  = state();

function interpolate(mat_start, mat_end) {
    // var out = vec4();
    var r0 = decompose(mat_start, state0);
    var r1 = decompose(mat_end, state1);
    var valid = r1 && r0;
    if (!valid)
        return false;

    return function (out, alpha) {
        // Now lerp/slerp the start and end components into a temporary state
        lerp(tmpState.translate, state0.translate, state1.translate, alpha);
        lerp(tmpState.skew, state0.skew, state1.skew, alpha);
        lerp(tmpState.scale, state0.scale, state1.scale, alpha);
        lerp(tmpState.perspective, state0.perspective, state1.perspective, alpha);
        slerp(tmpState.quaternion, state0.quaternion, state1.quaternion, alpha);

        console.log(out, alpha);
        // And recmpose into 'out' matrix 
        recomposeMat4(out, tmpState.translate, tmpState.scale, tmpState.skew, tmpState.perspective, tmpState.quaternion);

        return out;
    }
}
function findPos(obj) {
    let rect = obj.getBoundingClientRect();
    return [rect.x, rect.y];
}



// #####################################
// ##                                 ##
// ##                                 ##
// ##         MATH FUNCTIONS          ##
// ##                                 ##
// ##                                 ##
// ####################################


// Normalization recalculates all coordinates in a way that the resulting vector has a length of "1". 
// We achieve this by dividing the x, y and z-coordinates by the vector's length 
function normalize(vect){   
    var length = Math.sqrt( vect[0]*vect[0] + vect[1]*vect[1] + vect[2]*vect[2] );
    
    vect[0]/= length;
    vect[1]/= length;
    vect[2]/= length;

    return vect;
}

function calcMatrix(vector, angle){
    // calculate transformation-matrix from a vector[x,y,z] and an angle
    var x       = vector[0],
        y       = vector[1],
        z       = vector[2],
        sin     = Math.sin(angle),
        cos     = Math.cos(angle),
        cosmin  = 1-cos,
        matrix  = [(cos+x*x*cosmin), (y*x*cosmin+z*sin),(z*x*cosmin-y*sin),0,
                    (x*y*cosmin-z*sin), (cos+y*y*cosmin), (z*y*cosmin+x*sin),0,
                    (x*z*cosmin+y*sin), (y*z*cosmin-x*sin), (cos+z*z*cosmin),0,
                    0,0,0,1];
                                    
    return matrix;
}

// Calculate the angle between 2 vectors.
function calcAngle(vect_1, vect_2){
    var numerator   =   vect_1[0]*vect_2[0] + vect_1[1]*vect_2[1] + vect_1[2]*vect_2[2],
        denominator =   Math.sqrt(vect_1[0]*vect_1[0] + vect_1[1]*vect_1[1] + vect_1[2]*vect_1[2])*
                        Math.sqrt(vect_2[0]*vect_2[0] + vect_2[1]*vect_2[1] + vect_2[2]*vect_2[2]),
        angle       =   Math.acos(numerator/denominator);
    
    return angle;
}
//Some stupid matrix-multiplication. 
function multiplyMatrix(m1, m2){
    var matrix = [];

    matrix[0]   = m1[0]*m2[0]+m1[1]*m2[4]+m1[2]*m2[8]+m1[3]*m2[12];
    matrix[1]   = m1[0]*m2[1]+m1[1]*m2[5]+m1[2]*m2[9]+m1[3]*m2[13];
    matrix[2]   = m1[0]*m2[2]+m1[1]*m2[6]+m1[2]*m2[10]+m1[3]*m2[14];
    matrix[3]   = m1[0]*m2[3]+m1[1]*m2[7]+m1[2]*m2[11]+m1[3]*m2[15];
    matrix[4]   = m1[4]*m2[0]+m1[5]*m2[4]+m1[6]*m2[8]+m1[7]*m2[12];
    matrix[5]   = m1[4]*m2[1]+m1[5]*m2[5]+m1[6]*m2[9]+m1[7]*m2[13];
    matrix[6]   = m1[4]*m2[2]+m1[5]*m2[6]+m1[6]*m2[10]+m1[7]*m2[14];
    matrix[7]   = m1[4]*m2[3]+m1[5]*m2[7]+m1[6]*m2[11]+m1[7]*m2[15];
    matrix[8]   = m1[8]*m2[0]+m1[9]*m2[4]+m1[10]*m2[8]+m1[11]*m2[12];
    matrix[9]   = m1[8]*m2[1]+m1[9]*m2[5]+m1[10]*m2[9]+m1[11]*m2[13];
    matrix[10]  = m1[8]*m2[2]+m1[9]*m2[6]+m1[10]*m2[10]+m1[11]*m2[14];
    matrix[11]  = m1[8]*m2[3]+m1[9]*m2[7]+m1[10]*m2[11]+m1[11]*m2[15];
    matrix[12]  = m1[12]*m2[0]+m1[13]*m2[4]+m1[14]*m2[8]+m1[15]*m2[12];
    matrix[13]  = m1[12]*m2[1]+m1[13]*m2[5]+m1[14]*m2[9]+m1[15]*m2[13];
    matrix[14]  = m1[12]*m2[2]+m1[13]*m2[6]+m1[14]*m2[10]+m1[15]*m2[14];
    matrix[15]  = m1[12]*m2[3]+m1[13]*m2[7]+m1[14]*m2[11]+m1[15]*m2[15];

    return matrix;
}

function getCoords(eventObj) {
    return [eventObj.pageX, eventObj.pageY];
}
function logMatrix(matrix) {
    console.log(`${matrix[0]}, ${matrix[1]}, ${matrix[2]}, ${matrix[3]}`);
     console.log(`${matrix[4]}, ${matrix[5]}, ${matrix[6]}, ${matrix[7]}`);
     console.log(`${matrix[8]}, ${matrix[9]}, ${matrix[10]}, ${matrix[11]}`);
     console.log(`${matrix[12]}, ${matrix[13]}, ${matrix[14]}, ${matrix[15]}`);
}

export default {
    mounted: function () {
        const self = this;
        this.$el.ref = 'abc';
        this.stage   = this.$parent.$el;
        this.box     = this.$el;
        this.pos     = findPos(this.stage);
        this.angle   = 0;
        this.impulse = 0;

        // Determine radius for our virtual trackball
        this.h       = this.stage.offsetHeight / 2;
        this.w       = this.stage.offsetWidth / 2;
        this.radius  = this.h < this.w ? this.h : this.w;

        this.startMatrix = calcMatrix(this.axis, this.angle);

        window.addEventListener('keydown', function (params) {
            self.backToInitialState();
        })
    },
    data: function () {
        return {
            // vector: [1,1,0], //FRONT (only rotates xAxis, yAxis)
            xAxis: 0,
            yAxis: 0,
            zAxis: 0,
            speed: 8,
                radius: 0,  // Radius of the virtual trackball
                stage: null, // The DOM-container of the Cube ('this.$parent')
                axis : [0,1,0],  // The rotation axis
                pointerDownVect: [], // pointer vector on click
                pointerMoveVect: [], // Pointer vector on movement
                startMatrix: null, // Initial transformation-matrix  
                delta : 0, 
                impulse: 0, 
                pos: 0, 
                w: 0, 
                h: 0, 
                decr: 0, 
                angle: 0, 
                oldAngle: 0, 
                oldTime: 0, 
                currTime: 0,
                flag: 0,
                stopMatrix: [],
            size: 50
        }
    },
    computed: {
        style: function() {
            // console.log(`rotate3d(${this.axis}, ${this.angle}rad) matrix3d(${this.startMatrix})`);
            console.log(`matrix3d(${this.startMatrix})`);
            console.log(this.flag);
            return {
                transform: `rotate3d(${this.axis}, ${this.angle}rad) matrix3d(${this.startMatrix})`
            };
        }
    },
    methods: {
        onClick(e) {
            if(this.delta !== 0){this.stopSlide(); }
            e.preventDefault();

            this.pointerDownVect = this.calcZvector(getCoords(e));
            this.oldTime         = this.currTime = new Date().getTime();
            this.oldAngle        = 0;
            return 0;
        },
        onRotateStart() {

            return 0;
        },
        onRotate(e) {
            var coords = getCoords(e);
            this.oldTime  = this.currTime;
            this.oldAngle = this.angle;

            // Calculate the current z-component of the 3d-vector on the virtual trackball
            this.pointerMoveVect = this.calcZvector(coords);

            // Retrive current rotation-axis from the calculated z-vectors on pointerdown and on pointermove
            // We do so by finding the normalized cross-product of the pointerDown & pointerMove vector
            this.axis[0] = this.pointerDownVect[1]*this.pointerMoveVect[2] - this.pointerDownVect[2]*this.pointerMoveVect[1];
            this.axis[1] = this.pointerDownVect[2]*this.pointerMoveVect[0] - this.pointerDownVect[0]*this.pointerMoveVect[2];
            this.axis[2] = this.pointerDownVect[0]*this.pointerMoveVect[1] - this.pointerDownVect[1]*this.pointerMoveVect[0];
            this.axis    = normalize(this.axis);

            // Calculate the angle of the rotation between pointerDownVect and pointerMoveVect
            this.angle = calcAngle(this.pointerDownVect, this.pointerMoveVect);

            this.currTime = new Date().getTime();
            return 20;
        },
        onRotateEnd() {        

            this.calcSpeed();
            if(  this.delta > 0){
                requestAnimationFrame(this.slide);
            }else if(!(isNaN(this.axis[0]) || isNaN(this.axis[1]) || isNaN(this.axis[2]))) {
                this.stopSlide();
            }

            // let t = [], s = [], sk = [], p = [], q = [];
            // var stopMatrix = calcMatrix(this.axis, this.angle);
            // DecomposeMat4(stopMatrix, t,s,sk,p,q);
            // logMatrix(stopMatrix);
            // console.log( t, s, sk, p, q);
            return 0;
        },
        backToInitialState() {
            let self = this;
            var out = vec4();
            console.log(this.startMatrix);
            let lerp = interpolate(this.startMatrix, [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1]);
            var counter = 10;
            var newYearCountdown = setInterval(function(){
            console.log(out, 1 - counter / 10);
            counter--;
            lerp(out, 1 - counter / 10);
            self.startMatrix = out;
            console.log(self.setMateix(out));
            if (counter === 0) {
                console.log("HAPPY NEW YEAR!!");
                clearInterval(newYearCountdown);
            }
            }, 1000);
            newYearCountdown();
        },
        calcSpeed() {
            var dw = this.angle - this.oldAngle,
                dt = this.currTime - this.oldTime;

            this.delta = Math.abs(dw*17/dt);

            if (isNaN(this.delta)) {
                this.delta = 0;
            } else if (this.delta > 0.2) {
                this.delta = 0.2;
            }
            
        },
        cleanupMatrix() {
        
            // Clean up when finishing rotation. Only thing to do: create a new "initial" matrix for the next rotation.
            // If we don't, the object will flip back to the position at launch every time the user starts dragging.
            // Therefore we must:
            // 1. calculate a matrix from axis and the current angle
            // 2. Create a new startmatrix by combining current startmatrix and stopmatrix to a new matrix. 
            // let lolMatrix = calcMatrix([0,1,0], 0);
            // Matrices can be combined by multiplication, so what are we waiting for?
            this.stopMatrix  = calcMatrix(this.axis, this.angle);
            // console.log(stopMatrix);
            // logMatrix(stopMatrix)
            this.startMatrix = multiplyMatrix(this.startMatrix,this.stopMatrix);
            // console.log(this.startMatrix);
            // logMatrix(this.startMatrix);
        
        },
        setMateix(mat) {
            this.startMatrix = mat;
            console.log(`matrix3d(${this.startMatrix})`);
            this.flag++;
            return mat;
        },
        slide() {
            this.angle += this.delta;
            this.decr  = 0.01*Math.sqrt(this.delta);
            this.delta = this.delta > 0 ? this.delta-this.decr : 0;
                        
            if (this.delta === 0){
                this.stopSlide();
            }else{
                requestAnimationFrame(this.slide);
            }
            
        },
        stopSlide() {
            // cancelAnimationFrame(this.slide);
            this.cleanupMatrix();
            this.oldAngle = this.angle = 0;
            this.delta    = 0;
        },
        click: function (){
            // console.log('asd');
        },
        rotateX(qX) {
            this.xAxis += qX;
        },
        rotateY(qY) {
            this.yAxis += qY;
        },
        rotateZ(qZ) {
            this.zAxis += qZ;
        },
        setVector(v) {
            this.vector = v;
        },
        // This function will calculate a z-component for our 3D-vector from the mouse x and y-coordinates 
        // (the corresponding point on our virtual trackball):
        calcZvector: function (coords){
            var x       = coords[0] - this.pos[0],
                y       = coords[1] - this.pos[1],
                vector  = [(x/this.radius-1), (y/this.radius-1)],
                z       = 1 - vector[0]*vector[0] - vector[1]*vector[1];
                
            // Make sure that dragging stops when z gets a negative value:
            vector[2]   = z > 0 ? Math.sqrt(z) : 0;

            return vector;
        }

    }
}
</script>

<style scoped>

.wrapper {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

.cube {
    position: absolute;
    margin: 50px;
    width: 100px;
    height: 100px;
    transform-style: preserve-3d;
    /* transform:  rotate3d(1,1,1,180deg); */
    /* transform: perspective(40000px); */
    /* perspective: 6000px; */
    /* transform-origin: left; */
    /* perspective-origin: 0 7550px; */
        /* transform-style: flat; */
    /*browsers are bitches at z-sorting css3d-transforms. Use backface-visibility: hidden as often as you can! */
    backface-visibility: hidden;

}

.cube__face {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    position: absolute;
    backface-visibility: inherit;
    font-size: 20px;
    color: #fff;
}

.front {
    background: rgba(90,90,90,.7);
    transform: translateZ(50px);
    /* z-index: 5; */
}

.back {
    background: rgba(0,210,0,.7);
    transform: rotateY(180deg) translateZ(50px);
}

.right {
    background: rgba(210,0,0,.7);
    transform: rotateY(90deg) translateZ(50px);
}

.left {
    background: rgba(0,0,210,.7);
    transform: rotateY(-90deg) translateZ(50px);
}

.top {
    background: rgba(210,210,0,.7);
    transform: rotateX(90deg) translateZ(50px);
}

.bottom {
    background: rgba(210,0,210,.7);
    transform: rotateX(-90deg) translateZ(50px);
}


</style>