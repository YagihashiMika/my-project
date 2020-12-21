<template>
  <div id="app">
  <myheader></myheader>
  <mynav></mynav>
  <div id="AOJ">
    <div class="check-form">
      <input id="inputID" type="text" v-model="userID" placeholder="User ID"/>
      <input id="checkButton" class="btn btn-warning" type="button" value="CHECK" @click="handleClick()"/>
    </div>
    <div id="List" v-html="titleList"></div>
  </div>
  </div>
</template>

<script>
import myheader from './components/myheader'
import mynav from './components/mynav'
import { Problems } from './problem.js'

const ACCEPTED = 4;
const PRESENTATIONERROR = 8;

const GREY = '#cccccc';
const WHITE = '#ffffff';

const StatusIcons = [
  '<img src="images/ce@3x.png">',
  '<img src="images/wa@3x.png">', 
  '<img src="images/tl@3x.png">', 
  '<img src="images/ml@3x.png">', 
  '<img src="images/ac@3x.png">', 
  '<img src="images/w@3x.png">', 
  '<img src="images/ol@3x.png">', 
  '<img src="images/re@3x.png">', 
  '<img src="images/pe@3x.png">', 
  '<img src="images/sr@3x.png">',
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
      if (status === ACCEPTED || prev === GREY) {
        return GREY
      }
      return WHITE
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
#AOJ {
  width: 75%;
  float: left;
  margin: 120px 5% 0 20%;
}

#AOJ #List{
  margin-top: 20px;
}

#AOJ p{
    width: 100%;
    font-size: 15px;
    line-height: 2.6;
    border-bottom: 1px dotted #000;
}

#AOJ span{
  float: right;
}

#AOJ h2{
  margin-top: 50px;
}

#AOJ #inputID{
  font-size: 17px;
  font-family: 'Noto Sans JP', sans-serif;
  margin: 30px 0 0px 0;
  height: 40px;
  width: 13em;
  transition: all .6s ease;
  border-bottom:0.5px solid #000;
  border-right:none;
  border-left:none;
  border-top:none;
  line-height: 40px;
}

#AOJ #inputID:focus{
    width:16em;
    outline: none;
}

#AOJ #checkButton{
  margin: 0 20px;
  font-size: 14px;
  color: #fff;
}

#AOJ img{
    width: 20px;
    height: 20px;
    margin-right: 10px;
}
</style>
