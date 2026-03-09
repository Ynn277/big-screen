<template>
  <!-- <dv-full-screen-container> 
    <dv-loading v-if="loading">正在加载...</dv-loading>
    <div v-else style="display: flex; flex-direction: column; height: 100%">
      <TitleView style="flex: 0 0 80px;"/>
      <MainView style="flex: 1; height: 100%;"/>
      <FooterView style="flex: 0 0 200px;"/> -->
    <!-- </div>
  </dv-full-screen-container> -->

<v-scale-screen width="1920" height="1080" :fullScreen="false" :wrapperStyle="{ backgroundColor: '#000000' }" :boxStyle="{ backgroundColor: '#000000' }">
    
    <div class="content bg"> <dv-loading v-if="loading">正在加载...</dv-loading>
      
      <div v-else class="app-layout">
        <TitleView class="header-section"/>
        <MainView class="main-section"/>
        </div>

    </div>

  </v-scale-screen>
</template>


<script setup>
import {ref,onMounted} from 'vue';
import VScaleScreen from 'v-scale-screen'
import TitleView from './TitleView.vue'
import MainView from './MainView.vue'

let loading = ref(true);

onMounted(()=>{
  setTimeout(()=>{
    loading.value = false;
  },1000)
})


</script>


