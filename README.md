# vue-bootstrap-table

vue-bootstrap-table is a sortable and searchable table, with Bootstrap styling, for Vue.js.


[**[Demo](https://jbaysolutions.github.io/vue-bootstrap-table/examples/01-basic.html) | [Changelog](/CHANGELOG.md)**]

<!--
## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Contribute](#contribute)
- [TODO List](#todo-list)

## Demos


TODO UPDATE DOCS
TODO UPDATE CHANGELOG

-->

#### Projects using vue-bootstrap-table

- [Draxed](https://www.draxed.com/?utm_source=github&utm_medium=web&utm_campaign=vue-bootstrap-table)

*Know of others? Create a PR to let me know!*

## Features

* Sortable
* Searchable
* Select display columns


## Installation

Install the vue-bootstrap-table [package](https://www.npmjs.org/package/vue-bootstrap-table) package using [npm](https://www.npmjs.com/):

	npm install vue-bootstrap-table


## Usage

npm install vue-bootstrap-table

```javascript

    new Vue({
        el: '#app',
        components: {
            VueBootstrapTable: VueBootstrapTable
        },
        data: {
            columns: [
                {
                    title:"id",
                },
                {
                    title:"name",
                },
                {
                    title:"age",
                },
                {
                    title:"country",
                }
            ],
            values: [
                {
                    "id": 1,
                    "name": "John",
                    "country": "UK",
                    "age": 25,
                },
                {
                    "id": 2,
                    "name": "Mary",
                    "country": "France",
                    "age": 30,
                },
                {
                    "id": 3,
                    "name": "Ana",
                    "country": "Portugal",
                    "age": 20,
                }
            ]
        },
    });
```` 


````html

    <vue-bootstrap-table
            :columns="columns"
            :values="values"
            :show-filter="true"
            :show-column-picker="true"
            :sortable="true"
    >
    </vue-bootstrap-table>
```` 


## Contribute

If you have a feature request, please add it as an issue or make a pull request.


## TODO List

- [x] Basic table
- [x] Sorting
- [x] Filter
- [x] Column picker
- [ ] Responsive
- [ ] Correct date sorting
- [ ] Keyboard navigation
- [ ] Editing