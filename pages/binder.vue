<template>
  <div id="binder" class="binder-container w"></div>
</template>

<script>
  import Storage from '@/utils/storage'
  import * as API_Connect from '@/api/connect'
  import { domain } from '@/ui-domain'
  export default {
    name: 'binder',
    layout: 'full',
    mounted() {
      const { uuid } = this.$route.query
      if (uuid) Storage.setItem('uuid', uuid, { expires: 30 })
      // If there is a refresh token, binding or binding is performed after login
      if (Storage.getItem('refresh_token')) {
        const uuid_connect = Storage.getItem('uuid_connect')
        API_Connect.loginBindConnect(uuid_connect).then(response => {
          const { access_token, refresh_token } = response
          Storage.setItem('access_token', access_token)
          Storage.setItem('refresh_token', refresh_token)
          Storage.setItem('uuid', uuid_connect)
          Storage.removeItem('uuid_connect')
          this.$router.replace({ name: 'member-account-binding' })
        }).catch(() => {
          this.$router.replace({ name: 'member-account-binding' })
        })
        return false
      }
      this.$layer.open({
        title: 'Authorization success',
        btn: ['Sign in', 'Register'],
        closeBtn: 0,
        content: 'The current authorization is successful. Please：',
        move: false,
        yes: (index) => {
          this.$layer.close(index)
          this.$router.push({ name: 'login', query: { form: 'connect' } })
        },
        btn2: (index) => {
          this.$layer.close(index)
          this.$router.push({ name: 'register', query: { form: 'connect' } })
        }
      })
    },
    methods: {
    }
  }
</script>

<style type="text/scss" lang="scss" scoped>
  @import "../assets/styles/color";
  .binder-container {
    min-height: 200px;
  }
</style>
