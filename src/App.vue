<template>
  <section>
    <div class="hero is-primary">
      <div class="hero-body">
        <div class="container">
          <h1 class="title">Blaydon CC</h1>
          <h2 class="subtitle">Club Time Trial</h2>
        </div>
      </div>
      <div class="hero-foot">
        <nav class="tabs is-boxed">
          <div class="container">
            <ul>
              <li v-bind:class="{ 'is-active': category === '*'}"><a v-on:click="filter('*')">All</a></li>
              <li v-bind:class="{ 'is-active': category === 'W'}"><a v-on:click="filter('W')">Women</a></li>
              <li v-bind:class="{ 'is-active': category === 'S'}"><a v-on:click="filter('S')">Senior</a></li>
              <li v-bind:class="{ 'is-active': category === 'V'}"><a v-on:click="filter('V')">Vets</a></li>
              <li v-bind:class="{ 'is-active': category === 'R'}"><a v-on:click="filter('R')">Race Team</a></li>
              <li v-bind:class="{ 'is-active': category === 'TT'}"><a v-on:click="filter('TT')">TT</a></li>
            </ul>
          </div>
        </nav>
      </div>
    </div>

    <div class="container">
      <div class="content">
        <br>
        <b-table
            :data="filteredResults"
            :bordered=true
            :striped=true
            :narrowed=true
            :selectable=true>

            <b-table-column field="Name" label="Name" sortable/>
            <b-table-column field="Category" label="Category" width="50" sortable :format="formatCategory"/>
            <b-table-column field="Round 1" label="1" sortable :custom-sort="sortRound1"/>
            <b-table-column field="Round 2" label="2" sortable :custom-sort="sortRound2"/>
            <b-table-column field="Round 3" label="3" sortable :custom-sort="sortRound3"/>
            <b-table-column field="Round 4" label="4" sortable :custom-sort="sortRound4"/>
            <b-table-column field="Round 5" label="5" sortable :custom-sort="sortRound5"/>
            <b-table-column field="Round 6" label="6" sortable :custom-sort="sortRound6"/>
            <b-table-column field="Round 7" label="7" sortable :custom-sort="sortRound7"/>
            <b-table-column field="Round 8" label="8" sortable :custom-sort="sortRound8"/>
        </b-table>
      </div>
    </div>

    <footer class="footer">
      <div class="container">
        <div class="content">
          <p>Made by <a href="https://twitter.com/justinramel" target="_blank">Justin Ramel</a>.</p>
          </div>
        </div>
    </footer>
  </section>
</template>

<script>
  /* global Miso */
  export default {
    name: 'app',
    data () {
      return {
        headers: [],
        category: '*',
        categories: {
          W: 'Women',
          S: 'Seniors',
          V: 'Vets',
          R: 'Race Team',
          TT: 'TT'
        },
        results: [],
        filteredResults: []
      }
    },
    created () {
      let that = this
      const ds = new Miso.Dataset({
        importer: Miso.Dataset.Importers.GoogleSpreadsheet,
        parser: Miso.Dataset.Parsers.GoogleSpreadsheet,
        key: '1G9nHxp6P_jrfzNzfrV8CMhclA-g3HqS5ccNBAyncAAk',
        worksheet: '1'
      })

      ds.fetch({
        success: function () {
          that.headers = ds.columnNames()
          this.each(function (row) {
            that.results.push(row)
          })
          that.filteredResults = that.results
        },
        error: function (err) {
          console.dir(err)
          console.log('Are you sure you are connected to the internet?')
        }
      })
    },
    methods: {
      filter (category) {
        this.category = category
        this.filteredResults = this.results.filter(value => category === '*' ? true : value.Category === category)
      },
      formatCategory (category) {
        return this.categories[category]
      },
      sortRound1: sortTime('Round 1'),
      sortRound2: sortTime('Round 2'),
      sortRound3: sortTime('Round 3'),
      sortRound4: sortTime('Round 4'),
      sortRound5: sortTime('Round 5'),
      sortRound6: sortTime('Round 6'),
      sortRound7: sortTime('Round 7'),
      sortRound8: sortTime('Round 8')
    }
  }

  function sortTime (columnName) {
    return function sort (a, b) {
      const colA = a[columnName]
      const colB = b[columnName]
      if (!isNumeric(colA)) return 1
      if (!isNumeric(colB)) return -1
      return colA - colB
    }
  }

  function isNumeric (n) {
    return !isNaN(parseFloat(n)) && isFinite(n)
  }

</script>

<style lang="sass">
@import "~bulma/sass/utilities/_all"
// Set your colors
$blaydon: #a6192e
$primary: $blaydon
$primary-invert: findColorInvert($primary)


// Links
$link: $primary
$link-invert: $primary-invert
$link-focus-border: $primary

@import "~bulma"
@import "~buefy/src/scss/buefy"

.hero.is-primary
    background-color: $blaydon
    color: #fff

.hero.is-primary .tabs.is-boxed li.is-active a, .hero.is-primary .tabs.is-boxed li.is-active a:hover, .hero.is-primary .tabs.is-toggle li.is-active a, .hero.is-primary .tabs.is-toggle li.is-active a:hover
    color: $blaydon !important
</style>
