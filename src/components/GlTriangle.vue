<script setup lang="ts">
import * as glMatrix from 'gl-matrix'
import { computed, onMounted, ref } from 'vue'
import vertexShaderText from './gl-triangle/shaders/vertexShader.glsl?raw'
import fragmentShaderText from './gl-triangle/shaders/fragmentShader.glsl?raw'

let gl: WebGLRenderingContext
let program: WebGLProgram

const vertices = [0.0, 0.5, 0.0, -0.5, -0.5, 0.0, 0.5, -0.5, 0.0]
const colors = [
  [0.0, 0.0, 0.0], // white
  [1.0, 0.0, 0.0], // red
  [0.0, 1.0, 0.0], // green
  [0.0, 0.0, 1.0], // blue
]

const reverseColor = ref(false)
const direct = ref(1)
const fixTime = ref(0)
const angle = ref(0)

const matrix = {
  world: new Float32Array(16),
  proj: new Float32Array(16),
  view: new Float32Array(16),
}

const ucolor = computed(() => {
  let deg = angle.value * 180 / Math.PI
  while (deg > 360) deg -= 360

  const index = Math.floor(deg / 90)

  return reverseColor.value ? [...colors].reverse()[index] : colors[index]
})

function getGlContext() {
  const canvas = document.querySelector('#triangle') as HTMLCanvasElement

  gl = canvas.getContext('webgl2')!

  if (!gl) {
    console.warn('WebGL2 not supported, falling back on webgl')
    gl = canvas.getContext('webgl')!
  }

  if (!gl)
    console.warn('WebGL not supported')
}

function createShaders() {
  const vertexShader = gl.createShader(gl.VERTEX_SHADER)!
  const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)!

  gl.shaderSource(vertexShader, vertexShaderText)
  gl.shaderSource(fragmentShader, fragmentShaderText)

  gl.compileShader(vertexShader)
  if (!gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)) {
    console.error('ERROR compiling vertex shader!', gl.getShaderInfoLog(vertexShader))
    return
  }

  gl.compileShader(fragmentShader)
  if (!gl.getShaderParameter(fragmentShader, gl.COMPILE_STATUS)) {
    console.error('ERROR compiling fragment shader!', gl.getShaderInfoLog(fragmentShader))
    return
  }

  program = gl.createProgram()!
  gl.attachShader(program, vertexShader)
  gl.attachShader(program, fragmentShader)
  gl.linkProgram(program)

  if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
    console.error('ERROR linking program!', gl.getProgramInfoLog(program))
    return
  }

  gl.validateProgram(program)
  if (!gl.getProgramParameter(program, gl.VALIDATE_STATUS))
    console.error('ERROR validating program!', gl.getProgramInfoLog(program))
}

function bindBuffer() {
  const triangleVertexBufferObject = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexBufferObject)
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW)
}

function render(now: number) {
  now *= 0.001
  const deltaTime = now - fixTime.value
  fixTime.value = now

  if (angle.value >= 2 * Math.PI)
    angle.value -= 2 * Math.PI

  angle.value += Math.PI * deltaTime / 5 // 5 seconds full rotation

  glMatrix.mat4.rotate(
    matrix.world,
    matrix.world,
    angle.value,
    [0, direct.value, 0],
  )

  const matWorldUniformLocation = gl.getUniformLocation(program, 'mWorld')
  gl.uniformMatrix4fv(matWorldUniformLocation, false, matrix.world)

  const colorLocation = gl.getUniformLocation(program, 'fragColor')

  gl.uniform3fv(colorLocation, ucolor.value)

  gl.clearColor(0.5, 0.5, 0.5, 1.0)
  // gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT)

  gl.drawArrays(gl.TRIANGLES, 0, 3)

  requestAnimationFrame(render)
}

function setViewport() {
  const width = (gl.canvas as HTMLCanvasElement).clientWidth
  const height = (gl.canvas as HTMLCanvasElement).clientHeight
  gl.canvas.width = width
  gl.canvas.height = height
  gl.viewport(0, 0, width, height)
}

function play() {
  getGlContext()
  createShaders()
  bindBuffer()
  setViewport()

  const positionAttribLocation = gl.getAttribLocation(program, 'vertPosition')

  gl.vertexAttribPointer(
    positionAttribLocation,
    3,
    gl.FLOAT,
    false,
    3 * Float32Array.BYTES_PER_ELEMENT,
    0,
  )
  gl.enableVertexAttribArray(positionAttribLocation)

  gl.useProgram(program)

  const matViewUniformLocation = gl.getUniformLocation(program, 'mView')
  const matProjUniformLocation = gl.getUniformLocation(program, 'mProj')

  glMatrix.mat4.identity(matrix.world)
  glMatrix.mat4.lookAt(matrix.view, [0, 0, 2], [0, 0, 0], [0, 1, 0])
  glMatrix.mat4.perspective(
    matrix.proj,
    Math.PI / 4,
    (gl.canvas as HTMLCanvasElement).clientWidth / (gl.canvas as HTMLCanvasElement).clientHeight,
    0.1,
    1000.0,
  )

  gl.uniformMatrix4fv(matViewUniformLocation, false, matrix.view)
  gl.uniformMatrix4fv(matProjUniformLocation, false, matrix.proj)

  requestAnimationFrame(render)
}

onMounted(() => {
  play()
})
</script>

<template>
  <!-- <div id="app"> -->
  <canvas id="triangle">
    Your browser doesn't appear to support the HTML5 <code>&lt;this.$refs.canvas&gt;</code> element.
  </canvas>
  <div class="row justify-around">
    <q-chip clickable color="secondary" text-color="white" icon="directions" @click="direct = -direct">
      direction
    </q-chip>
    <q-chip clickable color="secondary" text-color="white" icon="color_lens" @click="reverseColor = !reverseColor">
      reverse colors
    </q-chip>
  </div>
</template>

<style scoped>
canvas {
  width: 50vw;
  height: 50vh;
}
</style>
