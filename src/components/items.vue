<template lang="pug">
.items
  _loading(v-if="!data")
  template(v-else)
    page-nav(
      :currentPage="page",
      :totalPage="totalPage",
      :onPrev="prev",
      :onNext="next",
    )
    .items__items
      list-item(
        v-for="item in items",
        :key="item.id",
        :item="item",
      )
      _loading(v-if="!items")
</template>

<script>
import axios from "axios"
import ListItem from "@/components/list-item"
import PageNav from "@/components/page-nav"

export default {
  props: {
    dataURL: {
      type: String,
      required: true,
    },
    path: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      page: null,
      data: null,
      items: null,
    }
  },
  computed: {
    totalPage() {
      return Math.ceil(this.data.length / 20)
    },
  },
  methods: {
    fetchItems() {
      var url = "https://hacker-news.firebaseio.com/v0/item/"
      var start = (this.page - 1) * 20
      var ids = this.data.slice(start, start + 20)
      Promise.all(ids.map(function(id) {
        return axios.get(url + id + ".json")
      })).then(d => {
        this.items = d.map(res => res.data)
      })
    },
    prev() {
      this.$router.push(this.path + "/" + (this.page - 1))
    },
    next() {
      this.$router.push(this.path + "/" + (this.page + 1))
    },
  },
  created() {
    axios.get(this.dataURL)
      .then(res => {
        this.data = res.data
        this.page = Number(this.$route.params.page) || 1
      })
  },
  watch: {
    "$route": function() {
      this.page = Number(this.$route.params.page) || 1
      this.items = null
    },
    page: function() {
      this.fetchItems()
    },
  },
  components: { ListItem, PageNav },
}
</script>

<style lang="stylus">
.items
  > ._loading
    margin: 200px 0
    text-align: center
    font-size: 1.5rem
    color: #aaa

.items__items
  max-width: 800px
  margin: auto
  margin-top: 20px
  margin-bottom: 30px
  background: white
  padding: 20px
  border-radius: 5px
</style>
