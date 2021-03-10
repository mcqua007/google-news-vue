<template>
  <div class="results">
    <div class="results-item" v-for="(result, index) in results" :key="index">
      <img @click="expandImg(result.urlToImage)" :src="result.urlToImage" />
      <div class="results-details">
        <strong>{{ result.title }}</strong>
        <p>{{ result.description }}</p>
        <em>by {{ result.author }}</em>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'App',
    props: ['results'],
    methods: {
      expandImg(imgUrl) {
        this.$emit('open', imgUrl);
      }
    }
  };
</script>
<style scoped>
  .results {
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 10px;
    border-top: 1px solid #eee;
    margin-top: 10px;
    padding-top: 10px;
  }

  .results-item p {
    margin: 5px 0;
  }

  .results-item {
    display: flex;
    align-items: center;
    text-align: left;
  }

  .results > .results-item:not(:first-child) {
    border-top: 1px solid #eee;
    padding-top: 10px;
  }

  .results-item img {
    width: 100%;
    max-width: 200px;
    margin-right: 10px;
    cursor: pointer;
  }

  @media (max-width: 550px) {
    .results-item {
      flex-wrap: wrap;
    }

    .results-item img {
      max-width: 100%;
    }

    .results-details {
      margin-top: 10px;
    }
  }
</style>
