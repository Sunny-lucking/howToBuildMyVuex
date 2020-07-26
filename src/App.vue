<template>
  <div id="app">
    123
    <p>state:{{this.$store.state.num}}</p>
    <p>getter:{{this.$store.getters.getNum}}</p>
    <button @click="add">+1</button>
    <button @click="asyncAdd">异步+2</button>
  </div>
</template>

<script>
  export default {
      methods:{
          add(){
              this.$store.commit('incre',1)
          },
          asyncAdd(){
              this.$store.dispatch('asyncIncre',2)
          }
      }
  }
</script>


