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
      postContent :"",
      drives:[],
      files:[],
      uploadImage:{},
      credential:import.meta.env.VITE_MISSKEY_TOKEN,
      origin:'https://misskey.systems',
    }
  },
  async mounted(){
    const profile = await client.request("i",{detail:false});
    const drives = await client.request("drive/files",{limit:99})
    this.profile = profile;
    this.drives = drives
  },
  methods:{
    async postMisskey(){
      await client.request("notes/create",{text:this.postContent,fileIds:this.files});
      this.postContent = ""
      this.files = []
    },
    async getMisskeyTimelien(){
      const timeline = await client.request("notes/local-timeline",{limit:10});
      this.timeline = timeline;
    },
    addFilesIds(fileId){
      this.files.push(fileId)
    },
    attachFile(){
      this.uploadImage = this.$refs.image.files[0];
      console.log(this.uploadImage)
    },
    async uploadFile(){
      const params = new FormData();
      //画像ファイルをアップロードする時、file/i(TOKEN)/nameは必須
      params.append("file",this.uploadImage)
      params.append("i",this.credential)
      params.append("name",this.uploadImage.name)

      const response = await fetch(`${this.origin}/api/drive/files/create`,{
        method:"POST",
        body:params,
        credentials: 'omit',
        cache: 'no-cache',
      })
      const data = await response.json();
      //dataの中にはアップロードした戻り値のファイル情報が格納されている。
      //data.idで、対象ファイルのIDを取得できる
      this.addFilesIds(data.id)
    }
  }
}
</script>

<template>
  <div class="d-flex flex-column">
    <div class="d-flex flex-column align-center">
      <img style="width:100px; height:100px;" :src="profile.avatarUrl" />
      <div style="width:300px; height:30px;">{{this.profile.name}}</div>
    </div>
    <button @click="getMisskeyTimelien()">TimeLineを取得</button>
    <input type="text" v-model="postContent" /><button @click="postMisskey()">
      とうこう！
    </button>
    <input type="file" @change="attachFile" ref="image" />
    <v-btn @click="uploadFile()">画像をアップロード！</v-btn>
    {{postContent}}
    {{files}}
    <v-list lines="one">
      <v-list-item v-for="(item,index) in timeline" :key="index"
        ><v-avatar :image="item.avatarUrl">
          <v-img :src="item.user.avatarUrl" /></v-avatar
        >{{item.text}}</v-list-item
      >
    </v-list>
    <v-list lines="two">
      <v-list-item v-for="(file,index) in drives" :key="index">
        <v-btn @click="addFilesIds(file.id)" variant="outlined"> 加える</v-btn
        ><v-img :src="file.thumbnailUrl"
      /></v-list-item>
    </v-list>
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
