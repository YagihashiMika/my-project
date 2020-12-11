<template>
  <div id="app">
  <myheader></myheader>
  <mynav></mynav>
  <div class="check-form">
    <input id="inputID" type="text" value="User ID"/>
    <input id="checkButton" type="button" value="CHECK"/>
  </div>
  <div v-html="titleList"></div>
  </div>
</template>

<script>
import myheader from './components/myheader'
import mynav from './components/mynav'
import { Problems } from './problem.js'

export default {
  components: {
    myheader,
    mynav
  },

  data: function() {
    return {
      titleList: '',
      item: {},
    }
  },

  created: function (){
    var count = 1;
    for (const p of Problems){
      if (!p.id){
        this.titleList += `<h2 id="${p.title}">${p.title}</h2>`
      }else{
        const url = `http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=${p.id}&lang=jp`;
        const item = {
          key: p.id,
          title: `(${count}) ${p.title}`,
          color: 'white',
          record: p.hint || '',
        }
        this.titleList += `
          <div style="background: ${item.color}">
            <p><a href=${url}>(${count}) ${p.title}</a><span>${item.record}</span></p>
          </div>
        `
        count += 1;
      }
    }
  },

  methods: {
    
  }
}
</script>

<style>
#app {
  
}
</style>
