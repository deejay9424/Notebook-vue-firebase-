<template>
  <div id="app">
    <Notebook @change-page="changePage" @new-page="newPage" :pages="pages" :activePage="index" />

    <Page @save-page="savePage" @delete-page="deletePage" :page="pages[index]" />
  </div>
</template>

<script>
import Notebook from "./components/Notebook";
import Page from "./components/Page";
import Firebase from "firebase";

var database = Firebase.initializeApp({
  apiKey: "apiKey",
  authDomain: "https://notebook-bdf97.firebaseapp.com/",
  databaseURL: "https://notebook-bdf97.firebaseio.com/",
  storageBucket: "storageBucket"
}).database().ref();

export default {
  name: "app",
  components: {
    Notebook,
    Page
  },
  data: () => ({
    pages: [],
    index: 0
  }),
  methods: {
    newPage() {
      this.pages.push({
        title: "",
        content: ""
      });
      this.index = this.pages.length - 1;
    },
    changePage(index) {
      this.index = index;
    },
    savePage() {
      var page = this.pages[this.index]
      if (page.ref) {
        this.updateExistingPage(page)
      } else {
        this.insertNewPage(page)
      }
    },
    insertNewPage (page) {
      page.ref = database.push(page)
    },
    updateExistingPage (page) {
      page.ref.update({
        title: page.title,
        content: page.content
      })
    },
    deletePage() {
      var ref = this.pages[this.index].ref;
      ref && ref.remove();
      this.pages.splice(this.index, 1);
      this.index = Math.max(this.index - 1, 0);
    }
  },
   mounted() {
      database.once('value', (pages) => {
        pages.forEach((page) => {
          this.pages.push({
            ref: page.ref,
            title: page.child('title').val(),
            content: page.child('content').val()
          })
        })
      })
    },
};
</script>

<style>
html,
body,
#app {
  height: 100%;
}

body {
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  display: flex;
  flex-direction: row;
}
</style>
