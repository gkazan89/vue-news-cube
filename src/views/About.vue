<template>
  <div class="home">
    <h1>Articles</h1>
    <div>
      <!-- Put cube right here -->
      <div class="bigbox">
        <!-- fuse the cube with the article text -->
        <div class="radio-group" v-on:click="changeSide">
          <div v-for="categoryWithSide in categoriesWithSides" class="radio-group-item">
            <label>
              <input type="radio" name="rotate-cube-side" v-bind:value="categoryWithSide.side"/> {{categoryWithSide.category.category}}
            </label>
          </div>
        </div>  
        <div class="scene">
          <div class="cube">
            <div v-for="categoryWithSide in categoriesWithSides">
              <div v-bind:class="'cube__face cube__face--'+categoryWithSide.side">
                <!-- category name i.e. "sports" -->
                <h2 class="words">{{categoryWithSide.category.category}}</h2>
                <!-- headline -->
                <h2 class="words" id="headline">{{ categoryWithSide.category.data[categoryWithSide.category.currentArticleIndex].webTitle }}</h2>
                <div class="articleContainer" v-if="categoryWithSide.category.currentArticleVisible">
                  <!-- article text -->
                  <div class="content">
                    <p v-html="info.response.content.blocks.body[0].bodyHtml">
                    </p>
                    <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--accent" id="show-dialog" v-on:click="read2(categoryWithSide.category)">COLLAPSE ARTICLE</button> 
                  </div>
                </div>
                <div class="buttons">
                  <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored" v-on:click="upOne(categoryWithSide.category)">NEXT</button>
                  <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--accent" v-on:click="downOne(categoryWithSide.category)">BACK</button>
                  <button class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored" id="show-dialog" v-on:click="read2(categoryWithSide.category)">READ ARTICLE</button>
                </div>
              </div>
            </div>
          </div>
        </div> 
      </div>  
    </div>
    <a href="https://www.theguardian.com/us" target="_blank"><img src="https://static.guim.co.uk/sys-images/Guardian/Pix/pictures/2010/03/01/poweredbyguardianBLACK.png"></a>
  </div>
</template>

<style>
  .article_img img {
    max-width: 500px;
    max-height: 300px;
  }

  .bigbox {
    box-sizing: border-box;
  }

  .scene {
    width: 800px;
    height: 800px;
    perspective: 400px;
  }

  .cube {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    transform: translateZ(-400px);
    transition: transform 1s;
  }

  /* show class rotates the cube */
  .cube.show-front {
    transform: translateZ(-400px) rotateY(0deg);
  }
  .cube.show-right {
    transform: translateZ(-400px) rotateY(-90deg);
  }
  .cube.show-back {
    transform: translateZ(-400px) rotateY(-180deg);
  }
  .cube.show-left {
    transform: translateZ(-400px) rotateY(90deg);
  }
  .cube.show-top {
    transform: translateZ(-400px) rotateX(-90deg);
  }
  .cube.show-bottom {
    transform: translateZ(-400px) rotateX(90deg);
  }

  .cube__face {
    position: absolute;
    width: 800px;
    height: 800px;
    border: 2px solid black;
    line-height: 500px;
    font-size: 100px;
    font-weight: bold;
    color: white;
    text-align: center;
  }

  .cube__face--front {
    background: hsla(0, 100%, 50%, 0.7);
  }
  .cube__face--right {
    background: hsla(60, 100%, 50%, 0.7);
  }
  .cube__face--back {
    background: hsla(120, 100%, 50%, 0.7);
  }
  .cube__face--left {
    background: hsla(180, 100%, 50%, 0.7);
  }
  .cube__face--top {
    background: hsla(240, 100%, 50%, 0.7);
  }
  .cube__face--bottom {
    background: hsla(300, 100%, 50%, 0.7);
  }

  .cube__face--front {
    transform: rotateY(0deg) translateZ(400px);
  }
  .cube__face--right {
    transform: rotateY(90deg) translateZ(400px);
  }
  .cube__face--back {
    transform: rotateY(180deg) translateZ(400px);
  }
  .cube__face--left {
    transform: rotateY(-90deg) translateZ(400px);
  }
  .cube__face--top {
    transform: rotateX(90deg) translateZ(400px);
  }
  .cube__face--bottom {
    transform: rotateX(-90deg) translateZ(400px);
  }

  .content {
    display: inline-block;
    height: 500px;
    overflow-y: scroll;
    border: solid 2px grey;
  }

  .content img {
    max-height: 300px;
    max-width: 500px;
    display: block;
    margin: auto;

  }

  .content p {
    color: black;
    background-color: white;
    font-size: 20px;
    text-align: left;
    padding-left: 10px;
    padding-right: 10px;
  }

  .radio-group {
    display: flex;
    margin-right: 10px;
    margin-bottom: 30px;
  }

  .radio-group-item {
    padding: 5px;
  }
  .radio-group label {
    padding-bottom: 10px;
    padding-top: 10px;
  }

  label {
    margin: auto;
    font-size: 30px;
    color: black;
    text-transform: uppercase;
  }

  .collapse {
    text-align: center;
  }

  .words {
    margin-top: 5px;
    font-size: 30px;
    color: black;
    background-color: rgba(255, 255, 255, 0.7);
  }

  #headline {
    background-color: rgba(255, 255, 255, 0.7);
  }
</style>


<script>
// set initial side
// changeSide();

// radioGroup.addEventListener("change", changeSide);
// end of all the cube js

var axios = require("axios");
export default {
  data: function() {
    return {
      categories: [],
      info: [],
      cube: null,
      radioGroup: null,
      currentClass: "",
      categoriesWithSides: []

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

        var sides = ["front", "right", "back", "left", "top", "bottom"];
        var index = 0;
        this.categories.forEach(category => {
          this.categoriesWithSides.push({
            category: category,
            side: sides[index]
          });
          index += 1;
        });

        // this.categoriesWithSides = this.categories.map(category => {category: category.category, })
      }.bind(this)
    );
    console.log("HELLO!!!");
    console.log(this);
    console.log("-------");

    // var other = axios.get();
  },
  mounted: function() {
    // all the js for the cube
    this.cube = document.querySelector(".cube");
    this.radioGroup = document.querySelector(".radio-group");
    this.currentClass = "";
  },

  methods: {
    upOne: function(category) {
      if (category.currentArticleIndex < category.data.length) {
        category.currentArticleIndex += 1;
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
      category.currentArticleVisible = !category.currentArticleVisible;
      console.log("NEW STATUS:");
      console.log(category.currentArticleVisible);
      console.log(this);

    },
    // cube code
    changeSide: function(event) {
      var checkedRadio = this.radioGroup.querySelector(":checked");
      var showClass = "show-" + checkedRadio.value;
      if (this.currentClass) {
        this.cube.classList.remove(this.currentClass);
      }
      this.cube.classList.add(showClass);
      this.currentClass = showClass;
    }

  },
  computed: {}
};
</script>
