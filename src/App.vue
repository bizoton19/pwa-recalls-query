<template>
  <v-app>

    <v-toolbar
      color="indigo darken-1"
      dark
      dense
      app
      fixed
      id ="toolbar"
      
    >

      <v-toolbar-title v-text="title" color="white" ></v-toolbar-title>
       <div class="d-flex align-center " style="margin-left: auto">
        <v-btn small 
         round
         color="orange darken-4"
         v-if="beforeinstallpromptfired"
         @click="showInstallPrompt($event)"
        >
          <v-icon >add_to_home_screen</v-icon>
          install
        </v-btn>
        <v-btn icon
         round
         color="white"
        
        >
          <v-avatar size="32px" tile>
            <img
              src="https://www.cpsc.gov/sites/all/themes/cpsc/images/logo.png"
              alt="cpsc"
            >
          </v-avatar>
        </v-btn>
      </div>
    </v-toolbar>
    <v-content id="content">
      <router-view/>
    </v-content>
  
<v-fab-transition v-if="showOnMdAndUp" >
      <v-btn
        color="orange darken-4"
        dark
        fab
        fixed
        bottom
        right
        v-show="action"
        id="btnActionScroll"
        @click="$vuetify.goTo(target, options)"
      >
        <v-icon>keyboard_arrow_up</v-icon>
        <v-icon>close</v-icon>
      </v-btn>
    </v-fab-transition>
    
  <bottom-nav v-if="showOnSmAndDown" ></bottom-nav>
  <mfooter v-if="showOnMdAndUp"></mfooter>
    
  </v-app>
</template>

<script>

import  bottomNav  from "./components/BottomNav.vue";
import  mfooter from "./components/mFooter.vue"

export default {
  components:{
    mfooter,
    bottomNav
  },
   name: 'App',
  data () {
    return {
      selector:"#content",
      duration:300,
      offset:15,
      easing: 'easeInOutCubic',
      title: 'CPSC Recalls Query',
      action: true,
      beforeinstallpromptfired:false,
      deferredPrompt:null
    
    }
  },
  computed: {
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
    },
    showOnMdAndUp(){
      return this.$vuetify.breakpoint.mdAndUp
    },
    showOnSmAndDown(){
     return  this.$vuetify.breakpoint.smAndDown
    }
  },
 
mounted(){
    const vm = this
    window.addEventListener('beforeinstallprompt', (e) => {
  // Prevent Chrome 67 and earlier from automatically showing the prompt
     e.preventDefault();
    
  // Stash the event so it can be triggered later.
    vm.deferredPrompt = e;
  //show install button
  vm.beforeinstallpromptfired = true

  window.addEventListener('appinstalled', (event) => {
     alert('app successfuly installed');
    });
});
  },
  methods:{
  showInstallPrompt(e){
   const vm = this
   
   vm.beforeinstallpromptfired = false 
   vm.deferredPrompt.prompt();
   vm.deferredPrompt.userChoice
    .then((choiceResult)=>{
      if(choiceResult.outcome === 'accepted'){
        
      }else{
        alert('user dimissed prompt, will have to wait until the beforepromptinstall event fires')
      }
      vm.deferredPrompt = null
      })
    
  }
  }
}
</script>
