<template>
  <Foldable
    :label="'Day(s)' | t"
    :collapse-id="id + '-toggle'"
    :counter="filtersCount"
    class="days-filter-component"
  >
    <div v-for="day in days" :key="id + '-day-' + day.value" class="option">
      <input
        :id="id + '-day-' + day.value"
        v-model="selectedDays"
        type="checkbox"
        :value="day.value"
      />
      <label :for="id + '-day-' + day.value">{{ day.search_value | capitalize }}</label>
    </div>
  </Foldable>
</template>

<script>
import Foldable from '@/components/Foldable'

export default {
  name: 'DaysFilter',
  components: {
    Foldable
  },
  props: {
    value: {
      type: Array,
      required: true
    },
    id: {
      type: String,
      required: true
    },
    days: {
      type: Array,
      required: true
    },
    facets: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      selectedDays: this.value
    }
  },
  computed: {
    filtersCount() {
      return this.selectedDays.length
    }
  },
  watch: {
    value() {
      this.selectedDays = this.value
    },
    selectedDays() {
      this.$emit('input', this.selectedDays)
    }
  },
  methods: {
    facetCount(value) {
      let facet = this.facets.find(x => x.filter === value)
      return facet ? facet.count : 0
    }
  }
}
</script>
