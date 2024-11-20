<script setup>
import { ref, onMounted, reactive } from 'vue';
import PieSocket from "piesocket-js";

const canvas = ref(null);
const state = reactive({
  drawing: false,
  context: null,
  lastX: 0,
  lastY: 0
});

const startDrawing = (e) => {
  state.drawing = true;
  [state.lastX, state.lastY] = [e.offsetX, e.offsetY];
};

const draw = (e) => {
  if (!state.drawing) return;
  broadcastDraw(state.lastX, state.lastY, e.offsetX, e.offsetY);
  drawLine(state.lastX, state.lastY, e.offsetX, e.offsetY);
  [state.lastX, state.lastY] = [e.offsetX, e.offsetY];
};

const stopDrawing = () => {
  state.drawing = false;
};
 
let broadcastDraw; 
const pieSocket = new PieSocket({
    clusterId: "free.blr2",
    apiKey: "B9UKgvptNTWrZxfCUTquFp7nKVsYqu2LtmBao5Jg",
});

pieSocket.subscribe("drawing-room").then(ch => {
  const channel = ch;
  console.log("Channel is ready");

  channel.listen("draw", (data) => {
    const { x0, y0, x1, y1 } = JSON.parse(data);
    drawLine(x0, y0, x1, y1);
  });

  broadcastDraw = (x0, y0, x1, y1) => {
    channel.publish("draw", JSON.stringify({ x0, y0, x1, y1 }));
  };
}).catch(error => {
  console.error("Error subscribing to PieSocket channel:", error);
});

const drawLine = (x0, y0, x1, y1) => {
  state.context.beginPath();
  state.context.moveTo(x0, y0);
  state.context.lineTo(x1, y1);
  state.context.stroke();
};

onMounted(() => {
  canvas.value.width = window.innerWidth;
  canvas.value.height = window.innerHeight;
  state.context = canvas.value.getContext('2d');
  state.context.strokeStyle = "black";
  state.context.lineJoin = 'round';
  state.context.lineCap = 'round';
  state.context.lineWidth = 10;
});
</script>

<template>
    <div>
      <canvas ref="canvas"
              @mousedown="startDrawing"
              @mousemove="draw"
              @mouseup="stopDrawing"
              @mouseout="stopDrawing"></canvas>
    </div>
  </template>


<style scoped>
canvas {
  border: 5px solid black;
}
</style>
