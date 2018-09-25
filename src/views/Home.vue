<template>
  <div class="home">
    <h1>Articles</h1>
    <div class="list">
      <div class="reading" v-for="category in categories">
        <h2>{{category.category}} News</h2>
        <h4>{{category.data[category.currentArticleIndex].webTitle}}</h4>
        <img v-if="category.data[category.currentArticleIndex].imageUrl" v-bind:src="category.data[category.currentArticleIndex].imageUrl" alt="">
        <img v-if="!category.data[category.currentArticleIndex].imageUrl" src="http://thehill.us/wp-content/uploads/2017/05/wait.jpg">
        <div class="buttonNavs">
          <div class="kids">
            <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored" v-on:click="downOne(category)">Back</button>
          </div>
          <div class="kids">
            <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--accent" id="show-dialog" v-on:click="read2(category)">READ ARTICLE</button>
          </div>
          <div class="kids">
            <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored" v-on:click="upOne(category)">Next</button>
          </div>
        </div>
        <div class="reading" v-if="category.currentArticleVisible">
          <p v-html="info.response.content.blocks.body[0].bodyHtml"></p>
          <div class="collapse">
            <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--accent" id="show-dialog" v-on:click="collapse(category)">COLLAPSE ARTICLE</button>
          </div>
        </div>
      </div>  
    </div>
  </div>
</template>

<style>
h2 {
  text-transform: uppercase;
}

.list,
.reading {
  border: 1px solid grey;
  text-align: center;
}

.reading img {
  max-height: 300px;
  max-width: 500px;
}

.reading p {
  text-align: left;
  font-size: 20px;
  padding: 10px;
}

.buttonNavs {
  display: flex;
}

.kids {
  margin: auto;
}
.collapse {
  text-align: center;
}
</style>


<script>
var axios = require("axios");
export default {
  data: function() {
    return {
      categories: [],
      info: []
      // can't affect all other categories....
    };
  },
  // make request to categories view to retrieve first and second item in category JSON data
  created: function() {
    axios.get("http://localhost:3000/api/articles").then(
      function(response) {
        console.log("categories");
        console.log(response);
        this.categories = response.data;
        this.categories.forEach(category => {
          console.log("getArticleImage for", category.data[0]);
          this.getArticleImage(category.data[0]);
        });
        // this.apiUrls = response.data.apiUrl;
        // console.log("HERE ARE YOUR PICS");
        // console.log(this);
      }.bind(this)
    );
  },

  methods: {
    upOne: function(category) {
      if (category.currentArticleIndex < category.data.length) {
        category.currentArticleIndex += 1;
        this.getArticleImage(category.data[category.currentArticleIndex]);
      }
      // console.log(category.currentArticleIndex);
    },
    downOne: function(category) {
      if (category.currentArticleIndex > 0) {
        category.currentArticleIndex -= 1;
      }
      // console.log(category.currentArticleIndex);
    },
    read2: function(category) {
      var link = category.data[category.currentArticleIndex].apiUrl;
      axios
        .get("http://localhost:3000/api/read/", {
          params: { api_URL: link}
        })
        .then(function(response) {
          this.info = response.data.info;
        }.bind(this)
        );
      console.log("CURRENT STATUS:");
      console.log(category.currentArticleVisible);
      this.categories.forEach(cat => (cat.currentArticleVisible = false));
      category.currentArticleVisible = !category.currentArticleVisible;
      console.log("NEW STATUS:");
      console.log(category.currentArticleVisible);
      console.log(this);

    },

    getArticleImage: function(article) {
      if (!article.imageUrl) {
        console.log("get the image using ", article.apiUrl);
        var params = {
          api_URL: article.apiUrl
        };
        // console.log("LOOK AT ME!!!");
        // console.log(params);
        axios
          .get("http://localhost:3000/api/pic/", {
            params: { api_URL: article.apiUrl }
          })
          .then(function(response) {
            article.imageUrl = response.data.img;
          });
      }
    },
    collapse: function(category) {
      category.currentArticleVisible = !category.currentArticleVisible;
      console.log("RIDIN DIRTY");
    }
  },
  computed: {}
};
</script>
