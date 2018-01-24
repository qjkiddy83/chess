<template>
  <div class="bg">
    <div :class="line+'-line'" v-for="(line,lidx) in lines">
      <ul>
        <li v-for="(li,index) in list" @click="downChess" :data-id="lidx+'-'+index" :class="computedClass.call(null,lidx+'-'+index)"></li>
      </ul>
    </div>
  </div>
</template>

<script>
let arr = Array.from({length:9})
let Vue = require('vue');

export default {
  name: 'Index',
  data () {
    return {
      lines:['outer','center','inner'],
      list : arr,
      kill : 0,
      fulled : false,
      curMember : 0,//白子：0，黑子：1
      moving:0,
      chessed : function(){
        let i=0,j=0,obj = {};
        for(i;i<3;i++){
          j=0;
          for(j;j<9;j++){
            if(j!=4)
            obj[`${i}-${j}`] = 0;
          }
        }
        return obj;
      }(),
      members:[{
        sucCount : 0,
        list:[]
      },{
        sucCount : 0,
        list:[]
      }]
    }
  },
  methods:{
    checkFull:function(){
      let disabled = [];
      for(let key in this.chessed){
        if(this.chessed[key] === 0){
          return false;
        }else if(this.chessed[key]>2){
          disabled.push(key)
        }
      }
      this.fulled = true;
      alert("棋子已经下满了，开始清理废弃的棋子");
      disabled.forEach(function(item){
        this.chessed[item] = 0;
      }.bind(this))
    },
    canmove:function(id){
      let movingChess = this.moving.split('-'),target = id.split('-');
      if(movingChess[1] == target[1]){
        console.log('can')
      }else if(movingChess[0] == target[0]&&(movingChess[1]==target[1]-1||movingChess[1]==target[1]+1)){
        console.log('can')
      }else{
        console.log('canot')
      }
    },
    checkSucc:function(callback){
      let arr = Array.from(this.members[this.curMember].list),tmp = [];
      //三颗棋子组合匹配，嵌套了三层循环
      for(let i=0;i<arr.length;i++){
        let p = arr[i],s_p = p.split('-');
        for(let j=i+1;j<arr.length;j++){
          let q = arr[j],s_q = q.split('-');
          for(let x=j+1;x<arr.length;x++){
            let r = arr[x],s_r = r.split('-');
            if(s_p[1]==s_q[1]&&s_p[1]==s_r[1]&&s_q[1]==s_r[1]){//斜对角匹配
              let str = [p,q,r].sort().join(',');
              if(tmp.indexOf(str)==-1){
                tmp.push(str)
              }
            }else if(s_p[0]==s_q[0]&&s_p[0]==s_r[0]&&s_q[0]==s_r[0]){//转圈匹配
              let str = [s_p[1],s_q[1],s_r[1]].sort().join(',');
              if({
                '0,1,2':1,
                '0,3,6':1,
                '6,7,8':1,
                '2,5,8':1
              }[str]){
                let str = [p,q,r].sort().join(',');
                if(tmp.indexOf(str)==-1){
                  tmp.push(str)
                }
              }
            }
          }
        }
      }
      let newSuc = tmp.length - this.members[this.curMember].sucCount;
      if(newSuc>0){
        alert(`您可以废弃对方${newSuc}颗棋子`);
        this.kill = newSuc;
        this.members[this.curMember].sucCount += newSuc;
      }else{
        callback();
      }
    },
    downChess:function(e){
      let id = e.target.dataset.id;
      if(!this.fulled){//第一阶段
        if(this.kill > 0){//销毁对方棋子状态
          if(this.chessed[id] == this.curMember+1){//选中的是自己的棋子
            alert('要销毁对方的棋子哦');
            return;
          }else if(!this.chessed[id]){//选中的为空位置
            alert('要销毁对方的棋子哦');
            return;
          }else{
            let distoryList = this.members[(this.curMember+1)%2].list;
            this.kill--;
            this.chessed[id] = this.chessed[id]+2;//disable状态比原值+2
            distoryList.map(function(item,i){//去除列表中被销毁的棋子
              if(item == id){
                distoryList.splice(i,1)
              }
            })
            this.curMember = (this.curMember+1)%2;
            return;
          }
        }else{//落子
          if(this.kill<=0 && this.chessed[id]){//落子位置不为空
            alert('该位置已经有棋子了');
            return;
          }
          this.members[this.curMember].list.push(id);
          //设置该位置状态
          this.chessed[e.target.dataset.id] = this.curMember+1;
          this.checkSucc(function(){//判定是否连成3子
            //切换下家
            this.curMember = (this.curMember+1)%2;
          }.bind(this))
        }
        if(this.kill<=0){
          this.checkFull()
        }
      }else{
        if(this.chessed[id] == this.curMember+1){
          this.moving = id;
        }else if(this.moving){
          this.canmove(id)
        }
      }
    },
    computedClass:function(e){
      let cls = "init";
      switch(this.chessed[e]){
        case 1:
          cls = "black";
        break;
        case 2:
          cls = "white";
        break;
        case 3:
          cls = "black disable";
        break;
        case 4:
          cls = "white disable";
        break;
        default:
          cls = "init"
        break;
      }
      return cls;
    }
  }
}
</script>

<style lang="scss">
@import '../assets/index.scss'
</style>
