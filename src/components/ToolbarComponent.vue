<template>
  <div class="toolbar">
    <button @click="selectTool('pencil')"><i class="fas fa-pencil-alt"></i> Pencil</button>
    <button @click="selectTool('rectangle')">Rectangle</button>
    <button @click="selectTool('circle')">Circle</button>
    <button @click="selectTool('line')">Line</button>
    <button @click="selectTool('eraser')"><i class="fas fa-eraser"></i> Eraser</button>
    <button @click="clearCanvas">Clear</button>

    <div class="options">
      <label for="colorPicker">Color: </label>
      <input id="colorPicker" type="color" v-model="selectedColor" @input="updateColor" />

      <label for="brushSize">Brush Size: </label>
      <input
        id="brushSize"
        type="range"
        min="1"
        max="20"
        v-model="brushSize"
        @input="updateBrushSize"
      />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedColor: '#000000',
      brushSize: 2
    }
  },
  methods: {
    selectTool(tool) {
      this.$emit('setTool', tool)
    },
    updateColor() {
      this.$emit('setColor', this.selectedColor)
    },
    updateBrushSize() {
      this.$emit('setBrushSize', this.brushSize)
    },
    clearCanvas() {
      this.$emit('clearCanvas')
    }
  }
}
</script>

<style scoped>
.toolbar {
  display: flex;
  align-items: center;
  width: 100%;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}

button {
  margin: 5px;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
}

button i {
  margin-right: 5px;
}

button:hover {
  background-color: #0056b3;
}

.options {
  display: flex;
  gap: 5px;
  align-items: center;
}

input[type='color'],
input[type='range'] {
  margin-top: 5px;
}
</style>
