<template>
     <v-card tile height="50px" flat >
    
    <v-bottom-nav 
      active.sync="recent"
      :value="true"
      fixed
      color="blue-grey darken-4"
    >
  
     <v-btn 
        
        color="white"
        flat
        value="home"
         @click="$vuetify.goTo(target, options)"
        
      >
    <v-icon>home</v-icon>
      </v-btn>
 
    <v-bottom-sheet v-model="sheet"  lazy >
       
<v-btn
        slot="activator"
        dark
        flat
        value="info"
      >  

        <v-icon>info</v-icon>
      </v-btn>
        
     <v-card raised ripple class="text-xs-center">
    
        <v-card-title primary-title>
        <footer-info>  </footer-Info>
        </v-card-title>
      </v-card>

      
    </v-bottom-sheet>
   
  </v-bottom-nav>
    </v-card>
</template>
<script>
import footerInfo from "./FooterInfo.vue"
export default {
    components:{
      footerInfo
    },
    name:"bottomNav",

    data: function() {
        return {
      selector:"#content",
      duration:300,
      offset:15,
      easing: 'easeInOutCubic',
      sheet: false,
       tiles: [
        { icon: 'search', title: 'Search' },
        { icon: 'share', title: 'Share' }
       
      ]
      
      
        }
    },
    computed:{
     target(){
      let vm = this
      let value = vm.selector
      if(!isNaN(value)) return Number(value)
      else return value
    },
    options () {
        return {
          duration: this.duration,
          offset: this.offset,
          easing: this.easing
        }
    }
    }
}
</script>
<style scoped>

</style>


