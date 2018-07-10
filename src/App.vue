<template>
  <section>
    <div class="hero is-primary">
      <div class="hero-body">
        <div class="container">
          <div class="columns">
            <div class="column is-half">
              <h1 class="title">Blaydon CC</h1>
              <h2 class="subtitle">Club Time Trial 2018</h2>
              <p>To compete for a trophy you must:</p>
              <div class="column is-offset-1">
                <ul>
                  <li><b-icon icon="done" type="is-success"></b-icon> 3 x <a href="https://ridewithgps.com/routes/6614834" target="_blank">M12</a></li>
                  <li><b-icon icon="done" type="is-success"></b-icon> 1 x Marshal</li>
                </ul>
              </div>
            </div>
            <div class="column">
              <h1 class="title">Leaderboard</h1>
              <table>
                <tr v-for="result in leaderboard">
                  <td>{{result.category}}</td>
                  <td>{{result.name}}</td>
                  <td>{{result.time | formatTime}}</td>
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
              <li v-bind:class="{ 'is-active': category === 'Women'}"><a v-on:click="filter('Women')">Women</a></li>
              <li v-bind:class="{ 'is-active': category === 'Senior Road'}"><a v-on:click="filter('Senior Road')">Senior Road</a></li>
              <li v-bind:class="{ 'is-active': category === 'Vet Road'}"><a v-on:click="filter('Vet Road')">Vet Road</a></li>
              <li v-bind:class="{ 'is-active': category === 'Senior TT'}"><a v-on:click="filter('Senior TT')">Senior TT</a></li>
              <li v-bind:class="{ 'is-active': category === 'Vet TT'}"><a v-on:click="filter('Vet TT')">Vet TT</a></li>
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

          <template scope="props">
            <b-table-column field="Name" label="Name" sortable>
              {{ props.row.Name }}
            </b-table-column>
            <b-table-column field="M12 Best" label="M12 Best" sortable :custom-sort="sortM12Best" :format="formatTime">
              {{ props.row['M12 Best'] }}
            </b-table-column>
            <b-table-column field="M12 2nd Best" label="M12 2nd Best" sortable :custom-sort="sortM122ndBest" :format="formatTime">
              {{ props.row['M12 2nd Best'] }}
            </b-table-column>
            <b-table-column field="M2511" label="M12 3rd Best" sortable :custom-sort="sortM2511" :format="formatTime">
              {{ props.row.M2511 }}
            </b-table-column>
            <b-table-column field="Total" label="Total" sortable :custom-sort="sortTotal" :format="formatTime">
              {{ props.row.Total }}
            </b-table-column>
            <b-table-column field="Category" label="Category" sortable>
              {{ props.row.Category }}
            </b-table-column>
            <b-table-column field="Marshalled" label="Marshal" width="1" sortable>
              <center>
                <b-icon :icon="props.row.Marshalled === 'Y' ? 'done' : ''" type="is-success"></b-icon>
              </center>
            </b-table-column>
            <b-table-column label="Complete" width="1" sortable>
              <center>
                <b-icon :icon="complete(props.row) ? 'done_all' : ''" type="is-success"></b-icon>
              </center>
            </b-table-column>
          </template>
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
        key: '1gcSCNA-MhatgS8UhmYbcwhewpiu8wd2-a9YLqOC7T0A',
        worksheet: '1'
      })

      ds.fetch({
        success: function () {
          that.headers = ds.columnNames()
          this.each(function (row) {
            that.results.push(row)
          })
          that.results.sort(sortByLeader)
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
      formatTime: formatTime,
      complete: complete,
      sortM12Best: sortTime('M12 Best'),
      sortM122ndBest: sortTime('M12 2nd Best'),
      sortM2511: sortTime('M2511'),
      sortTotal: sortTime('Total')
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
      return asDate(a[columnName]) - asDate(b[columnName])
    }
  }

  function sortByLeader (a, b) {
    if (complete(a) && complete(b)) {
      return asDate(a.Total) - asDate(b.Total)
    }
    if (complete(a)) return -1
    if (complete(b)) return 1

    if (notMarshalled(a) && notMarshalled(b)) {
      return asDate(a.Total) - asDate(b.Total)
    }
    if (notMarshalled(a)) return -1
    if (notMarshalled(b)) return 1

    if (m12Complete(a) && m12Complete(b)) {
      return asDate(a.Total) - asDate(b.Total)
    }
    if (m12Complete(a)) return -1
    if (m12Complete(b)) return 1

    return asDate(a['M12 Best']) - asDate(b['M12 Best'])
  }

  function asDate (result = '') {
    if (result === null) result = ''
    const time = result.length === 8 ? result : `0${result}`
    return new Date(`Wed Apr 26 2017 ${time} GMT+0100 (BST)`)
  }

  function isNumeric (n) {
    return !isNaN(parseFloat(n)) && isFinite(n)
  }

  function empty (data) {
    return (!data || data.length === 0)
  }

  function createLeaderboard (results) {
    return [
      {category: 'Women', ...first(results.filter(r => r.Category === 'Women'))},
      {category: 'Senior Road', ...first(results.filter(r => r.Category === 'Senior Road'))},
      {category: 'Vet Road', ...first(results.filter(r => r.Category === 'Vet Road'))},
      {category: 'Senior TT', ...first(results.filter(r => r.Category === 'Senior TT'))},
      {category: 'Vet TT', ...first(results.filter(r => r.Category === 'Vet TT'))}
    ].sort(sortTime('time'))
  }

  function first (results) {
    if (results instanceof Array && results.length > 0) {
      return {
        name: results[0].Name,
        time: results[0]['Total']
      }
    } else {
      return {
        name: 'Could be you!',
        time: '--.--'
      }
    }
  }

  function complete (result) {
    return !empty(result['M12 Best']) && !empty(result['M12 2nd Best']) && !empty(result['M2511']) && result['Marshalled'] === 'Y'
  }
  function notMarshalled (result) {
    return !empty(result['M12 Best']) && !empty(result['M12 2nd Best']) && !empty(result['M2511'])
  }
  function m12Complete (result) {
    return !empty(result['M12 Best']) && !empty(result['M12 2nd Best'])
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
