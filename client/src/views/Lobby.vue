<template>
    <div class="manual-container-lobby">
        <span class="lobby-font">welcome</span>
        <div class="manual-container-sub-lobby">
          <h2 class="lobby-name"> {{player.name}} </h2>
          <h2 class="lobby-name"> {{enemy.name}} </h2>
          <div>
            
          </div>
          <div>
            <a class="btn btn-primary" style="color:white" v-if="this.$store.state.totalPlayer.length>0" @click="startGameTogether"> Battle </a>
            <a class="btn btn-primary" style="color:white" @click="setData" v-else> CheckOponent </a>
          </div>
          <div>
              <button @click="howToPlay" class="btn btn-info mt-2">How to play</button>
          </div>
        </div>
    </div>
</template>
<script>
import io from 'socket.io-client'
import axios from 'axios'

const socket = io('http://localhost:3000')

export default {
  data(){
    return {
      player:{
        name:localStorage.getItem('name')
      },
      enemy:{
        name:''
      },
      isTrigger:false
    }
  },
  methods:{
    setData(){
      // this.$store.dispatch('getDataPlayer')
      this.$store.dispatch('getDataEnemy')
      socket.emit('notify enemy', {msg:`challanger ${this.player.name} is ready` } )
      // console.log(this.$store.state.totalPlayer)
    },

    refillHealth(){
      axios({
        method:'put',
        url:`${this.$store.state.url}health`,
        headers:{token: localStorage.getItem('token')}
      })
      .then(()=>{
        this.$router.push("battle");
      })
      .catch((err)=>{
        console.log(err)
      })
    },

    startGameTogether(){
      let obj={
        trigger:true
      }
      socket.emit('startgame',obj)
    },

    howToPlay(){
        this.$router.push('/rules')
    }

  },
  created() {
    socket.on('announce', (msg)=>{
      console.log(msg.msg)
      this.enemy.name=msg.msg
    })
  },
  mounted(){
    socket.on('letsgo',(obj)=>{
      console.log(obj)
      this.isTrigger=obj.trigger

    })
  },
 watch : {
        isTrigger() {
            if(this.isTrigger==true){
              setTimeout(() => {
                console.log('oke')
                this.refillHealth()
              }, 5000);
            }
        }
    }
}
</script>