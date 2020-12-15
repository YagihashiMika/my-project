<template>
  <div id="app">
  <myheader></myheader>
  <mynav></mynav>
  <div class="check-form">
    <input id="inputID" type="text" v-model="userID" placeholder="User ID"/>
    <input id="checkButton" type="button" value="CHECK" @click="handleClick()"/>
  </div>
  <div v-html="titleList"></div>
  </div>
</template>

<script>
import myheader from './components/myheader'
import mynav from './components/mynav'
import { Problems } from './problem.js'

const ACCEPTED = 4;
const PRESENTATIONERROR = 8;

const GREEN = '#72af68';
const YELLOW = '#f9e169';
const RED = '#f48876';

const StatusIcons = [
  '😭', '😡', '😱', '😱', '🍺',
  '⚡️', '⚡️', '😨', '😓', '⚡️',
]

export default {
  components: {
    myheader,
    mynav
  },

  data: function() {
    return {
      titleList: '',
      item: {},
      userID: '',
      pdb: {}
    }
  },

  created: function (){
    var count = 1;
    for (const p of Problems){
      if (!p.id){
        this.titleList += `<h2 id="${p.title}">${p.title}</h2>`
      }else{
        const url = `http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=${p.id}&lang=jp`;
        this.item = {
          key: p.id,
          title: `(${count}) ${p.title}`,
          color: 'white',
          record: p.hint || '',
        }
        this.titleList += `
          <div style="background: ${this.item.color}">
            <p><a href=${url}>(${count}) ${p.title}</a><span>${this.item.record}</span></p>
          </div>
        `
        count += 1;
      }
    }
  },

  methods: {
    colors(status, prev){
      if (status === ACCEPTED || prev === GREEN) {
        return GREEN
      }
      if (status === PRESENTATIONERROR || prev === YELLOW) {
        return YELLOW
      }
      return RED
    },

    records(status, prev= ''){
      return StatusIcons[status] + prev
    },

    uname(){
      return sessionStorage.getItem('aoj_uname') || ''
    },

    check(data){
      this.pdb = {}
      for (const d of data){
        if (d.language !== 'Python3') {
          continue
        }
        if (d.submissionDate < 1587915078539) {
          continue
        }
        if (d.status !== 4 && !d.accuracy.startsWith('0')) {
          d.status = PRESENTATIONERROR
        }
        if (d.problemId in this.pdb){
          const entry = this.pdb[d.problemId];
          entry.color = this.colors(d.status, entry.color);
          entry.record = this.records(d.status, entry.record);
        } else {
          const entry = { 
            problemId: d.problemId,
            color: this.colors(d.status),
            record: this.records(d.status),
            date: new Date(d.submissionDate / 1000),
          }
          this.pdb[d.problemId] = entry;
        }
      } 
    },

    solvedList(){
      this.titleList = ''
      var count = 1;
      for (const p of Problems){
        if(!p.id){
          this.titleList += `<h2 id="${p.title}">${p.title}</h2>`;
        }else {
          const url = `http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=${p.id}&lang=jp`;
          this.item.key = p.id;
          this.item.color = 'white';
          this.item.record = p.hint || '';
          if (p.id in this.pdb) {
            const entry = this.pdb[p.id];
            this.item.color = entry.color;
            this.item.record = entry.record;
          }
          this.titleList += `
            <div style="background: ${this.item.color}">
              <p><a href=${url}>(${count}) ${p.title}</a><span>${this.item.record}</span></p>
            </div>
            `;
          count += 1;
        }
      }
    },

    handleClick(){
      let vc = this;
      fetch(`https://judgeapi.u-aizu.ac.jp/submission_records/users/${this.userID}?size=2000`)
      .then(function(response) {
        return response.json()
      })
      .then(function(json){
        vc.check(json)
        vc.solvedList()
      })
      .catch(function(error) {
        alert(error)
      });
    } 
  }
}
</script>

<style>
#app {
  
}
</style>
