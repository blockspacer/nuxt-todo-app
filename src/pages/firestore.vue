<template>
  <div class="container">
     <table class="table is-narrow">
       <tbody>
         <tr>
           <th>todo</th>
           <th>limit</th>
         </tr>
       </tbody>
       <tbody>
         <tr v-for="(todo, index) in todos" :key="index">
           <td>{{todo.todo}}</td>
           <td>{{todo.limit}}</td>
         </tr>
       </tbody>
     </table>
     <div class="field is-grouped">
      <p class="control is-expanded">
        <input v-model="newTodo" class="input" type="text" placeholder="todo">
      </p>
      <p class="control is-expanded">
        <input v-model="newLimit" class="input" type="text" placeholder="limit">
      </p>
      <p class="control">
        <a class="button is-primary" @click="addTodo">
          add
        </a>
      </p>
      <hr />
      <p>Elasticsearch</p>
      <el-form ref="form" :model="form" label-width="120px" style="width: 500px">
        <el-form-item label="検索ワード">
          <el-input type="text" placeholder="Please input" v-model="form.searchText"></el-input>
        </el-form-item>
        <el-form-item label="カテゴリー">
          <el-select v-model="form.category" placeholder="カテゴリー">
            <el-option
              v-for="item in categoryList"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="サブカテゴリー">
          <el-select v-model="form.subC" placeholder="サブカテゴリー">
            <el-option
              v-for="item in subCategoryList"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-button class="button" @click="search">Search</el-button>
      </el-form>  
      <div  v-for="item in items">
        <span>ID:{{ item.source.id }} </span>
        <span>NAME:{{ item.source.itemName }}</span>
      </div>

    </div>
  </div>
</template>

<script>
import firebase from '~/plugins/firebase'
import 'firebase/firestore'
import { mapGetters } from 'vuex'
import itemOption from "./itemOption.json"
import undefined from 'firebase/firestore'

export default {
  data() {
    return {
      newTodo: '',
      newLimit: '',
      items: [],
      unsubscribe: null,
      ref_req: firebase.firestore().collection('search_request'),
      ref_res: firebase.firestore().collection('search_response'),
      categoryList : itemOption.categoryList,
      subCategoryList : itemOption.subCategoryList1,
      form: {
        searchText: '',
        subC: ''
      }
    }
  },
  computed: {
    ...mapGetters({ todos: 'firestoreItem/getTodos' })
  },
  methods: {
    // ...mapActions({
    //   getOnSaleItems: "item/getOnSaleItems"
    // }),
    addTodo() {
      const todo = this.newTodo
      const limit = this.newLimit
      
      this.$store.dispatch('firestoreItem/addTodo', {todo, limit})
      this.newTodo = ''
      this.newLimit = ''
    },
    async search() {
      console.log(this.subC == undefined)
      if (this.subC !== undefined && this.searchText == undefined) {
        this.getCategorizedItemFromFirestore() 
      } else {
        let query = {
          // formで変数は変更可能にできる
          index: 'flibra',
          type: 'item',
          size: 20,
          from: 0,
          searchText: this.searchText,
          subC: this.subC
        }
        console.log(query)
        //let ref_req = await firebase.firestore().collection('search_request');
        const snap = await this.ref_req.add(query);
        const key = await snap.id;
        this.unsubscribe = await this.ref_res.doc(key).onSnapshot(this.showResults);
      }
    },
    async showResults(snap) {
      if (snap.data() == undefined ){
        return;
      }else{
        this.items = snap.data();
        //  [{
        //   key: 0,
        //   source: snap.data(),
        // }],
        await snap.ref.delete()
        this.unsubscribe = null;
      }
    },
    async getCategorizedItemFromFirestore() {
      await this.getCategorizedItem
      this.item = this.$store.state.item.categorizedItem
    }
  },
  async created() {
    this.$store.dispatch('firestoreItem/setTodosRef')
  },
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  flex-direction: column;
}
</style>