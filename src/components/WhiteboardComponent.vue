<template>
  <div class="whiteboard">
    <canvas id="canvas"></canvas>
    <ToolbarComponent @setTool="setTool" />
  </div>
</template>

<script>
import * as fabric from "fabric";
import io from "socket.io-client";
import ToolbarComponent from "./ToolbarComponent.vue";

export default {
  props: ["user"],
  components: {
    ToolbarComponent,
  },
  data() {
    return {
      canvas: null,
      socket: null,
      currentTool: "pencil",
      isDrawing: false, // To control custom shapes like rectangle
      drawingObject: null, // For handling custom shapes
    };
  },
  mounted() {
    this.setupCanvas();
    this.setupSocket();
  },
  methods: {
    setupCanvas() {
      this.canvas = new fabric.Canvas("canvas", {
        width: 800,
        height: 600,
        backgroundColor: "#fff", // Changed background color to white
      });

      this.canvas.isDrawingMode = true; // Enable drawing by default (pencil)
      this.canvas.on("path:created", this.sendDrawing);

      // Handle drawing rectangle or other shapes
      this.canvas.on("mouse:down", this.onMouseDown);
      this.canvas.on("mouse:move", this.onMouseMove);
      this.canvas.on("mouse:up", this.onMouseUp);
    },
    setupSocket() {
      this.socket = io("https://foyr-be.onrender.com");

      this.socket.on("connect", () => {
        this.socket.emit("join", { user: this.user });
      });

      this.socket.on("drawing", (data) => {
        fabric.util.enlivenObjects([data], (objects) => {
          objects.forEach((obj) => {
            this.canvas.add(obj);
          });
        });
      });
    },
    sendDrawing(event) {
      const drawing = event.path;
      this.socket.emit("drawing", drawing);
    },
    setTool(tool) {
      this.currentTool = tool;
      this.canvas.isDrawingMode = tool === "pencil"; // Enable freehand drawing only if the tool is pencil

      // If not pencil, disable freehand drawing mode
      if (tool !== "pencil") {
        this.canvas.isDrawingMode = false;
      }
    },
    // Mouse event handlers for custom shapes (e.g., rectangle)
    onMouseDown(event) {
      if (this.currentTool === "rectangle") {
        const pointer = this.canvas.getPointer(event.e);
        this.isDrawing = true;

        // Create a rectangle and store its reference
        this.drawingObject = new fabric.Rect({
          left: pointer.x,
          top: pointer.y,
          width: 0,
          height: 0,
          fill: "transparent",
          stroke: "black",
          strokeWidth: 2,
        });
        this.canvas.add(this.drawingObject);
      }
    },
    onMouseMove(event) {
      if (!this.isDrawing || !this.drawingObject) return;

      const pointer = this.canvas.getPointer(event.e);
      const width = pointer.x - this.drawingObject.left;
      const height = pointer.y - this.drawingObject.top;

      this.drawingObject.set({
        width: width > 0 ? width : 0,
        height: height > 0 ? height : 0,
      });

      this.canvas.renderAll();
    },
    onMouseUp() {
      if (this.isDrawing) {
        this.isDrawing = false;

        // Once drawing is done, send the object data to the socket
        this.sendDrawing({ path: this.drawingObject.toObject() });
        this.drawingObject = null; // Reset the drawing object
      }
    },
  },
};
</script>
