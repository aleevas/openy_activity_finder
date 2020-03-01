<template>
  <div id="activity-finder-app">
    <WizardBar
      v-if="wizardSteps.includes(step)"
      :ages="ages"
      :days="days"
      :times="times"
      :days-times="daysTimes"
      :locations="locations"
      :activities="activities"
      :selected-ages="selectedAges"
      :selected-days="selectedDays"
      :selected-times="selectedTimes"
      :selected-days-times="selectedDaysTimes"
      :selected-locations="selectedLocations"
      :selected-activities="selectedActivities"
      @startOver="startOver()"
      @viewResults="viewResults()"
    />
    <ResultsBar v-if="step === 'results'" class="hidden-md hidden-lg">
      <template v-slot:search>
        <SearchForm v-model="searchKeywords" />
      </template>
      <template v-slot:filter>
        <Filters
          id="mobile"
          :data="data"
          :ages="ages"
          :days="days"
          :times="times"
          :locations="locations"
          :activities="activities"
          :initial-ages="selectedAges"
          :initial-days="selectedDays"
          :initial-times="selectedTimes"
          :initial-locations="selectedLocations"
          :initial-activities="selectedActivities"
          :max-ages="maxAges"
          :is-loading-data="isLoadingData"
          @filterChange="changeFilter($event)"
          @clearFilters="clearFilters()"
        />
      </template>
      <template v-slot:sort>
        <SortRadios v-model="selectedSort" :sort-options="sortOptions" />
      </template>
    </ResultsBar>
    <Loading v-if="step !== 'results' && isLoadingData" />
    <SelectPath
      v-else-if="step === 'selectPath'"
      v-model="selectedPath"
      :paths="paths"
      @nextStep="nextStep('selectPath')"
    />
    <SelectAges
      v-else-if="step === 'selectAges'"
      v-model="selectedAges"
      :ages="ages"
      :max-ages="maxAges"
      :facets="data.facets.static_age_filter"
      @nextStep="nextStep('selectAges')"
    />
    <SelectDays
      v-else-if="step === 'selectDays'"
      v-model="selectedDays"
      :days="days"
      :facets="data.facets.days_of_week"
      @nextStep="nextStep('selectDays')"
    />
    <SelectTimes
      v-else-if="step === 'selectTimes'"
      v-model="selectedTimes"
      :times="times"
      :facets="data.facets.af_parts_of_day"
      @nextStep="nextStep('selectTimes')"
    />
    <SelectDaysTimes
      v-else-if="step === 'selectDaysTimes'"
      v-model="selectedDaysTimes"
      :days-times="daysTimes"
      :facets="data.facets.af_weekdays_parts_of_day"
      @nextStep="nextStep('selectDaysTimes')"
    />
    <SelectLocations
      v-else-if="step === 'selectLocations'"
      v-model="selectedLocations"
      :locations="locations"
      :facets="data.facets.locations"
      @nextStep="nextStep('selectLocations')"
    />
    <SelectActivities
      v-else-if="step === 'selectActivities'"
      v-model="selectedActivities"
      :activities="activities"
      :facets="data.facets.field_activity_category"
      @nextStep="nextStep('selectActivities')"
    />
    <Results
      v-else-if="step === 'results'"
      :data="data"
      :ages="ages"
      :selected-ages="selectedAges"
      :is-loading-data="isLoadingData"
      :cart-items="cartItems"
      @startOver="startOver()"
      @addItem="addItem($event)"
      @removeItem="removeItem($event)"
    >
      <template v-slot:search>
        <SearchForm v-model="searchKeywords" />
      </template>
      <template v-slot:filters>
        <Filters
          :data="data"
          :ages="ages"
          :days="days"
          :times="times"
          :locations="locations"
          :activities="activities"
          :initial-ages="selectedAges"
          :initial-days="selectedDays"
          :initial-times="selectedTimes"
          :initial-locations="selectedLocations"
          :initial-activities="selectedActivities"
          :max-ages="maxAges"
          @filterChange="changeFilter($event)"
          @clearFilters="clearFilters"
        />
      </template>
      <template v-slot:sort>
        <SortSelect v-model="selectedSort" :sort-options="sortOptions" />
      </template>
      <template v-slot:pager>
        <Pager v-model="selectedPage" :total-pages="data.pager_info.total_pages" />
      </template>
      <template v-slot:no-results>
        <NoResults
          :selected-ages="selectedAges"
          :selected-days="selectedDays"
          :selected-times="selectedTimes"
          :selected-days-times="selectedDaysTimes"
          :selected-locations="selectedLocations"
          :selected-activities="selectedActivities"
          @noResultsChoice="noResultsChoice($event)"
        />
      </template>
    </Results>
  </div>
</template>

<script>
import client from '@/client/index.js'
import SelectPath from '@/components/steps/SelectPath.vue'
import SelectAges from '@/components/steps/SelectAges.vue'
import SelectDays from '@/components/steps/SelectDays.vue'
import SelectTimes from '@/components/steps/SelectTimes.vue'
import SelectDaysTimes from '@/components/steps/SelectDaysTimes.vue'
import SelectLocations from '@/components/steps/SelectLocations.vue'
import SelectActivities from '@/components/steps/SelectActivities.vue'
import Results from '@/components/Results.vue'
import Loading from '@/components/Loading.vue'
import ResultsBar from '@/components/ResultsBar.vue'
import WizardBar from '@/components/WizardBar.vue'
import Pager from '@/components/filters/Pager.vue'
import Filters from '@/components/filters/Filters'
import SortRadios from '@/components/filters/SortRadios'
import SortSelect from '@/components/filters/SortSelect'
import SearchForm from '@/components/filters/SearchForm'
import NoResults from '@/components/NoResults.vue'

export default {
  name: 'ActivityFinder',
  components: {
    SelectPath,
    SelectAges,
    SelectDays,
    SelectTimes,
    SelectDaysTimes,
    SelectLocations,
    SelectActivities,
    Results,
    Loading,
    ResultsBar,
    WizardBar,
    Pager,
    Filters,
    SortRadios,
    SortSelect,
    SearchForm,
    NoResults
  },
  props: {
    ages: {
      type: Array,
      required: true
    },
    days: {
      type: Array,
      required: true
    },
    times: {
      type: Array,
      required: true
    },
    daysTimes: {
      type: Array,
      required: true
    },
    categories: {
      type: Array,
      required: true
    },
    categoriesType: {
      type: String,
      required: true
    },
    activities: {
      type: Array,
      required: true
    },
    locations: {
      type: Array,
      required: true
    },
    sortOptions: {
      type: Array,
      required: true
    },
    legacyMode: {
      type: Number,
      required: true
    },
    maxAges: {
      type: Number,
      default: 2
    }
  },
  data() {
    let data = {
      // Flag to show if the data request is in progress.
      isLoadingData: false,
      // Controls if we can load data now - we can load only in between the steps or on results step.
      canLoadData: true,
      // Search parameters of the last request casted to string primitive.
      lastRequestParamsString: '',
      data: {},
      steps: [
        'selectPath',
        'selectAges',
        'selectActivities',
        // 'selectTimes',
        // 'selectDays',
        'selectDaysTimes',
        'selectLocations',
        'results'
      ],
      completedSteps: [],
      wizardSteps: [
        'selectAges',
        'selectActivities',
        'selectTimes',
        'selectDays',
        'selectDaysTimes',
        'selectLocations'
      ],
      paths: [
        {
          id: 'selectAges',
          name: 'Find by Age'
        },
        {
          // id: 'selectTimes',
          // name: 'Find by Time and Day'
          id: 'selectDaysTimes',
          name: 'Find by Day and Time'
        },
        {
          id: 'selectLocations',
          name: 'Find by Location'
        },
        {
          id: 'selectActivities',
          name: 'Find by Activity'
        }
      ],
      selectedPath: '',
      defaults: {
        step: 'selectPath',
        selectedAges: [],
        selectedDays: [],
        selectedTimes: [],
        selectedDaysTimes: [],
        selectedLocations: [],
        selectedActivities: [],
        selectedPage: 1,
        selectedSort: 'af_time_of_day__ACS',
        searchKeywords: ''
      },
      cartItems: [],
      cartItemsKey: 'activity_finder.cartItems',
      clearFiltersSkip: ['step', 'selectedSort']
    }

    // Initialize reactive properties for every default value.
    for (let key in data.defaults) {
      data[key] = data.defaults[key]
    }

    return data
  },
  computed: {
    // Search parameters for data request.
    searchParams() {
      return {
        ages: this.selectedAges.join(','),
        days: this.selectedDays.join(','),
        times: this.selectedTimes.join(','),
        daystimes: this.selectedDaysTimes.join(','),
        locations: this.selectedLocations.join(','),
        categories: this.selectedActivities.join(','),
        page: this.selectedPage,
        sort: this.selectedSort,
        keywords: this.searchKeywords
      }
    },
    // Search parameters casted to string primitive.
    searchParamsString() {
      return [
        ...this.selectedAges,
        ...this.selectedDays,
        ...this.selectedTimes,
        ...this.selectedDaysTimes,
        ...this.selectedLocations,
        ...this.selectedActivities,
        this.selectedPage,
        this.selectedSort,
        this.searchKeywords
      ].join('_')
    },
    // Reset page parameters casted to string primitive.
    resetPageString() {
      return [
        ...this.selectedAges,
        ...this.selectedDays,
        ...this.selectedTimes,
        ...this.selectedDaysTimes,
        ...this.selectedLocations,
        ...this.selectedActivities,
        this.selectedSort,
        this.searchKeywords
      ].join('_')
    },
    // Parameters to update the url casted to string primitive.
    updateUrlString() {
      return [
        ...this.selectedAges,
        ...this.selectedDays,
        ...this.selectedTimes,
        ...this.selectedDaysTimes,
        ...this.selectedLocations,
        ...this.selectedActivities,
        this.selectedPage,
        this.selectedSort,
        this.step,
        this.searchKeywords
      ].join('_')
    },
    // Controls if the search parameters were changed since last data request.
    shouldUpdateData() {
      return this.lastRequestParamsString !== this.searchParamsString
    }
  },
  watch: {
    searchParamsString() {
      // If data is empty then the page is just loading and we shouldn't react to changes yet.
      if (Object.keys(this.data).length === 0) {
        return
      }

      if (this.step === 'results') {
        this.canLoadData = true
      }
    },
    resetPageString() {
      // If data is empty then the page is just loading and we shouldn't react to changes yet.
      if (Object.keys(this.data).length === 0) {
        return
      }

      this.selectedPage = this.defaults.selectedPage
    },
    updateUrlString() {
      this.updateUrl()
    },
    step(val, oldVal) {
      // Check if we returned to previously completed step - using browser back button - then we
      // need to remove this step from completed steps.
      // The else part tracks the completed step if the first (default) step was already completed
      // or if we have just completed this default step.
      if (this.completedSteps.includes(val)) {
        this.completedSteps = this.completedSteps.filter(x => x !== val)
      } else if (
        this.completedSteps.includes(this.defaults.step) ||
        oldVal === this.defaults.step
      ) {
        this.completedSteps.push(oldVal)
      }
      this.canLoadData = true
    },
    canLoadData() {
      this.loadData()
    },
    shouldUpdateData() {
      this.loadData()
    },
    $route: {
      handler() {
        this.getDataFromUrl()
      },
      immediate: true
    },
    cartItems() {
      localStorage.setItem(this.cartItemsKey, JSON.stringify(this.cartItems))
    }
  },
  created() {
    this.loadData()
  },
  mounted() {
    if (localStorage.getItem(this.cartItemsKey)) {
      try {
        this.cartItems = JSON.parse(localStorage.getItem(this.cartItemsKey))
      } catch (e) {
        localStorage.removeItem(this.cartItemsKey)
      }
    }
  },
  methods: {
    nextStep(step) {
      // First step should be the one chosen by the user - selectedPath.
      if (!this.completedSteps.includes(this.selectedPath) && this.step !== this.selectedPath) {
        this.step = this.selectedPath
      } else {
        // selectTimes and selectDays steps should go one after another.
        if (step === 'selectTimes') {
          this.step = 'selectDays'
        } else {
          // Find the next step - exclude completed steps and current step from the steps list and get the first one.
          const difference = this.steps.filter(
            x => !this.completedSteps.includes(x) && x !== this.step
          )
          this.step = difference[0]
        }
      }
    },
    viewResults() {
      this.step = 'results'
    },
    loadData() {
      if (!this.canLoadData) {
        return
      }
      if (!this.shouldUpdateData) {
        this.canLoadData = false
        return
      }
      if (this.isLoadingData) {
        return
      }

      this.canLoadData = false
      this.lastRequestParamsString = this.searchParamsString

      this.isLoadingData = true
      client.request({ params: this.searchParams }).then(response => {
        this.data = response.data
        this.isLoadingData = false

        // If there were other changes while the request was in progress - we should load data again.
        this.loadData()
      })
    },
    getDataFromUrl() {
      const query = this.$route.query
      for (let key in this.defaults) {
        if (query.hasOwnProperty(key) && query[key] !== '') {
          if (Array.isArray(this.defaults[key])) {
            this[key] = query[key].split(',')
          } else {
            if (key === 'selectedPage') {
              // selectedPage should be number, not a string.
              this[key] = parseInt(query[key])
            } else {
              this[key] = query[key]
            }

            // If completedSteps is empty then we should initialize it.
            if (key === 'step' && this.completedSteps.length === 0) {
              this.completedSteps.push(this.defaults.step)
              this.selectedPath = query[key]
            }
          }
        } else {
          this[key] = this.defaults[key]
        }
      }
    },
    updateUrl() {
      // Prepare query with non-default values only.
      let query = {}
      for (let key in this.defaults) {
        if (Array.isArray(this.defaults[key])) {
          if (this[key].length !== 0) {
            query[key] = this[key].join(',')
          }
        } else {
          if (this[key] !== this.defaults[key]) {
            query[key] = this[key]
          }
        }
      }

      this.$router
        .push({
          query
        })
        // TODO: is there any good way to detect if we are already at this router location?
        // Catch to avoid "NavigationDuplicated" error.
        .catch(err => {
          err
        })
    },
    changeFilter(event) {
      this[event.filter] = event.value
    },
    clearFilters() {
      for (let key in this.defaults) {
        if (this.clearFiltersSkip.includes(key)) {
          continue
        }
        this[key] = this.defaults[key]
      }
    },
    startOver() {
      for (let key in this.defaults) {
        this[key] = this.defaults[key]
      }

      this.completedSteps = []
      this.selectedPath = ''
      this.cartItems = []
    },
    addItem(item) {
      this.cartItems.push(item)
    },
    removeItem(index) {
      this.cartItems.splice(index, 1)
    },
    noResultsChoice(choice) {
      const skip = [...this.clearFiltersSkip, choice]
      for (let key in this.defaults) {
        if (skip.includes(key)) {
          continue
        }
        this[key] = this.defaults[key]
      }
    }
  }
}
</script>

<style lang="scss">
// Overrides for external CSS rules.
.landing-content {
  padding-top: 0 !important;
}
</style>