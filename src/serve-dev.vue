<script>
import Vue from 'vue';
import { PwRest } from '@/entry';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    PwRest,
  }
});
</script>

<template>
  <div id="app">
    <pw-rest
      :description="'A list of users'"
      :path="'api/users'"
      :url="'https://reqres.in/'"
    />
  </div>
</template>
