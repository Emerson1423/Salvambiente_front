<template>
    <NavComponente></NavComponente>
    <br>
    <br>
    <perfilComponente></perfilComponente>
   
</template>

<script>
import perfilComponente from '@/components/perfilComponente.vue'
import NavComponente from '@/components/NavComponente.vue';


export default {
    components: {
        perfilComponente,
        NavComponente
       
    }
}
</script>
