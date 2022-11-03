<script setup>
import { RouterView } from 'vue-router'
import { NSpace, NButton, NLayout, NLayoutContent, NMessageProvider, NDialogProvider } from 'naive-ui'
import Header from './layouts/Header.vue'
import Sidebar from './layouts/Sidebar.vue'
import Footer from './layouts/Footer.vue'
</script>

<template>
<NDialogProvider>
  <NMessageProvider> 
    <NLayout has-sider>
      <Sidebar/>
      <NLayout style="min-height: 100vh;">
        <Header/>
        <NLayoutContent content-style="padding: 24px;">
          <RouterView />
        </NLayoutContent>
        <Footer/>
      </NLayout>
    </NLayout>
  </NMessageProvider>
</NDialogProvider>

  

  <!-- <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
      </nav>
    </div>
  </header>

  <NSpace>
    <NButton type="info">Ini sebuah Button</NButton>
  </NSpace> -->

  
</template>




