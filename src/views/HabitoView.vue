<template>
  <NavComponente></NavComponente>
  <br>
  <div>
    <HabitosComponente />
  </div>
  <footerComponente/>
</template>

<script>
import HabitosComponente from '../components/HabitosComponente.vue';
import NavComponente from '@/components/NavComponente.vue';
import footerComponente from '@/components/footerComponente.vue';
export default {
  name: 'HabitosView',
  components: {
    HabitosComponente,
    NavComponente,
    footerComponente
  }
}
</script>