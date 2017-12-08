<template>
  <div class="hello">
    <vue-custom-form :model="fileModel" :fields="fileFields" :formInfo="FormInfo"></vue-custom-form>
  </div>
</template>

<script>
import VueCustomForm from "@/components/VueCustomForm";
export default {
  name: 'HelloWorld',
  data () {
    return {
      fileModel:{},
      fileFields:[],
      FormInfo:{}
    }
  },
  created(){
    this.$http.get("/static/json/form.json").then(res => {
      this.fileModel = res.data.Model
      this.fileFields = res.data.Fields
      this.FormInfo = res.data.FormInfo
    })
  },
  components:{VueCustomForm}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
