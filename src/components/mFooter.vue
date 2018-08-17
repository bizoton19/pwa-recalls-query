<template>
    <v-footer
    class="blue-grey darken-4"
    height="auto"
  >
    <v-card
      class="flex text-lg-center"
      flat
      tile

    >
      <v-card-title class="justify-center blue-grey darken-4"  >

        <footer-info></footer-info>
      </v-card-title>
      
    
    </v-card>
  </v-footer>
</template>
<script>
import footerInfo from "./FooterInfo.vue"
export default {
  name: "mfooter",
  components:{
    footerInfo
  }
};
</script>

