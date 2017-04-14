<template>
  <section>
    <div class="hero is-primary">
      <div class="hero-body">
        <div class="container">
          <div class="columns">
            <div class="column is-two-thirds">
              <h1 class="title">Blaydon CC</h1>
              <h2 class="subtitle">Club Time Trial 2016</h2>
            </div>
            <div class="column">
              <h1 class="title">Leaderboard</h1>
              <table>
                <tr v-for="result in leaderboard">
                  <td>{{result.category}}</td>
                  <td>{{result.leader.name}}</td>
                  <td>{{result.leader.time | formatTime}}</td>
                </tr>
              </table>
            </div>
          </div>
        </div>
      </div>
      <div class="hero-foot">
        <nav class="tabs is-boxed">
          <div class="container">
            <ul>
              <li v-bind:class="{ 'is-active': category === '*'}"><a v-on:click="filter('*')">All</a></li>
              <li v-bind:class="{ 'is-active': category === 'W'}"><a v-on:click="filter('W')">Women</a></li>
              <li v-bind:class="{ 'is-active': category === 'S'}"><a v-on:click="filter('S')">Senior</a></li>
              <li v-bind:class="{ 'is-active': category === 'V'}"><a v-on:click="filter('V')">Vet</a></li>
              <li v-bind:class="{ 'is-active': category === 'R'}"><a v-on:click="filter('R')">Race Team</a></li>
              <li v-bind:class="{ 'is-active': category === 'TT'}"><a v-on:click="filter('TT')">TT</a></li>
            </ul>
          </div>
        </nav>
      </div>
    </div>

    <div class="container">
      <div class="content">
          <div class="icon-at-kom-1"></div>
        <br>
        <b-table
            :data="filteredResults"
            :bordered=true
            :striped=true
            :narrowed=true
            :selectable=true>

            <b-table-column field="Name" label="Name" sortable/>
            <b-table-column field="Category" label="Category" width="50" sortable :format="formatCategory"/>
            <b-table-column field="Round 1" label="1" sortable :custom-sort="sortRound1" :format="formatTime"/>
            <b-table-column field="Round 2" label="2" sortable :custom-sort="sortRound2" :format="formatTime"/>
            <b-table-column field="Round 3" label="3" sortable :custom-sort="sortRound3" :format="formatTime"/>
            <b-table-column field="Round 4" label="4" sortable :custom-sort="sortRound4" :format="formatTime"/>
            <b-table-column field="Round 5" label="5" sortable :custom-sort="sortRound5" :format="formatTime"/>
            <b-table-column field="Round 6" label="6" sortable :custom-sort="sortRound6" :format="formatTime"/>
            <b-table-column field="Round 7" label="7" sortable :custom-sort="sortRound7" :format="formatTime"/>
            <b-table-column field="Round 8" label="8" sortable :custom-sort="sortRound8" :format="formatTime"/>
            <b-table-column field="Best 2 (MMM:SS)" label="Best 2" sortable :custom-sort="sortBest" :format="formatTime"/>
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
          S: 'Senior',
          V: 'Vet',
          R: 'Race Team',
          TT: 'TT'
        },
        results: [],
        filteredResults: [],
        leaderboard: []
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
          that.results.sort(sortTime('Best 2 (MMM:SS)'))
          that.leaderboard = createLeaderboard(that.results)
          that.filteredResults = that.results
        },
        error: function (err) {
          console.log(err)
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
      formatTime: formatTime,
      sortRound1: sortTime('Round 1'),
      sortRound2: sortTime('Round 2'),
      sortRound3: sortTime('Round 3'),
      sortRound4: sortTime('Round 4'),
      sortRound5: sortTime('Round 5'),
      sortRound6: sortTime('Round 6'),
      sortRound7: sortTime('Round 7'),
      sortRound8: sortTime('Round 8'),
      sortBest: sortTime('Best 2 (MMM:SS)')
    },
    filters: {
      formatTime: formatTime
    }
  }

  function formatTime (time) {
    if (!isNumeric(time)) return time
    return parseFloat(Math.round(time * 100) / 100).toFixed(2)
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

  function createLeaderboard (results) {
    return [
      {category: 'Women', leader: first(results.filter(r => r.Category === 'W'))},
      {category: 'Senior', leader: first(results.filter(r => r.Category === 'S'))},
      {category: 'Vet', leader: first(results.filter(r => r.Category === 'V'))},
      {category: 'Race Team', leader: first(results.filter(r => r.Category === 'R'))},
      {category: 'TT', leader: first(results.filter(r => r.Category === 'TT'))}
    ]
  }

  function first (results) {
    if (results instanceof Array && results.length > 0) {
      return {
        name: results[0].Name,
        time: results[0]['Best 2 (MMM:SS)']
      }
    } else {
      return {
        name: 'Could be you!',
        time: '--.--'
      }
    }
  }
</script>

<style lang="sass">
@import "~bulma/sass/utilities/_all"
// Setup colours
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

.icon-at-kom-1
    background-image: url(data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZlcnNpb249IjEuMSIgeD0iMCIgeT0iMCIgaGVpZ2h0PSI5NiIgd2lkdGg9IjI0IiB2aWV3Qm94PSIwIDAgMjQgOTYiIGVuYWJsZS1iYWNrZ3JvdW5kPSJuZXcgMCAwIDI0IDk2IiB4bWw6c3BhY2U9InByZXNlcnZlIj48cGF0aCBmaWxsPSIjRkVDODM1IiBkPSJNMjAuNCAzLjdjMCAyLTEuNiAzLjYtMy42IDMuNnMtMy42LTEuNi0zLjYtMy42aC0yLjNjMCAyLTEuNiAzLjYtMy42IDMuNlMzLjcgNS43IDMuNyAzLjdIMS40TDIuOSAxN2gxOC4ybDEuNi0xMy4zSDIwLjR6TTEyLjEgMTQuNGwtMi42LTIuNiAyLjYtMi42IDIuNiAyLjZMMTIuMSAxNC40ek0xOC4zIDEzLjJsLTEuNS0xLjUgMS41LTEuNSAxLjUgMS41TDE4LjMgMTMuMnpNNS45IDEzLjJsLTEuNS0xLjUgMS41LTEuNSAxLjUgMS41TDUuOSAxMy4yeiIvPjxwYXRoIGZpbGw9IiM2NjY2NjYiIGQ9Ik0yMC40IDI3LjdjMCAyLTEuNiAzLjYtMy42IDMuNnMtMy42LTEuNi0zLjYtMy42aC0yLjNjMCAyLTEuNiAzLjYtMy42IDMuNnMtMy42LTEuNi0zLjYtMy42SDEuNEwyLjkgNDFoMTguMmwxLjYtMTMuM0gyMC40ek0xMi4xIDM4LjRsLTIuNi0yLjYgMi42LTIuNiAyLjYgMi42TDEyLjEgMzguNHpNMTguMyAzNy4ybC0xLjUtMS41IDEuNS0xLjUgMS41IDEuNUwxOC4zIDM3LjJ6TTUuOSAzNy4ybC0xLjUtMS41IDEuNS0xLjUgMS41IDEuNUw1LjkgMzcuMnoiLz48cGF0aCBmaWxsPSIjRkZGRkZGIiBkPSJNMjAuNCA1MS43YzAgMi0xLjYgMy42LTMuNiAzLjZzLTMuNi0xLjYtMy42LTMuNmgtMi4zYzAgMi0xLjYgMy42LTMuNiAzLjZzLTMuNi0xLjYtMy42LTMuNkgxLjRMMi45IDY1aDE4LjJsMS42LTEzLjNIMjAuNHpNMTIuMSA2Mi40bC0yLjYtMi42IDIuNi0yLjYgMi42IDIuNkwxMi4xIDYyLjR6TTE4LjMgNjEuMmwtMS41LTEuNSAxLjUtMS41IDEuNSAxLjVMMTguMyA2MS4yek01LjkgNjEuMmwtMS41LTEuNSAxLjUtMS41IDEuNSAxLjVMNS45IDYxLjJ6Ii8%2BPC9zdmc%2BDQo%3D)


</style>
