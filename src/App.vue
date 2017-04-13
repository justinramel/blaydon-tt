<template>
  <section>
    <div class="hero is-danger is-bold">
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
              <li class="is-active"><a>All</a></li>
              <li class=""><a>Women</a></li>
              <li class=""><a>Senior</a></li>
              <li class=""><a>Vets</a></li>
              <li class=""><a>Race Team</a></li>
              <li class=""><a>TT</a></li>
            </ul>
          </div>
        </nav>
      </div>
    </div>

<div class="container">

  <b-table
      :data="results"
      bordered=true
      striped=true
      narrowed=true
      selectable=true
      default-sort="Name">

      <b-table-column field="Name" label="Name" sortable></b-table-column>
      <b-table-column field="Category" label="Category" width="50" sortable></b-table-column>
      <b-table-column field="Round 1" label="1" sortable></b-table-column>
      <b-table-column field="Round 2" label="2" sortable></b-table-column>
      <b-table-column field="Round 3" label="3" sortable></b-table-column>
      <b-table-column field="Round 4" label="4" sortable></b-table-column>
      <b-table-column field="Round 5" label="5" sortable></b-table-column>
      <b-table-column field="Round 6" label="6" sortable></b-table-column>
      <b-table-column field="Round 7" label="7" sortable></b-table-column>
      <b-table-column field="Round 8" label="8" sortable></b-table-column>

  </b-table>

</div>



  <footer class="footer">
    <div class="container">
      <div class="content">
        <p>Made with <span class="icon is-unselectable is-danger"><i class="mdi">favorite</i></span> by <a href="https://twitter.com/justinramel" target="_blank">Justin Ramel</a>.</p> <p>
        </p>
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
        results: []
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
        },
        error: function () {
          console.log('Are you sure you are connected to the internet?')
        }
      })
    }
  }

</script>

<style lang="sass">
// Import Bulma's core
@import "~bulma/sass/utilities/_all";

// Set your colors
$primary: #a6192e;
$primary-invert: findColorInvert($primary);

// Links
$link: $primary;
$link-invert: $primary-invert;
$link-focus-border: $primary;

</style>
