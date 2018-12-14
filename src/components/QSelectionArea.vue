<template>
  <div>
    <div class="q-mb-lg">
      <div class="text-right text-primary">
        <q-btn label="Select all" @click="selectAll" />
        <q-btn label="Reset" @click="reset" />
      </div>
    </div>
      <div class="row" style="height: 70vh !important;">
        <q-list inset-delimiter class="col-12 col-md-6">
          <q-list-header>
            <div class="row justify-between">
              Selected
              <div>
                <q-btn size="sm" text-color="primary" label="Toggle selection" @click="toggle" />
              </div>
            </div>
          </q-list-header>
          <q-item separator :key="item.value" v-for="item in selectedItemsList">
            <q-item-main label @click.native="remove(item.value)">
              {{item.label}}
            </q-item-main>
          </q-item>
        </q-list>
        <q-list inset-delimiter class="col-12 col-md-6 text-center">
          <q-list-header>
            Available
            <q-field>
              <q-search
                v-model="search"
                clearable
                class="q-mb-sm"
                :debounce="250"
              />
            </q-field>
          </q-list-header>
          <q-infinite-scroll
            :handler="loadMore"
            inline
            style="height: 50vh !important; overflow: auto;"
            ref="infiniteScroll"
          >
            <q-item separator :key="item.value" v-for="(item, key) in unselectedItemsList" v-if="key < nrShownResults">
              <q-item-main label @click.native="add(item.value)">
                {{item.label}}
              </q-item-main>
            </q-item>

            <q-spinner-dots slot="message" :size="40" ref="spinner"></q-spinner-dots>
          </q-infinite-scroll>
        </q-list>
      </div>
  </div>
</template>

<script>
const MAX_RESULTS = 25

export default {
  name: 'q-selection-area',
  data: function () {
    return {
      search: '',
      nrShownResults: 0
    }
  },
  methods: {
    loadMore (_, done) {
      setTimeout(() => {
        if (this.unselectedItemsList.length - this.nrShownResults > MAX_RESULTS) {
          this.nrShownResults += MAX_RESULTS
        } else {
          this.nrShownResults = this.unselectedItemsList.length
          this.stopScrolling()
        }
        done()
      }, 300)
    },
    add (value) {
      ++this.nrShownResults
      this.$emit('update:selected', this.selected.concat(value))
    },
    remove (value) {
      --this.nrShownResults
      this.$emit('update:selected', this.selected.filter((f) => { return f.indexOf(value) === -1 }))
    },
    selectAll () {
      this.nrShownResults = this.itemsList.length
      this.$emit('update:selected', this.itemsList.map((item) => { return item.value }))
    },
    reset () {
      this.nrShownResults = 0
      this.$emit('update:selected', [])
      this.resumeScrolling()
    },
    toggle () {
      this.$emit('update:selected', this.itemsList.filter((f) => {
        return this.selected.indexOf(f.value) === -1
      }).map((item) => {
        return item.value
      }))
      this.nrShownResults = this.selectedItemsList.length
    },
    stopScrolling () {
      if (typeof this.$refs.infiniteScroll !== 'undefined') {
        this.$refs.infiniteScroll.stop()
      }
    },
    resumeScrolling () {
      if (typeof this.$refs.infiniteScroll !== 'undefined') {
        this.$refs.infiniteScroll.resume()
        this.$refs.infiniteScroll.loadMore()
      }
    }
  },
  computed: {
    selectedItemsList () {
      return this.itemsList.filter((item) => {
        return this.selected.indexOf(item.value) !== -1
      })
    },
    unselectedItemsList () {
      const search = this.search.toUpperCase()
      return this.itemsList.filter((item) => {
        return this.selected.indexOf(item.value) === -1 &&
          (search.length === 0 || item.label.toUpperCase().indexOf(search) !== -1)
      })
    }
  },
  watch: {
    search () {
      this.resumeScrolling()
    }
  },
  props: ['itemsList', 'selected']
}
</script>
