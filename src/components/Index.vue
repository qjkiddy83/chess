<template>
  <div class="bg">
    <h1>我是{{name}}（{{memberRole}}）</h1>
    <h2 v-if="started">{{curMember==1?'白方':'黑方'}}落子中。。。</h2>
    <h3>{{result}}</h3>
    <p>{{msg}}</p>
    <div :class="line+'-line'" v-for="(line,lidx) in lines">
      <ul>
        <li v-for="(li,index) in list" @click="downChess" :data-id="lidx+'-'+index" :class="computedClass.call(null,lidx+'-'+index)"></li>
      </ul>
    </div>
  </div>
</template>

<script>
let arr = Array.from({length:8})
let Vue = require('vue');
let userId = null;
// let socket = require('socket.io-client')('http://192.168.105.101:3000');
let socket;

export default {
  name: 'Index',
  props:["name"],
  data () {
    return {
      memberRole : '匹配中',
      started : false,
      lines:['outer','center','inner'],
      list : arr,
      kill : 0,
      fulled : false,
      curMember : 0,//白子：1，黑子：0
      moving:0,
      result : '',
      control: false,
      msg:'',
      chessed : function(){
        let i=0,j=0,obj = {};
        for(i;i<3;i++){
          j=0;
          for(j;j<8;j++){
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
      this.msg = "棋子已经下满了，开始清理杀死的棋子";
      setTimeout(function(){
        disabled.forEach(function(item){
          this.chessed[item] = 0;
          this.msg = '';
          this.changeControl();
        }.bind(this))
      }.bind(this),500)
    },
    move:function(org,target){
      this.members[this.curMember].list.map((item,i)=>{
        if(item == org){
          this.members[this.curMember].list.splice(i,1,target);
        }
      })
      this.chessed[org] = 0;
      this.chessed[target] = this.curMember+1;
      this.moving = 0;
      this.checkMoveSucc(target)
    },
    canmove:function(id){
      let movingChess = this.moving.split('-'),target = id.split('-');
      if(this.chessed[id]){
        this.msg = '该位置已经有棋子了'
      }else if(movingChess[1] == target[1]){
        // console.log('can')
        this.move(this.moving,id)
      }else if(movingChess[0] == target[0]){
        if(movingChess[1]==parseInt(target[1])-1||movingChess[1]==parseInt(target[1])+1){
          // console.log('can')
          this.move(this.moving,id)
        }else if([movingChess[1],target[1]].sort().join('') == '07'){
          // console.log('can')
          this.move(this.moving,id)
        }else{
          console.log('cannot')
        }
      }else{
        console.log('cannot')
      }
    },
    checkMoveSucc:function(id){
      let arr = Array.from(this.members[this.curMember].list),tmp = [];
      //三颗棋子组合匹配，嵌套了三层循环
      let p = id,s_p = p.split('-');
      for(let j=0;j<arr.length;j++){
        let q = arr[j],s_q = q.split('-');
        if(q == p) continue;
        for(let x=j+1;x<arr.length;x++){
          let r = arr[x],s_r = r.split('-');
          if(r == p) continue;
          if(s_p[1]==s_q[1]&&s_p[1]==s_r[1]&&s_q[1]==s_r[1]){//斜对角匹配
            let str = [p,q,r].sort().join(',');
            if(tmp.indexOf(str)==-1){
              tmp.push(str)
            }
          }else if(s_p[0]==s_q[0]&&s_p[0]==s_r[0]&&s_q[0]==s_r[0]){//转圈匹配
            let str = [s_p[1],s_q[1],s_r[1]].sort().join(',');
            if({
              '0,1,2':1,
              '2,3,4':1,
              '4,5,6':1,
              '0,6,7':1
            }[str]){
              let str = [p,q,r].sort().join(',');
              if(tmp.indexOf(str)==-1){
                tmp.push(str)
              }
            }
          }
        }
      }
      if(tmp.length > 0){
        this.msg = `可以杀死${this.curMember==0?'白':'黑'}方${tmp.length}颗棋子`;
        this.kill = tmp.length;
      }else{
        this.changeControl();
      }
    },
    checkSucc:function(id,callback){
      let arr = Array.from(this.members[this.curMember].list),tmp = [];
      // console.log(id,arr);
      //三颗棋子组合匹配，嵌套了三层循环
      let p = id,s_p = p.split('-');
      for(let j=0;j<arr.length;j++){
        let q = arr[j],s_q = q.split('-');
        if(q == p) continue;
        for(let x=j+1;x<arr.length;x++){
          let r = arr[x],s_r = r.split('-');
          if(r == p) continue;
          if(s_p[1]==s_q[1]&&s_p[1]==s_r[1]&&s_q[1]==s_r[1]){//斜对角匹配
            let str = [p,q,r].sort().join(',');
            if(tmp.indexOf(str)==-1){
              tmp.push(str)
            }
          }else if(s_p[0]==s_q[0]&&s_p[0]==s_r[0]&&s_q[0]==s_r[0]){//转圈匹配
            let str = [s_p[1],s_q[1],s_r[1]].sort().join(',');
            if({
              '0,1,2':1,
              '2,3,4':1,
              '4,5,6':1,
              '0,6,7':1
            }[str]){
              let str = [p,q,r].sort().join(',');
              if(tmp.indexOf(str)==-1){
                tmp.push(str)
              }
            }
          }
        }
      }
      
      if(tmp.length>0){
        this.msg = `可以杀死${this.curMember==0?'白':'黑'}方${tmp.length}颗棋子`;
        this.kill = tmp.length;
      }else{
        callback();
      }
    },
    changeControl:function(isSocket){
      // this.control = true;
      if(!isSocket){
        socket.emit('change', localStorage.userId);
      }
      this.curMember = (this.curMember+1)%2;
    },
    downChess:function(e,isSocket){
      let id = isSocket?e:e.target.dataset.id;
      console.log('(!this.control && !isSocket)',this.control,isSocket)
      if(!this.control && !isSocket){
        return;
      }
      this.msg = '';
      if(this.result){
        return;
      }
      if(!this.fulled){//第一阶段
        if(this.kill > 0){//杀掉对方棋子状态
          if(this.chessed[id] == this.curMember+1){//选中的是自己的棋子
            this.msg = `要杀掉${this.curMember==0?'白':'黑'}方的棋子哦`;
          }else if(!this.chessed[id]){//选中的为空位置
            this.msg = `要杀掉${this.curMember==0?'白':'黑'}方的棋子哦`;
          }else{
            let distoryList = this.members[(this.curMember+1)%2].list;
            this.kill = this.kill-1<=0?0:this.kill-1;
            this.chessed[id] = this.chessed[id]+2;//disable状态比原值+2
            distoryList.map(function(item,i){//去除列表中被杀掉的棋子
              if(item == id){
                distoryList.splice(i,1)
              }
            })
            this.changeControl(isSocket)
          }
        }else{//落子
          if(this.kill==0 && this.chessed[id]){//落子位置不为空
            this.msg = '该位置已经有棋子了';
            return;
          }
          this.members[this.curMember].list.push(id);
          //设置该位置状态
          this.chessed[id] = this.curMember+1;
          this.checkSucc(id,function(){//判定是否连成3子
            //切换下家
            this.changeControl(isSocket)
          }.bind(this))
        }
        if(this.kill<=0){
          this.checkFull()
        }
      }else{
        if(this.kill > 0){//杀掉对方棋子状态
          if(this.chessed[id] == this.curMember+1){//选中的是自己的棋子
            this.msg = `要杀掉${this.curMember==0?'白':'黑'}方的棋子哦`;
          }else if(!this.chessed[id]){//选中的为空位置
            this.msg = `要杀掉${this.curMember==0?'白':'黑'}方的棋子哦`;
          }else{
            let distoryList = this.members[(this.curMember+1)%2].list;
            this.kill = this.kill-1<=0?0:this.kill-1;
            this.chessed[id] = 0;//disable状态比原值+2
            distoryList.map(function(item,i){//去除列表中被杀掉的棋子
              if(item == id){
                distoryList.splice(i,1)
              }
            }.bind(this))
            if(this.kill == 0 ){//切换下家
              this.changeControl(isSocket)
            }
          }
        }else{
          if(this.chessed[id] == this.curMember+1){
            this.moving = id;
          }else if(this.moving){
            this.canmove(id)
          }
        }
        this.checkFail();
      }
      if(!isSocket){
        socket.emit('chess', {
          userId:userId,
          chessId:id
        });
      }
    },
    checkMoveAble:function(list){//判断是否还有可以走的棋子，否则直接判输
      let res = false;
      list.forEach(id =>{
        let s_id = id.split('-'),_prev = parseInt(s_id[1])-1,prev = _prev<0?(_prev+8):_prev,_next = parseInt(s_id[1])+1,next = _next>7?_next-8:_next;
        if(!this.chessed[`${s_id[0]}-${prev}`] || !this.chessed[`${s_id[0]}-${next}`]){//转圈有空位置
          res = true;
        }else{
          if(parseInt(s_id[0])>0 && !this.chessed[`${parseInt(s_id[0])-1}-${s_id[1]}`]){//内圈有空位
            res = true;
          }
          if(parseInt(s_id[0])<2 && !this.chessed[`${parseInt(s_id[0])+1}-${s_id[1]}`]){//外圈有空位
            res = true;
          }
        }
      })
      return res;
    },
    checkFail:function(){
      //黑方
      if(this.members[0].list.length<3){
        this.result = '白方胜';
        return 'white';
      }
      if(!this.checkMoveAble(this.members[0].list)){
        this.result = '白方胜';
        return 'white';
      }
      //白方
      if(this.members[1].list.length<3){
        this.result = '黑方胜'
        return 'black';
      }
      if(!this.checkMoveAble(this.members[1].list)){
        this.result = '黑方胜';
        return 'black';
      }
      return 0;
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
      if(e == this.moving){
        cls += ' moving'
      }
      return cls;
    }
  },
  mounted:function(){
    let that = this;
    socket = require('socket.io-client')('http://chess.jsers.cn');
    socket.on('connect', function(){
      socket.emit('connect_user', localStorage.userId);
    });
    socket.on('connected', function(data){
      userId = data.userid;
      localStorage.userId = userId;
    });
    socket.on('disconnect', function(){

    });
    socket.on('start', function(data){
      console.log('data.control',data.control)
      that.control = data.control;
      that.started = true;
      that.memberRole = data.control ?"黑方":"白方";
    });
    socket.on('changeControl', function(data){
      console.log('data.control',data.control)
      that.control = data.control;
    });
    socket.on('chessed', function(data){
      // console.log('chessed')
      if(data.userId != userId){
        that.downChess(data.chessId,true);
      }
    });
  }
}
</script>

<style lang="scss">
@import '../assets/index.scss'
</style>
