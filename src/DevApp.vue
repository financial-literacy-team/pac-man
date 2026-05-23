<template>
  <div style="display: flex; justify-content: center; padding: 20px;">
    <PacmanGame @gameOver="(res) => console.log(res)" />
  </div>
</template>

<script setup lang="ts">
import { PacmanGame } from './index'
</script>
