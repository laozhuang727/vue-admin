<template>
  <div>
    <div class="tile is-ancestor">
      <div class="tile is-parent">
        <article class="tile is-child box">
          <div id="app" class="container">
            <!-- Example row of columns -->
            <h2 class="sub-header">List of Users</h2>
            <hr>
            <div class="row">
              <div class="col-md-5">
                <div class="form-inline form-group">
                  <label>Search:</label>
                  <input v-model="searchFor" class="form-control" @keyup.enter="setFilter">
                  <button class="btn btn-primary" @click="setFilter">Go</button>
                  <button class="btn btn-default" @click="resetFilter">Reset</button>
                </div>
              </div>
              <div class="col-md-7">
                <div class="dropdown form-inline pull-right">
                  <label>Pagination:</label>
                  <select class="form-control" v-model="paginationComponent">
                    <option value="vuetable-pagination">vuetable-pagination</option>
                    <option value="vuetable-pagination-dropdown">vuetable-pagination-dropdown</option>
                  </select>
                  <label>Per Page:</label>
                  <select class="form-control" v-model="perPage">
                    <option value=10>10</option>
                    <option value=15>15</option>
                    <option value=20>20</option>
                    <option value=25>25</option>
                  </select>
                  <button class="btn btn-default dropdown-toggle" data-toggle="dropdown">
                    <i class="glyphicon glyphicon-cog"></i>
                  </button>
                  <ul class="dropdown-menu">
                    <li v-for="field in fields">
                            <span class="checkbox">
                                <label>
                                    <input type="checkbox" v-model="field.visible">
                                    {{ field.title == '' ? field.name.replace('__', '') : field.title | capitalize }}
                                </label>
                            </span>
                    </li>
                  </ul>
                </div>
              </div>
            </div>
            <div class="table-responsive">
              <vuetable v-ref:vuetable
                        api-url="http://vuetable.ratiw.net/api/users"
                        pagination-path=""
                        :fields="fields"
                        :sort-order="sortOrder"
                        table-class="table table-bordered table-striped table-hover"
                        ascending-icon="glyphicon glyphicon-chevron-up"
                        descending-icon="glyphicon glyphicon-chevron-down"
                        pagination-class=""
                        pagination-info-class=""
                        pagination-component-class=""
                        :pagination-component="paginationComponent"
                        :item-actions="itemActions"
                        :per-page="perPage"
                        :append-params="moreParams"
                        wrapper-class="vuetable-wrapper "
                        table-wrapper=".vuetable-wrapper"
                        loading-class="loading"
              ></vuetable>
            </div>
          </div> <!-- /container -->
        </article>
      </div>
    </div>
  </div>
</template>
<script>
  var E_SERVER_ERROR = 'Error communicating with the server'

  //
  // secondly, require or import Vuetable and optional VuetablePagination component
  //
  import Vue from 'vue'
  import moment from 'moment'
  import sweetAlert from 'bootstrap-sweetalert'

  Vue.component('vuetable', require('vuetable/src/components/Vuetable.vue'))
  Vue.component('vuetable-pagination', require('vuetable/src/components/VuetablePagination.vue'))
  Vue.component('vuetable-pagination-dropdown', require('vuetable/src/components/VuetablePaginationDropdown.vue'))

  export default Vue.extend({
    data () {
      return {
        fields: [
          'name',
          'nickname',
          'email',
          'birthdate',
          'gender',
          '__actions'
        ],
        sortOrder: [{
          field: 'name',
          direction: 'asc'
        }],
        perPage: 10,
        paginationComponent: 'vuetable-pagination',
        paginationInfoTemplate: 'แสดง {from} ถึง {to} จากทั้งหมด {total} รายการ',
        itemActions: [
          {name: 'view-item', label: '', icon: 'glyphicon glyphicon-zoom-in', class: 'btn btn-info'},
          {name: 'edit-item', label: '', icon: 'glyphicon glyphicon-pencil', class: 'btn btn-warning'},
          {name: 'delete-item', label: '', icon: 'glyphicon glyphicon-remove', class: 'btn btn-danger'}
        ],
        moreParams: []
      }
    },
    methods: {
      /**
       * Callback functions
       */
      allCap: function (value) {
        return value.toUpperCase()
      },
      gender: function (value) {
        return value === 'M'
          ? '<span class="label label-info"><i class="glyphicon glyphicon-star"></i> Male</span>'
          : '<span class="label label-success"><i class="glyphicon glyphicon-heart"></i> Female</span>'
      },
      formatDate: function (value, fmt) {
        if (value === null) return ''
        fmt = (typeof fmt === 'undefined') ? 'D MMM YYYY' : fmt
        return moment(value, 'YYYY-MM-DD').format(fmt)
      },
      /**
       * Other functions
       */
      setFilter: function () {
        this.moreParams = [
          'filter=' + this.searchFor
        ]
        this.$nextTick(function () {
          this.$broadcast('vuetable:refresh')
        })
      },
      resetFilter: function () {
        this.searchFor = ''
        this.setFilter()
      },
      changePaginationComponent: function () {
        this.$broadcast('vuetable:load-success', this.$refs.vuetable.tablePagination)
      },
      preg_quote: function (str) {
        // http://kevin.vanzonneveld.net
        // +   original by: booeyOH
        // +   improved by: Ates Goral (http://magnetiq.com)
        // +   improved by: Kevin van Zonneveld (http://kevin.vanzonneveld.net)
        // +   bugfixed by: Onno Marsman
        // *     example 1: preg_quote("$40");
        // *     returns 1: '\$40'
        // *     example 2: preg_quote("*RRRING* Hello?");
        // *     returns 2: '\*RRRING\* Hello\?'
        // *     example 3: preg_quote("\\.+*?[^]$(){}=!<>|:");
        // *     returns 3: '\\\.\+\*\?\[\^\]\$\(\)\{\}\=\!\<\>\|\:'

        return (str + '').replace(/([\\\.\+\*\?\[\^\]\$\(\)\{\}=!<>\|:])/g, '\\$1')
      },
      highlight: function (needle, haystack) {
        return haystack.replace(
          new RegExp('(' + this.preg_quote(needle) + ')', 'ig'),
          '<span class="highlight">$1</span>'
        )
      },
      paginationConfig: function (componentName) {
        console.log('paginationConfig: ', componentName)
        if (componentName === 'vuetable-pagination') {
          this.$broadcast('vuetable-pagination:set-options', {
            wrapperClass: 'pagination',
            icons: {first: '', prev: '', next: '', last: ''},
            activeClass: 'active',
            linkClass: 'btn btn-default',
            pageClass: 'btn btn-default'
          })
        }
        if (componentName === 'vuetable-pagination-dropdown') {
          this.$broadcast('vuetable-pagination:set-options', {
            wrapperClass: 'form-inline',
            icons: {prev: 'glyphicon glyphicon-chevron-left', next: 'glyphicon glyphicon-chevron-right'},
            dropdownClass: 'form-control'
          })
        }
      }
    },
    events: {
      'vuetable:action': function (action, data) {
        console.log('vuetable:action', action, data)

        if (action === 'view-item') {
          sweetAlert(action, data.name)
        } else if (action === 'edit-item') {
          sweetAlert(action, data.name)
        } else if (action === 'delete-item') {
          sweetAlert(action, data.name)
        }
      },
      'vuetable:cell-dblclicked': function (item, field, event) {
        console.log('cell-dblclicked: old value =', item[field.name])
        this.$editable(event, function (value) {
          console.log('$editable callback:', value)
        })
      },
      'vuetable:load-success': function (response) {
        console.log('main.js: total = ', response.data.total)
//        var data = response.data.data
        if (this.searchFor !== '') {
//          for (n in data) {
//            data[n].name = this.highlight(this.searchFor, data[n].name)
//            data[n].email = this.highlight(this.searchFor, data[n].email)
//          }
        }
      },
      'vuetable:load-error': function (response) {
        if (response.status === 400) {
          sweetAlert('Something\'s Wrong!', response.data.message, 'error')
        } else {
          sweetAlert('Oops', E_SERVER_ERROR, 'error')
        }
      }
    }
  })
</script>
<style>
  @import "https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.6/css/bootstrap.min.css";
  @import "https://cdnjs.cloudflare.com/ajax/libs/sweetalert/1.1.3/sweetalert.min.css";

  /* Move down content because we have a fixed navbar that is 50px tall */
  ul.dropdown-menu li {
    margin-left: 20px;
  }

  .vuetable th.sortable:hover {
    color: #2185d0;
    cursor: pointer;
  }

  .vuetable-actions {
    width: 11%;
    padding: 12px 0px;
    text-align: center;
  }

  .vuetable-actions > button {
    padding: 3px 6px;
    margin-right: 4px;
  }

  .pagination {
    display: inline-flex;
  }

  .vuetable-pagination {
  }

  .vuetable-pagination-info {
    float: left;
    margin-top: auto;
    margin-bottom: auto;
  }

  .vuetable-pagination-component {
    display: inline-flex;
    float: right;
  }

  .vuetable-pagination-component li a {
    cursor: pointer;
  }

  [v-cloak] {
    display: none;
  }

  .highlight {
    background-color: yellow;
  }

  /* Loading Animation: */
  .vuetable-wrapper {
    opacity: 1;
    position: relative;
    filter: alpha(opacity=100); /* IE8 and earlier */
  }

  .vuetable-wrapper.loading {
    opacity: 0.4;
    transition: opacity .3s ease-in-out;
    -moz-transition: opacity .3s ease-in-out;
    -webkit-transition: opacity .3s ease-in-out;
  }

  .vuetable-wrapper.loading:after {
    position: absolute;
    content: '';
    top: 40%;
    left: 50%;
    margin: -30px 0 0 -30px;
    border-radius: 100%;
    -webkit-animation-fill-mode: both;
    animation-fill-mode: both;
    border: 4px solid #000;
    height: 60px;
    width: 60px;
    background: transparent !important;
    display: inline-block;
    -webkit-animation: pulse 1s 0s ease-in-out infinite;
    animation: pulse 1s 0s ease-in-out infinite;
  }

  @keyframes pulse {
    0% {
      -webkit-transform: scale(0.6);
      transform: scale(0.6);
    }
    50% {
      -webkit-transform: scale(1);
      transform: scale(1);
      border-width: 12px;
    }
    100% {
      -webkit-transform: scale(0.6);
      transform: scale(0.6);
    }
  }
</style>
