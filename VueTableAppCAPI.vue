<script>
import { ref, reactive, computed, onMounted } from 'vue';
import axios from 'axios';
import moment from 'moment';

export default {
  setup() {
    const data = reactive({
      requests: [],
      isSearchable: true,
      isSortable: true,
      currentSort: 'date',
      currentSortDir: 'desc',
      pageSize: 4,
      currentPage: 1,
      filter: '',
      filterResultsLength: null,
      calcHeight: 0,
      errored: false,
    });

    const totalRows = ref(0); // Create totalRows as a reactive ref

    const formatDate = (value) => {
      return moment(value).format('DD-MM-YYYY');
    };

    const getDataFromAPI = (url) => {
      axios
          .get(url)
          .then((response) => {
            data.requests = response.data;
            // data.totalRows = response.headers['x-total-count'];
            totalRows.value = response.headers['x-total-count']; // Update totalRows
            /*nextTick(() => {
              setMinHeight();
            });*/
            // setMinHeight();
          })
          .catch((error) => {
            console.log(error);
            data.errored = true;
          });
    };

    const sort = (s) => {
      if (s === data.currentSort) {
        data.currentSortDir = data.currentSortDir === 'asc' ? 'desc' : 'asc';
      }
      data.currentSort = s;
      data.currentPage = 1;

      if (data.filter !== '') {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=1&_limit=${data.pageSize}&q=${data.filter}`;
        getDataFromAPI(url);
      } else {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=1&_limit=${data.pageSize}`;
        getDataFromAPI(url);
      }
    };

    const prevPage = () => {
      if (data.currentPage > 1) data.currentPage--;

      if (data.currentPage !== 1) {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=${data.currentPage}&_limit=${data.pageSize}`;
        getDataFromAPI(url);
      }
    };

    const nextPage = () => {
      if (data.currentPage * data.pageSize < totalRows.value) data.currentPage++;

      if (data.currentPage <= numPages.value) {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=${data.currentPage}&_limit=${data.pageSize}`;
        getDataFromAPI(url);
      }
    };

    const gotoPage = (pageNum) => {
      data.currentPage = pageNum;
      const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=${data.currentPage}&_limit=${data.pageSize}&q=${data.filter}`;
      getDataFromAPI(url);
    };

    const setMinHeight = () => {
      let tableHeight = $refs.vuetable.clientHeight;
      data.calcHeight = tableHeight + 2;
    };

    const filteredResults = () => {
      if (data.filter !== '') {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=1&_limit=${data.pageSize}&q=${data.filter}`;
        getDataFromAPI(url);
        data.currentPage = 1;
      } else {
        const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=1&_limit=${data.pageSize}`;
        getDataFromAPI(url);
        data.currentPage = 1;
      }
    };

    const numResults = computed(() => {
      return data.requests.length;
    });

    const numPages = computed(() => {
      // return Math.ceil(data.totalRows / data.pageSize);
      return Math.ceil(totalRows.value / data.pageSize);
    });

    onMounted(() => {
      console.log('mounted');
      const url = `https://my-json-server.typicode.com/studio-henk/sh-apis/requests?_sort=${data.currentSort}&_order=${data.currentSortDir}&_page=1&_limit=${data.pageSize}`;
      axios
          .get(url)
          .then((response) => {
            data.requests = response.data;
            totalRows.value = response.headers['x-total-count']; // Update totalRows
          })
          .catch((error) => {
            console.log(error);
            data.errored = true;
          });
    });

    return {
      data,
      formatDate,
      sort,
      prevPage,
      nextPage,
      gotoPage,
      filteredResults,
      numResults,
      numPages,
      totalRows
    };
  },
};
</script>

<template>
  <div class="table-container-2">
    <div class="org-vue-datatable" id="table2" v-cloak>
      <label for="search-field">Search</label>
      <input class="table-search-input" v-if="data.isSearchable" type="search" placeholder="search in table rows"
             v-model="data.filter" @input="filteredResults()" id="search-field">
      <p v-if="data.isSearchable && totalRows > 1 || totalRows == 0">{{totalRows}} results</p>
      <p v-if="data.isSearchable && totalRows == 1">{{totalRows}} result</p>
      <div class="demo vue-datatable" id="table-vue-requests2" ref="vuetable">
        <table class="--striped --responsive xvue-datatable">
          <thead>
          <tr v-if="data.isSortable">
            <th class="--sort" @click="sort('date')" width="20%"
                :class="{ '--sorting-active': data.currentSort == 'date', '--sort-asc': data.currentSortDir == 'asc' }">
              Date
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                   fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                   stroke-linejoin="round" class="feather feather-code">
                <polyline points="16 18 22 12 16 6" id="sort-desc"></polyline>
                <polyline points="8 6 2 12 8 18" id="sort-asc"></polyline>
              </svg>
            </th>
            <th class="--sort" @click="sort('id')" width="15%"
                :class="{ '--sorting-active': data.currentSort == 'id', '--sort-asc': data.currentSortDir == 'asc' }">
              Request ID
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                   fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                   stroke-linejoin="round" class="feather feather-code">
                <polyline points="16 18 22 12 16 6" id="sort-desc"></polyline>
                <polyline points="8 6 2 12 8 18" id="sort-asc"></polyline>
              </svg>
            </th>
            <th class="--sort" @click="sort('comment')" width="55%"
                :class="{ '--sorting-active': data.currentSort == 'comment', '--sort-asc': data.currentSortDir == 'asc' }">
              Comment
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                   fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                   stroke-linejoin="round" class="feather feather-code">
                <polyline points="16 18 22 12 16 6" id="sort-desc"></polyline>
                <polyline points="8 6 2 12 8 18" id="sort-asc"></polyline>
              </svg>
            </th>
            <th class="--sort" @click="sort('status')" width="10%"
                :class="{ '--sorting-active': data.currentSort == 'status', '--sort-asc': data.currentSortDir == 'asc' }">
              Status
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
                   fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                   stroke-linejoin="round" class="feather feather-code">
                <polyline points="16 18 22 12 16 6" id="sort-desc"></polyline>
                <polyline points="8 6 2 12 8 18" id="sort-asc"></polyline>
              </svg>
            </th>
          </tr>
          <tr v-if="!data.isSortable"><th width="20%">Date</th><th width="15%">Request ID</th><th width="55%">Comment</th><th width="10%">Status</th></tr></thead>
          <tbody name="fade" is="transition-group" appear>
          <tr v-for="request in data.requests" :key="request.id" :class="{ '--status-open' : request.status == 'open', '--status-closed' : request.status == 'closed', '--status-rejected' : request.status == 'rejected' }">
            <td data-heading="Date">{{formatDate(request.date)}}</td>
            <td data-heading="Request ID">{{request.id}}</td>
            <td data-heading="Comment">{{request.comment}}</td>
            <td data-heading="Status">{{request.status}}</td>
          </tr>
          </tbody>
        </table>
      </div>
      <nav class="molecule-pagination" v-show="numPages > 1">
            <span class="molecule-pagination__link molecule-pagination__link--prev --state-disabled" v-if="data.currentPage == 1">
                <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <use xlink:href="/images/svg/feather-sprite.svg#chevrons-left"></use>
                </svg>
            </span>
        <a class="molecule-pagination__link molecule-pagination__link--prev" href="#table2" @click="gotoPage(1)" v-else>
          <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <use xlink:href="/images/svg/feather-sprite.svg#chevrons-left"></use>
          </svg>
        </a>
        <span class="molecule-pagination__link molecule-pagination__link--prev --state-disabled" v-if="data.currentPage == 1">
                <svg class="iconx" width="24" height="24" fill="currentColor">
                    <use xlink:href="/images/svg/sh-svgsprite.svg#chevron_left-24px"></use>
                </svg>
            </span>
        <a class="molecule-pagination__link molecule-pagination__link--prev" href="#table2" @click="gotoPage(data.currentPage-1)" v-else>
          <svg class="icon" width="24" height="24" fill="currentColor">
            <use xlink:href="/images/svg/sh-svgsprite.svg#chevron_left-24px"></use>
          </svg>
        </a>
        <a class="molecule-pagination__link molecule-pagination__page" href="#table2" v-show="data.currentPage == numPages && numPages > 4" @click="gotoPage(data.currentPage-4)">{{data.currentPage-4}}</a><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage == numPages && numPages >= 4 || data.currentPage == numPages - 1 && numPages > 4" @click.prevent="gotoPage(data.currentPage-3)">{{data.currentPage-3}}</a><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage > 2 && numPages > 2" @click.prevent="gotoPage(data.currentPage-2)">{{data.currentPage-2}}</a><!-- left of current page -1. only if not currentPage itself and numPages > 1--><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage > 1 && numPages > 1" @click.prevent="gotoPage(data.currentPage-1)">{{data.currentPage-1}}</a><!-- current page button--><a class="molecule-pagination__currentpage" href="#">{{data.currentPage}}</a><!-- right of current page +1. only if not currentPage itself and currentPage is < numPages--><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage > 0 && data.currentPage < numPages" @click.prevent="gotoPage(data.currentPage+1)">{{data.currentPage+1}}</a><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage > 0 && data.currentPage < numPages - 1" @click.prevent="gotoPage(data.currentPage+2)">{{data.currentPage+2}}</a><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage < 3 && data.currentPage + 3 <= numPages" @click.prevent="gotoPage(data.currentPage+3)">{{data.currentPage+3}}</a><a class="molecule-pagination__link molecule-pagination__page" href="#" v-show="data.currentPage == 1 && numPages > 4" @click.prevent="gotoPage(data.currentPage+4)">{{data.currentPage+4}}</a>
        <span class="molecule-pagination__link molecule-pagination__link--next --state-disabled" v-if="data.currentPage == numPages">
                <svg class="icon" width="24" height="24" fill="currentColor">
                    <use xlink:href="/images/svg/sh-svgsprite.svg#chevron_right-24px"></use>
                </svg>
            </span>
        <a class="molecule-pagination__link molecule-pagination__link--next" href="#table2" @click="gotoPage(data.currentPage+1)" v-else>
          <svg class="icon" fill="currentColor" width="24" height="24">
            <use xlink:href="/images/svg/sh-svgsprite.svg#chevron_right-24px"></use>
          </svg>
        </a>
        <!-- last page button--><!-- only show when there are 5 or more pages-->
        <span class="molecule-pagination__link molecule-pagination__link--next --state-disabled" v-if="data.currentPage == numPages">
                <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <use xlink:href="/images/svg/feather-sprite.svg#chevrons-right"></use>
                </svg>
            </span>
        <a class="molecule-pagination__link molecule-pagination__link--next" href="#table2" @click="gotoPage(numPages)" v-else>
          <svg width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <use xlink:href="/images/svg/feather-sprite.svg#chevrons-right"></use>
          </svg>
        </a>
      </nav>
      <section v-show="data.errored">
        <p>We're sorry, we're not able to retrieve this information at the moment, please try back later</p>
      </section>
    </div>
  </div>
</template>
<style scoped>
html {
  scroll-behavior: smooth;
}

.table-container-2 {
  max-width: 60em;
  margin: 0 auto;
  font-size: 1rem;
}

.table-container-2 > h1 {
  border-bottom: 1px solid #eeeeee;
  line-height: 54px;
  margin-bottom: 2rem;
}

/* nprogress */
/* Make clicks pass-through */
#nprogress {
  pointer-events: none;
}

#nprogress .bar {
  background: red;
  position: fixed;
  z-index: 1031;
  top: 0;
  left: 0;
  width: 100%;
  height: 2px;
}

/* Fancy blur effect */
#nprogress .peg {
  display: block;
  position: absolute;
  right: 0;
  width: 100px;
  height: 100%;
  box-shadow: 0 0 10px #29d, 0 0 5px #29d;
  opacity: 1.0;
  transform: rotate(3deg) translate(0px, -4px);
}

/* Remove these to get rid of the spinner */
#nprogress .spinner {
  display: block;
  position: fixed;
  z-index: 1031;
  top: 25%;
  left: 50%;
}

#nprogress .spinner-icon {
  width: 18px;
  height: 18px;
  box-sizing: border-box;

  border: solid 2px transparent;
  border-top-color: red;
  border-left-color: green;
  border-radius: 50%;

  -webkit-animation: nprogress-spinner 400ms linear infinite;
  animation: nprogress-spinner 400ms linear infinite;
}

.nprogress-custom-parent {
  overflow: hidden;
  position: relative;
}

.nprogress-custom-parent #nprogress .spinner,
.nprogress-custom-parent #nprogress .bar {
  position: absolute;
}

@keyframes nprogress-spinner {
  0%   { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

input[type=text], input[type=email], input[type=tel], input[type=password], input[type=search] {
  width: 100%;
  padding: 14px 8px;
  line-height: 24px;
  border: 1px solid #000;
  outline-color: #b6a58d;
}

.table-search-input[type=search] {
  border-color: #d3d3d3;
  border-radius: 0;
  appearance: textfield;
  -webkit-appearance: none;
}

[type=search]::-webkit-search-cancel-button, [type=search]::-webkit-search-decoration {
  -webkit-appearance: none;
}

::placeholder {
  color: #d3d3d3 !important;
}

::-webkit-input-placeholder {
  color: #d3d3d3 !important;
}

label, input {
  box-sizing: border-box;
  display: block;
}

table {
  width: 100%;
  margin: 0;
  border-collapse: collapse;
  box-sizing: border-box;
  background-color: #fff;
}

table th {
  text-align: left;
  font-weight: 400;
  border-bottom: 1px solid #d3d3d3;
  white-space: nowrap;
  vertical-align: middle;
  line-height: 1;
  padding: 30px 8px 10px 8px;
}

table td {
  vertical-align: top;
  line-height: 1.6;
  margin: 0;
  padding: 14px 8px 15px 8px;
}

td[data-heading=Date],
td[data-heading=Status] {
  white-space: nowrap;
}

/* hide col on mobile */
@media only screen and (max-width: 767px) {
  /*thead th:first-child,
  td[data-heading=Date] {
    display: none;
  }*/

  thead {
    display: none;
  }

  tbody td {
    display: block;
  }
}

.org-vue-datatable {
  margin-bottom: 5rem;
}

.org-vue-datatable .vue-datatable {
  border: 1px solid hsla(0deg, 0%, 92%, 1);
  color: black;
}

html.dark .vue-datatable  {
  border-color: rgba(255, 255, 255, 0.51);
}

.org-vue-datatable tbody {
  margin: 0;
}

.org-vue-datatable[v-cloak] {
  display: none;
}

.vue-datatable th.--sort, .vue-datatable tbody tr {
  cursor: pointer;
}

.vue-datatable tbody tr {
  transition: all .3s ease;
  box-shadow: 0 0 0 black;
}

.vue-datatable tbody tr:hover {
  transform: perspective(50px) translate3d(0 , 0, .15rem);
  box-shadow: 0 0 4px #ccc;
}

.--striped tbody tr:nth-child(odd) {
  background-color: #fff;
}

.--striped tbody tr:nth-child(even) {
  background-color: #f8f8f8;
}

table.--striped tr+tr td, table.--rows-table tr+tr td {
  border-color: transparent;
}
table tr+tr td {
  border-top: 1px solid #d3d3d3;
}

.--sort.--sorting-active {
  border-color: #b6a58d;
  border-width: 2px;
}
.--sort {
  position: relative;
}

.--sort svg {
  width: 18px;
  height: 18px;
  transform: rotate(90deg);
  position: absolute;
  right: 8px;
  top: 28px;
  color: #d3d3d3;
}

.--sort.--sorting-active #sort-desc {
  color: #b6a58d;
}

.--sort.--sort-asc #sort-desc {
  color: #d3d3d3;
}

.--sort.--sorting-active.--sort-asc #sort-asc {
  color: #b6a58d;
}

.--status-closed [data-heading="Status"] {
  color: green;
}

.--status-rejected {
  color: #ccc;
}

.--status-rejected [data-heading="Status"] {
  color: red;
  text-decoration: line-through;
}

.molecule-pagination {
  display: -webkit-flex;
  display: flex;
  -webkit-justify-content: center;
  justify-content: center;
  -webkit-align-items: center;
  align-items: center;
  margin: 2rem auto;
  transition: all .3s ease-in-out;
}

.molecule-pagination__link {
  margin: 0;
  -webkit-flex: 0 0 40px;
  flex: 0 0 40px;
  padding: 8px 0;
  text-align: center;
  text-decoration: none;
  transition: all .3s ease-in-out;
  line-height: 24px;
  font-family: monospace;
    color: currentColor;
}

.molecule-pagination__link--prev, .molecule-pagination__link--next {
  background-color: #f8f8f8;
  border-radius: 50%;
}

.molecule-pagination .--state-disabled {
  cursor: not-allowed;
  opacity: .25;
  color: #111;
}

.molecule-pagination__link>svg {
  display: block;
  margin: 0 auto;
}

.molecule-pagination__page {
  border-radius: 50%;
}

@media only screen and (max-width: 767px) {
  .vue-datatable {
    min-height: auto !important;
  }

  .molecule-pagination {
    position: sticky;
    bottom: 0;
    background: lightgrey;
    margin: 0 -1rem;
    padding: 1rem;
    justify-content: space-between;
  }

  .molecule-pagination__page {
    display: none;
  }
}

.molecule-pagination__currentpage {
  background: #000;
  color: #fff;
  border-radius: 100%;
  margin: 0;
  flex: 0 0 40px;
  text-align: center;
  padding: 8px 0;
  text-decoration: none;
  cursor: default;
  font-family: monospace;
  line-height: 24px;
}
</style>