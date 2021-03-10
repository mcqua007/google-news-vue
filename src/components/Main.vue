<template>
  <div class="container">
    <h2>{{ title }}</h2>
    <form class="search-form" v-on:submit="getSearchResults($event)">
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
        modalImg: null
      };
    },
    computed: {
      sortDirection() {
        return this.isAscending ? 'Descending' : 'Ascending';
      }
    },
    methods: {
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
      getSearchResults(e) {
        e.preventDefault();
        const url = `https://newsapi.org/v2/everything?q=${this.query}&apiKey=${this.key}`;
        if (this.query) {
          this.get(url).then(res => {
            this.query = null; //clear query input
            console.log('before: ', res.articles);

            var articles = this.sortArticles(res.articles, this.isAscending);

            this.searchResults = articles;
            this.total = res.totalResults;
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
    max-width: calc(100vw - 20px);
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
