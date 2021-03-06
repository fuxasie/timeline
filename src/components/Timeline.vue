<template>
  <div v-infinite-scroll="loadNextPage" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
    <ul class="timeline">
      <li class="year first">2020</li>
      <li class="event" v-for="e in events" :key="e.id">
        <span class="meta">
          <span class="time">{{ e.timestamp }}</span>
          <span class="tag" v-for="t in e.tags" :key="t">{{ t }}</span>
        </span>
        <div class="md-result" v-html="e.htmlContent"></div>
      </li>
      <li class="event" v-show="allLoaded">
        - End -
      </li>
    </ul>
  </div>
</template>

<script>
const MarkdownIt = require("markdown-it");
const md = new MarkdownIt();
const axios = require("axios");

export default {
  name: "Timeline",
  data() {
    return {
      events: [],
      currentPage: 0,
      busy: false,
      allLoaded: false
    };
  },
  methods: {
    mdRender(mdStr) {
      return md.render(mdStr);
    },
    loadNextPage() {
      if (this.allLoaded) {
        return;
      }
      this.busy = true;
      axios.default
        .create()
        .get(`/data/data-${this.currentPage}.json`)
        .then((resp) => {
          for (let i = 0; i < resp.data.length; i++) {
            this.events.push(resp.data[i]);
          }
          this.events.map((e) => {
            e.htmlContent = this.mdRender(e.content);
            let d = new Date(Date.parse(e.timestamp));
            e.timestamp = `${d.getMonth() > 9 ? "" : "0"}${d.getMonth() + 1}-${
              d.getDate() > 9 ? "" : "0"
            }${d.getDate()} ${
              d.getHours() > 9 ? "" : "0"
            }${d.getHours()}:${d.getMinutes()}`;
          });
          this.currentPage = this.currentPage + 1;
          this.busy = false;
        })
        .catch((err) => {
          if (err.response.status === 404) {
            this.allLoaded = true;
            console.log("no more data");
          } else {
            console.log("fail");
          }
          this.busy = false;
        });
    },
  },
  mounted() {
    this.loadNextPage(null);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("/css/ie.css");
@import url("/css/print.css");
@import url("/css/screen.css");
@import url("/css/timeline.css");

/* line 166, ../sass/timeline.scss */
.event .meta {
  font-size: smaller;
  font-size-adjust: inherit;
  display: block;
  margin-bottom: 0.25em;
}

/* line 170, ../sass/timeline.scss */
.event .time {
  background-color: #505050;
  color: white;
  padding: 0 0.5em;
  width: auto;
  float: none;
  display: inline-block;
  border-radius: 5px;
  margin-right: 0.25em;
  margin-bottom: 0.25em;
}

/* line 180, ../sass/timeline.scss */
.event .tag {
  background-color: #ffdd40;
  color: #505050;
  width: auto;
  padding: 0 0.5em;
  display: inline-block;
  border-radius: 5px;
  margin-right: 0.25em;
}

.event .tag::before {
  content: "#";
}

.event .tag::after {
  content: "#";
}

.event .md-result {
  overflow-x: auto;
  overflow-y: hidden;
  text-align: left;
}
</style>