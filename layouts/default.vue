<template>
  <v-app>
    <navbar></navbar>
    <div id="app-content">
      <Nuxt></Nuxt>
    </div>
    <MFooter></MFooter>
  </v-app>
</template>

<script>
import Navbar from '~/components/Navbar'
import MFooter from '~/components/MFooter'
import authMixin from '~/mixins/authMixin'
import listenMixin from '~/mixins/listenMixin'
import userUpdatedMixin from '~/mixins/userUpdatedMixin'
import chatMixin from '~/mixins/chatMixin'

export default {
  name: 'Default',
  components: {
    Navbar,
    MFooter
  },
  mixins: [authMixin, listenMixin, userUpdatedMixin, chatMixin],
  data: ()=>({
    socket: null,
  }),
  watch:{
    isLoggedIn(loggedIn){
      if (loggedIn){
        this.run_once(this.listen)
      }else{
        this.socket = null
      }
    }
  },
  methods: {
    listen() {
      this.socket = this.$nuxtSocket({ persist: 'defaultSocket' })
      this.socket.emit('join-room', this.$auth.user.uuid)
      this.socket.on('fetch-user', async ()=>{
        await this.$auth.fetchUser()
        this.user_was_updated()
      })
      this.socket.on('user-reload', ()=>{
        this.getChats(true)
      })
    }
  }
}
</script>

<style lang='sass'>
body
  overflow-y: auto
#app-content
  margin-top: 50px !important
</style>
