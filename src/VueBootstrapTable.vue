<template>
    <div @click="closeDropdown" @keyup.esc="closeDropdown">
        <!--<pre>{{columns | json}}</pre>-->
        <!--<pre>{{$data | json}}</pre>-->
        <div class="col-sm-6">
            <div v-if="showFilter" style="padding-top: 10px;padding-bottom: 10px;">
                <div class="input-group">
                    <input type="text" class="form-control" placeholder="Filter" v-model="filterKey">
                    <div class="input-group-addon">
                        <i class="glyphicon glyphicon-search"></i>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-sm-6">
            <div v-if="showColumnPicker" style="padding-top: 10px;padding-bottom: 10px;float:right;">
                <div class="btn-group" :class="{'open' : columnMenuOpen}">
                    <button @click.stop.prevent="columnMenuOpen = !columnMenuOpen" @keyup.esc="columnMenuOpen = false"
                            type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown"
                            aria-haspopup="true">
                        Columns <span class="caret"></span>
                    </button>
                    <ul class="dropdown-menu">
                        <li v-for="column in displayCols">
                            <a href="#" @click.stop.prevent="toggleColumn(column)">
                                <i v-if="column.visible" class="glyphicon glyphicon-ok"></i> {{column.title}}
                            </a>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="col-sm-12">
            <table class="table table-bordered table-hover table-condensed table-striped vue-table">
                <thead>
                    <tr>
                        <th v-for="column in displayCols | filterBy true in 'visible'" @click="sortBy(column.title)"
                            track-by="$index"
                            :class="getClasses(column.title)">
                            {{ column.title }}
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="entry in filteredValues | orderBy sortKey sortOrders[sortKey]" track-by="$index">
                        <td v-for="column in displayCols | filterBy true in 'visible'" track-by="$index"
                            v-show="column.visible">
                            <span v-if="!column.editable"> {{ entry[column.title] }} </span>
                            <value-field-section v-else
                                v-bind:entry="entry"
                                :columntitle="column.title"
                                :value="entry[column.title]"><value-field-section>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div v-if="paginated" class="col-sm-12">
            <div class="btn-toolbar" role="toolbar" aria-label="pagination bar">
              <div class="btn-group" role="group" aria-label="first page">
                <button type="button" class="btn btn-default" @click="this.page=1">&laquo;</button>
              </div>
              <div class="btn-group" role="group" aria-label="pages">
                <button v-for="index in validPageNumbers"
                    type="button" class="btn btn-default"
                    v-bind:class="{ active: this.page===index }"
                    @click="this.page=index">
                        {{index}}
                </button>
              </div>
              <div class="btn-group" v-if="showPaginationEtc">...</div>
              <div class="btn-group" role="group" aria-label="last page">
                <button type="button" class="btn btn-default" @click="this.page=this.maxPage">&raquo;</button>
              </div>
            </div>
        </div>
    </div>
</template>
<style>
    .vue-table {

    }

    table.vue-table thead > tr > th {
        cursor: pointer;
        padding-right: 30px !important;
    }

    /*.vue-table th.active {
        color: red;
    }*/

    .vue-table .arrow {
        opacity: 1;
        position: relative;
    }

    .vue-table .arrow:after {
        position: absolute;
        bottom: 8px;
        right: 8px;
        display: block;
        font-family: 'Glyphicons Halflings';
        content: "\e150";
        /*
        display: inline-block;
        vertical-align: middle;
        width: 0;
        height: 0;
        margin-left: 5px;
        opacity: 0.66;*/
    }

    .vue-table .arrow.asc:after {
        content: "\e155";
        /*
        border-left: 4px solid transparent;
        border-right: 4px solid transparent;
        border-bottom: 4px solid #000;
        */
    }

    .vue-table .arrow.dsc:after {
        content: "\e156";
    }


    .vue-table .editableField {
        cursor:pointer;
    }

    /*.vue-table .selected-cell {
        background-color: #F7C072;
    }

    .vue-table .selected-row {
        background-color: #FAE1BE !important;
    }*/
</style>
<script>

    /* Field Section used for displaying and editing value of cell */
    var valueFieldSection = {
      template: '<span v-if="!enabled" @dblclick="toggleInput" class="editableField"> {{ value }} </span>'+
          '<div v-if="enabled" class="input-group">'+
          '  <input type="text" class="form-control" v-model="value" @keyup.enter="saveThis" @keyup.esc="cancelThis">'+
          '  <span class="input-group-btn">'+
          '    <button class="btn btn-danger" type="button" @click="cancelThis" ><span class="glyphicon glyphicon-remove" aria-hidden="true"></span></button>'+
          '    <button class="btn btn-primary" type="button" @click="saveThis" ><span class="glyphicon glyphicon-ok" aria-hidden="true"></span></button>'+
          '  </span>'+
          '</div>',
      props: ['entry','value','columntitle'],
      data: function () {
          return {
            enabled: false,
          }
      },
      methods: {
        saveThis: function () {
            var originalValue = this.entry[this.columntitle];
            this.entry[this.columntitle] = this.value;
            this.$dispatch('cellDataModifiedEvent', originalValue, this.value, this.columntitle,  this.entry);
            this.enabled = !this.enabled;
        },
        cancelThis: function () {
            this.value = this.entry[this.columntitle];
            this.enabled = !this.enabled;
        },
        toggleInput: function () {
            this.enabled=!this.enabled
        },
      }
    };

    export default {
        name: "VueBootstrapTable",
        components: {
            'value-field-section': valueFieldSection,
        },
        props: {
            /**
             * The column titles, required
             */
            columns: {
                type: Array,
                required: true,
            },
            /**
             * The rows, an Array of objects
             */
            values: {
                type: Array,
                required: true,
            },
            /**
             * Enable/disable table sorting, optional, default true
             */
            sortable: {
                type: Boolean,
                required: false,
                default: true,
            },
            /**
             * Enable/disable input filter, optional, default false
             */
            showFilter: {
                type: Boolean,
                required: false,
                default: false,
            },
            /**
             * Enable/disable column picker to show/hide table columns, optional, default false
             */
            showColumnPicker: {
                type: Boolean,
                required: false,
                default: false,
            },
            /**
             * Enable/disable pagination for the table, optional, default false
             */
            paginated: {
                type: Boolean,
                required: false,
                default: false,
            },
            /**
             * If pagination is enabled defining the page size, optional, default 10
             */
            pageSize: {
                type: Number,
                required: false,
                default: 10,
            },

        },
        data: function () {
            return {
                filteredSize: 0,
                filterKey: "",
                sortKey: "",
                sortOrders: {},
                columnMenuOpen: false,
                displayCols: [],
                page: 1,
            };
        },
        ready: function () {
            this.setSortOrders();
            var self = this;
            this.columns.forEach(function (column) {
                var obj = {};
                obj.title = column.title;
                if ( typeof column.visible !== "undefined")
                    obj.visible = column.visible;
                else
                    obj.visible = true;
                if ( typeof column.editable !== "undefined")
                    obj.editable = column.editable;
                else
                    obj.editable = false;
                self.displayCols.push(obj);
            });
        },
        watch: {
            values: function () {
            },
            columns: function () {
                this.displayCols = [];
                var self = this;
                this.columns.forEach(function (column) {
                    var obj = {};
                    obj.title = column.title;
                    obj.visible = true;
                    self.displayCols.push(obj);
                });
                this.setSortOrders();
            },
            showFilter: function () {
                this.filterKey = "";
            },
            showColumnPicker: function () {
                this.columnMenuOpen = false;

                this.displayCols.forEach(function (column) {
                    column.visible = true;
                });
            },
            filterKey: function () {
                // filter was updated, so resetting to page 1
                this.page = 1;
            },
        },
        computed: {
            filteredValues: function () {
                var result = this.$options.filters.filterBy(this.values, this.filterKey);
                result = this.$options.filters.orderBy(result, this.sortKey, this.sortOrders[this.sortKey]);
                this.filteredSize = result.length;
                if(this.paginated) {
                    var startIndex = (this.page-1)*this.pageSize;
                    var tIndex = 0;
                    var tempResult = [];
                    while (tIndex < this.pageSize){
                        if ( typeof result[startIndex+tIndex] !== "undefined")
                            tempResult.push( result[startIndex+tIndex]);
                        tIndex++;
                    }
                    return tempResult;
                }
                return result;
            },
            validPageNumbers: function () {
                // 5 page max
                var result = [];
                var start = 1;
                if (this.page > 3)
                    start = this.page-2;
                for ( var i = 0 ; start <= this.maxPage && i<5; start++ ) {
                    result.push(start);
                    i++;
                }
                return result;
            },
            maxPage: function () {
                return Math.ceil(this.filteredSize / this.pageSize);
            },
            showPaginationEtc: function () {
                var temp = 1;
                if (this.page > 3)
                    temp = this.page-2;
                return ( (temp+4) < this.maxPage  );
            },
        },
        methods: {
            setSortOrders: function () {
                this.sortKey = "";
                var sortOrders = {};
                this.columns.forEach(function (column) {
                    sortOrders[column.title] = 0;
                });
                this.sortOrders = sortOrders;

            },
            sortBy: function (key) {
                if (this.sortable) {
                    var self = this;
                    this.sortKey = key;
                    this.columns.forEach(function (column) {
                        if (column.title !== key) {
                            self.sortOrders[column.title] = 0;
                        }
                    });
                    if (this.sortOrders[key] === 0) {
                        this.sortOrders[key] = 1;
                    } else {
                        this.sortOrders[key] = this.sortOrders[key] * -1;
                    }
                }
                //console.log(JSON.stringify(this.sortOrders));
            },
            getClasses: function (key) {
                var classes = [];
                if (this.sortable) {
                    classes.push("arrow");
                    if (this.sortKey === key) {
                        classes.push("active");
                    }
                    if (this.sortOrders[key] === 1) {
                        classes.push("asc");
                    } else if (this.sortOrders[key] === -1) {
                        classes.push("dsc");
                    }
                }
                return classes;
            },
            toggleColumn: function (column) {
                column.visible = !column.visible;
            },
            closeDropdown: function () {
                this.columnMenuOpen = false;
            },
        },
        events: {
        }
    }
</script>