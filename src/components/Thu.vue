<template>
    <div class="grid-container">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
        <div class="item">5</div>
        <div class="item">6</div>
    </div>
</template>
<script setup>

</script>
<style scoped>
.grid-container{
    display: grid;
    grid-template-columns: repeat(6 ,1fr);
    gap: 10px;
}
.item {
  background-color: lightblue;
  padding: 20px;
  text-align: center;
}
</style>