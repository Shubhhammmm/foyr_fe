<template>
  <div class="whiteboard">
    <canvas id="canvas"></canvas>
    <ToolbarComponent @setTool="setTool" />
  </div>
</template>

<script>
import * as fabric  from "fabric";
import io from "socket.io-client";
import ToolbarComponent from "./ToolbarComponent.vue";

export default {
  props: ['user'],
  components: {
    ToolbarComponent,
  },
  data() {
    return {
      canvas: null,
      socket: null,
      currentTool: 'pencil',
    };
  },
  mounted() {
    this.setupCanvas();
    this.setupSocket();
  },
  methods: {
    setupCanvas() {
      this.canvas = new fabric.Canvas('canvas', {
        width: 800,
        height: 600,
        backgroundColor: '#fff',
      });

      this.canvas.isDrawingMode = true;
      this.canvas.on('path:created', this.sendDrawing);
    },
    setupSocket() {
      this.socket = io('https://foyr-be.onrender.com');

      this.socket.on('connect', () => {
        this.socket.emit('join', { user: this.user });
      });

      this.socket.on('drawing', (data) => {
        fabric.util.enlivenObjects([data], (objects) => {
          objects.forEach((obj) => {
            this.canvas.add(obj);
          });
        });
      });
    },
    sendDrawing(event) {
      const drawing = event.path;
      this.socket.emit('drawing', drawing);
    },
    setTool(tool) {
      this.currentTool = tool;
      this.canvas.isDrawingMode = tool === 'pencil';
    },
  },
};

</script>
