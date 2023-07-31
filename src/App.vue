<script>
import HelloWorld from './components/HelloWorld.vue'
import * as Misskey from 'misskey-js';

const client = new Misskey.api.APIClient({
  origin: 'https://misskey.systems',
  credential: import.meta.env.VITE_MISSKEY_TOKEN,
});

export default {
  components: {
    HelloWorld
  },
  data(){
    return {
      profile:{},
      timeline:[],
      meta:{},
      postContent :""
    }
  },
  methods:{
    async misskey() {
      const res = await fetch("https://misskey.systems/api/i",{
        method:"POST",
        headers: {
          'Content-Type': 'application/json'
        },
        body:JSON.stringify({
          "i":"",
          "detail":false
        })
      })
      this.profile = await res.json()
    },
    async getTimeline() {
      const res = await fetch("https://misskey.systems/api/notes/local-timeline",{
        method:"POST",
        headers: {
          'Content-Type': 'application/json'
        },
        body:JSON.stringify({
          "i":"",
        }),
        Params:JSON.stringify({"limit":"10"})
      })
      this.timeline = await res.json();
    },
    async getMisskeyAPI(){
      const profile = await client.request("i",{detail:false});
      this.profile = profile;
    },
    async postMisskey(){
      await client.request("notes/create",{text:this.postContent});
      this.postContent = ""
    }
  }
}
</script>

<template>
  <div>
    <div>{{this.profile.name}}</div>
    <img :src="profile.avatarUrl" />
    <button @click="misskey()">Misskey!</button>
    <button @click="getTimeline()">TimeLine!</button>
    <button @click="getMisskeyAPI()">MisskeyJS!</button>
    <input type="text" v-model="postContent" /><button @click="postMisskey()">
      押すよ！
    </button>
    {{postContent}}
    <v-list lines="one">
      <v-list-item v-for="(item,index) in timeline" :key="index"
        ><v-avatar :image="item.avatarUrl">
          <v-img :src="item.user.avatarUrl" /></v-avatar
        >{{item.text}}</v-list-item
      >
    </v-list>
    <HelloWorld>
      <img src="https://images.dog.ceo/breeds/weimaraner/n02092339_6869.jpg"
    /></HelloWorld>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
