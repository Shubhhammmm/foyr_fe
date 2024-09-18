<template>
  <div class="toolbar">
    <button @click="selectTool('pencil')">Pencil</button>
    <button @click="selectTool('rectangle')">Rectangle</button>
  </div>
</template>

<script>
export default {
  methods: {
    selectTool(tool) {
      this.$emit('setTool', tool);
    }
  }
};
</script>


