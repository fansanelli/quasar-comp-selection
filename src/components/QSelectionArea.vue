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
                <q-radio v-model="include" :val="true" label="Include" />
                <q-radio v-model="include" :val="false" label="Exclude" />
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
      selectedItemsListValues: [],
      include: true,
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
          if (typeof this.$refs.infiniteScroll !== 'undefined') {
            this.$refs.infiniteScroll.stop()
          }
        }
        done()
      }, 300)
    },
    getListOfSelectedItems () {
      const include = this.include
      return this.itemsList.filter((item) => {
        return include ? this.selectedItemsListValues.indexOf(item.value) !== -1
          : this.selectedItemsListValues.indexOf(item.value) === -1
      })
    },
    add (value) {
      --this.nrShownResults
      this.selectedItemsListValues.push(value)
      this.$emit('update:selected', this.getListOfSelectedItems())
    },
    remove (value) {
      ++this.nrShownResults
      this.selectedItemsListValues.splice(this.selectedItemsListValues.indexOf(value), 1)
      this.$emit('update:selected', this.getListOfSelectedItems())
    },
    selectAll () {
      this.nrShownResults = this.itemsList.length
      this.selectedItemsListValues = this.itemsList.map((item) => { return item.value })
      this.$emit('update:selected', this.getListOfSelectedItems())
    },
    reset () {
      this.nrShownResults = 0
      this.selectedItemsListValues = []
      this.$emit('update:selected', this.getListOfSelectedItems())
    }
  },
  computed: {
    selectedItemsList () {
      return this.itemsList.filter((item) => {
        return this.selectedItemsListValues.indexOf(item.value) !== -1
      })
    },
    unselectedItemsList () {
      const search = this.search.toUpperCase()
      return this.itemsList.filter((item) => {
        return this.selectedItemsListValues.indexOf(item.value) === -1 &&
          (search.length === 0 || item.label.toUpperCase().indexOf(search) !== -1)
      })
    }
  },
  watch: {
    include () {
      this.$emit('update:selected', this.getListOfSelectedItems())
    },
    search () {
      this.nrShownResults = this.selectedItemsList.length
      if (typeof this.$refs.infiniteScroll !== 'undefined') {
        this.$refs.infiniteScroll.resume()
        this.$refs.infiniteScroll.loadMore()
      }
    },
    nrShownResults () {
      if (this.nrShownResults === 0) {
        if (typeof this.$refs.infiniteScroll !== 'undefined') {
          this.$refs.infiniteScroll.resume()
          this.$refs.infiniteScroll.loadMore()
        }
      }
    }
  },
  created: function () {
    this.beginSelected.forEach(el => {
      this.add(el)
    })
  },
  props: ['itemsList', 'beginSelected']
}
</script>
