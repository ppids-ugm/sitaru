<template>
  <div class="about">
    <h1>This is an about page</h1>
    <account-info username='matt' />
  </div>
</template>

<script>
import AccountInfo from '@/components/DataBasemap.vue'
export default {
  components: {
    AccountInfo
  }
}
</script>
