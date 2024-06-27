<script setup>
import HelloWorld from './components/HelloWorld.vue';
</script>

<template>
  <div class="body">
    <h1>TEST</h1>
    <div>
      <label>名前</label>
      <input />
    </div>
    <div>
      <label>開始時期</label>
      <select>
        <option disabled value="">Please select one</option>
        <option>6</option>
        <option>7</option>
        <option>8</option>
        <option>9</option>
        <option>10</option></select
      ><span>月</span>
      <select>
        <option disabled value="">Please select one</option>
        <option>上</option>
        <option>中</option>
        <option>下</option></select
      ><span>旬</span>
    </div>
    <div>
      <label>終了時期</label>
      <select>
        <option disabled value="">Please select one</option>
        <option>6</option>
        <option>7</option>
        <option>8</option>
        <option>9</option>
        <option>10</option></select
      ><span>月</span>
      <select>
        <option disabled value="">Please select one</option>
        <option>上</option>
        <option>中</option>
        <option>下</option></select
      ><span>旬</span>
    </div>
    <div>
      <input type="radio" name="texure" />
      <label>やわらかい</label>
      <input type="radio" name="texure" />
      <label>かたい</label>
    </div>
    <button @click="addItem">追加</button>
  </div>
</template>

<style scoped>
.body {
  background-color: #fcffe0;
  height: 100vh;
}
</style>
