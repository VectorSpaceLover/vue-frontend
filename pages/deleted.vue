<template>
  <a-row class='mh-100v'>
    <a-col>
      <p class='h1 text-center'>{{$t('usr_del')}}</p>
    </a-col>
  </a-row>
</template>

<script>
export default {
  name: 'Blocked',
  middleware: ['authenticated', 'deleted-only', 'verified'],
  head() {
    return {
      title: this.$t('acc_del'),
    }
  },
}
</script>
