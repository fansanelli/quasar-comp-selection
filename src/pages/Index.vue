<template>
  <q-page padding>
    <q-selection-area :itemsList="this.myList" :beginSelected="this.selected" @update:selected="selected = $event" />
  </q-page>
</template>

<style>
</style>

<script>
import QSelectionArea from '../components/QSelectionArea'
import countries from '../statics/countries.json'

export default {
  name: 'PageIndex',
  data: function () {
    return {
      myList: countries.list,
      selected: ['IT', 'GB', 'DE']
    }
  },
  watch: {
    selected () {
      // console.log(this.selected)
    }
  },
  components: { QSelectionArea }
}
</script>
