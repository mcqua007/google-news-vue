<template>
  <div class="container">
    <h2>{{ title }}</h2>
    <form class="search-form" v-on:submit="submitSearchForm($event)">
      <input type="text" v-model="query" placeholder="Search news..." required />
      <button type="submit">Search</button>
    </form>
    <div class="header" v-show="searchResults">
      <em v-if="total">{{ total }} result(s)</em>
      <button class="sort-btn" @click="toggleSort()">
        <i :class="{ 'las': true, 'la-sort-alpha-down': !isAscending, 'la-sort-alpha-down-alt': isAscending }"></i>
        {{ sortDirection }}
      </button>
    </div>
    <Results @open="openImgModal" :results="searchResults"></Results>
    <div
      v-show="searchResults"
      class="pagination"
      style="width: 100%; display:flex; align-items: center; max-width: 300px; margin: 10px auto;"
    >
      <button @click="goPage(currentPage - 1)" :disabled="currentPage === 1"><i class="la la-angle-left"></i></button>
      <ul class="pagination_page-list">
        <li
          v-for="(page, index) in pagesInView"
          :key="index"
          :class="{ 'active': page === currentPage }"
          @click="goPage(page)"
        >
          {{ page }}
        </li>
      </ul>
      <button @click="goPage(currentPage + 1)" :disabled="currentPage === totalPages">
        <i class="la la-angle-right"></i>
      </button>
    </div>
    <div class="modal" @click="closeModal()" v-if="modalImg">
      <img :src="modalImg" />
    </div>
  </div>
</template>

<script>
  import Results from './Results.vue';
  export default {
    name: 'Main',
    components: {
      Results
    },
    data() {
      return {
        title: 'Google News Api',
        key: 'a3dd4d66e8794a998e7831ecb76bafaa',
        query: null,
        searchResults: null,
        total: null,
        isAscending: true,
        modalImg: null,
        currentPage: null,
        totalPages: null,
        pagesInView: [],
        maxRange: 7,
        apiMaxResults: 100,
        apiPageSize: 10,
        nytKey: 'vA1At5otQZ7AQGS3QupEWq4DZeBGBbrO',
        nytImgPrefix: 'http://static01.nyt.com',
        nytArticleURL:
          'https://api.nytimes.com/svc/search/v2/articlesearch.json?q=[QUERY]&page=[PAGE]&api-key=[API-KEY]',
        nytMovieReviewURL:
          'https://api.nytimes.com/svc/movies/v2/reviews/search.json?query=godfather&api-key=vA1At5otQZ7AQGS3QupEWq4DZeBGBbrO'
      };
    },
    computed: {
      sortDirection() {
        return this.isAscending ? 'Descending' : 'Ascending';
      }
    },
    methods: {
      goPage(page) {
        page = page > 1 ? page : 1;
        this.getSearchResults(page);
      },
      pageRange() {
        // this.maxPagesInView = this.maxPagesInView > this.totalPages ? this.totalPages : this.maxPagesInView;
        this.maxRange = this.maxRange > this.totalPages ? this.totalPages : this.maxRange;
        var num = this.currentPage;
        if (num + this.maxRange <= this.totalPages + 1) {
          this.pagesInView = [];
          for (var i = 1; i <= this.maxRange; i++) {
            this.pagesInView.push(num++);
          }
        }
      },
      closeModal() {
        this.modalImg = null;
      },
      openImgModal(url) {
        this.modalImg = url;
      },
      toggleSort() {
        this.isAscending = !this.isAscending;
        this.searchResults = this.sortArticles(this.searchResults, this.isAscending);
      },
      sortArticles(arr, sortByAscending = true) {
        if (sortByAscending) {
          return [...arr].sort((a, b) => (a.title > b.title ? 1 : -1));
        } else {
          return [...arr].sort((a, b) => (a.title < b.title ? 1 : -1));
        }
      },
      submitSearchForm(e) {
        e.preventDefault();
        this.getSearchResults(1);
      },
      getSearchResults(page = 1) {
        const url = `https://newsapi.org/v2/everything?q=${this.query}&apiKey=${this.key}&pageSize=${this.apiPageSize}&page=${page}`;
        if (this.query) {
          this.get(url).then(res => {
            console.log('api res: ', res);

            //this.totalPages = Math.round(res.totalResults / apiPageSize);  //limited to 100 results because dev version;
            this.total = res.totalResults <= 100 ? res.totalResults : 100; //because api limit
            this.totalPages = Math.round(this.total / this.apiPageSize);
            this.currentPage = page;
            let articles = this.sortArticles(res.articles, this.isAscending);
            this.searchResults = articles;
            //this.query = null; //clear query input

            this.pageRange();
          });
        } else {
          alert('Must input a query!');
        }
      },
      async get(url) {
        try {
          let res = await fetch(url);
          if (res.status === 200) {
            var response = await res.json();
            return response;
          }
        } catch (error) {
          console.log(error);
        }
      }
    },
    creadted() {}
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  ul.pagination_page-list {
    display: flex;
    align-items: center;
    grid-gap: 5px;
    margin: 0 10px;
    overflow-x: hidden;
    list-style-type: none;
    padding-inline-start: 0;
  }
  ul.pagination_page-list li {
    display: grid;
    cursor: pointer;
  }
  ul.pagination_page-list li.active {
    font-weight: 600;
  }
  .modal {
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    z-index: 2;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: #000000b3;
  }
  .modal img {
    max-width: 50vw;
  }

  @media (max-width: 768px) {
    .modal img {
      max-width: calc(100vw - 20px);
    }
  }

  .modal-close-btn {
    background: none;
    border: none;
    font-size: 25px;
    color: #fff;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-top: 10px;
  }

  .sort-btn {
    background: none;
    border: none;
    font-size: 16px;
    width: 150px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
  }
  .sort-btn i {
    font-size: 18px;
    margin-right: 7px;
  }

  .container {
    width: 100%;
    max-width: 1500px;
  }
  @media (min-width: 800px) {
    .container {
      margin: 25px auto;
    }
  }
  .search-form {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
  }
  .search-form input {
    margin-right: 10px;
    border: 1px solid #ddd;
    padding: 10px;
    height: 20px;
    width: 100%;
    max-width: 400px;
  }
  .search-form button {
    background: #2a2b2c;
    min-height: 42px;
    border: none;
    color: #fff;
    padding: 10px;
    font-weight: 600;
  }
</style>
