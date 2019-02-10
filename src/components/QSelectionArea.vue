<template>
  <div>
    <div class="q-mb-lg">
      <div class="text-right text-primary">
        <q-btn label="Select all" @click="selectAll" />
        <q-btn label="Reset" @click="reset" />
      </div>
    </div>
    <div class="row" style="height: 70vh !important;">
      <q-list bordered separator class="col-12 col-md-6">
        <q-item-section>
          <q-item-label overline class="row justify-between">
            Selected
            <div>
              <q-btn size="sm" text-color="primary" label="Toggle selection" @click="toggle" />
            </div>
          </q-item-label>
        </q-item-section>
        <q-item clickable v-ripple :key="item.value" v-for="item in selectedItemsList">
          <q-item-section @click.native="remove(item.value)">
            {{item.label}}
          </q-item-section>
        </q-item>
      </q-list>
      <q-list bordered separator class="col-12 col-md-6 border-top-color-sm">
        <q-item-section>
          <q-item-label overline>
            <q-input
              v-model="search"
              debounce="500"
              filled
              placeholder="Search"
            >
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>
            Available
          </q-item-label>
        </q-item-section>
        <q-item clickable v-ripple :key="item.value" v-for="item in unselectedItemsList">
          <q-item-section @click.native="add(item.value)">
            {{item.label}}
          </q-item-section>
        </q-item>
      </q-list>
    </div>
  </div>
</template>

<script>
export default {
  name: 'q-selection-area',
  data: function () {
    return {
      search: ''
    }
  },
  methods: {
    add (value) {
      this.$emit('update:selected', this.selected.concat(value))
    },
    remove (value) {
      this.$emit('update:selected', this.selected.filter((f) => { return f !== value }))
    },
    selectAll () {
      this.$emit('update:selected', this.itemsList.map((item) => { return item.value }))
    },
    reset () {
      this.$emit('update:selected', [])
    },
    toggle () {
      const selected = this.itemsList.map((item) => {
        return item.value
      }).filter((f) => {
        return !this.selected.includes(f)
      })
      this.$emit('update:selected', selected)
    }
  },
  computed: {
    selectedItemsList () {
      return this.itemsList.filter((item) => {
        return this.selected.includes(item.value)
      })
    },
    unselectedItemsList () {
      const searchUpperCase = this.search.toUpperCase()
      return this.itemsList.filter((item) => {
        return !this.selected.includes(item.value) &&
          (searchUpperCase.length === 0 || item.label.toUpperCase().indexOf(searchUpperCase) !== -1)
      })
    }
  },
  props: ['itemsList', 'selected']
}
</script>

<style>
@media only screen and (max-width: 767px) {
  .border-top-color-sm {
    border-top-color: #929499;
  }
}
</style>
