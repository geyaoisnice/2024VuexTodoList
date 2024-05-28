# 前言

> 大家好 我是歌谣 今天继续给大家带来Vuex+Vue2的了解


#  实现效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/512b7be9a73243f49afe1330b6b1a3d3.png)

# 目录结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4ba9932566054002ac9890745e22776b.png)

# node版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b9bf0be9d32b4eb2b7ca8c43a13486a8.png)

# 主代码app.vue

```css
<template>
  <div id="app">
    <h1>Vue2+ElementUi+Vuex+TodoList</h1>
    <h2>歌谣</h2>
    <el-input placeholder="请输入待办事项" class="td-input" :value="inputValue" @input="handleInputChange"></el-input>
    <el-button type="primary" @click="addItemToList">添加事项</el-button>
    <el-main class="td-main" >
        <el-table
          ref="multipleTable"
          :data="infolist"
          style="width: 100%"
          @selection-change="handleSelectionChange"
          @select="statusChanged"
          @select-all="statusChangedAll">
          <el-table-column
            type="selection"
            width="55"
            >
          </el-table-column>
          <el-table-column
            label="待办事项"
            width="350">
            <!-- 数据渲染 -->
            <template slot-scope="item">{{ item.row.info }}</template>
          </el-table-column>
          <el-table-column
            prop="name"
            label="删除"
            width="50">
            <el-button slot-scope="item" @click="removeItemById(item.row.id)" size="mini" class="btn-close" type="primary" icon="el-icon-close" circle></el-button>
          </el-table-column>
        </el-table>
        <div class="footer">
          <span>{{unDoneLength}}条剩余</span>
          <el-radio-group size="small" v-model="radio1" @change="changeList">
            <el-radio-button class="btn-radio" label="全部"></el-radio-button>
            <el-radio-button class="btn-radio" label="未完成"></el-radio-button>
            <el-radio-button class="btn-radio" label="已完成"></el-radio-button>
          </el-radio-group>
          <el-button @click="clean">清除已完成</el-button>
        </div>
    </el-main>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'

export default {
  name: 'app',
  data () {
    return {
      radio1: '全部'
    }
  },
  created () {
    this.$store.dispatch('getList')
  },
  computed: {
    ...mapState(['list', 'inputValue', 'viewKey']),
    ...mapGetters(['unDoneLength', 'infolist'])
  },
  methods: {
    // 监听文本框内容的变化
    handleInputChange (val) {
      this.$store.commit('setInputValue', val)
    },
    // 向列表中新增 item 项
    addItemToList () {
      // console.log(1111111)
      if (this.inputValue.trim().length <= 0) {
        return this.$message.warning('添加事项不能为空！')
      }
      // const obj = {
      //   id: this.list.length,
      //   info: this.inputValue.trim(),
      //   // 未完成状态默认是false
      //   done: false
      // }
      this.$store.commit('addItem')
      // this.$store.dispatch('addList', obj)
    },
    // 很据Id删除对应的任务事项
    removeItemById (id) {
      this.$store.commit('removeItem', id)
    },
    handleSelectionChange (val) {
      this.multipleSelection = val
    },
    // 复选框状态绑定
    checked () {
      this.$nextTick(() => {
        this.list.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row, row.done)
        })
      })
    },
    // 监听复选框选中状态变化事件
    statusChanged (val, row) {
      const param = {
        id: row.id,
        // 点击后状态取反
        status: !row.done
      }
      this.$store.commit('changeStatus', param)
    },
    statusChangedAll (val) {
      if (val.length !== 0) {
        this.$store.commit('changeStatusAll')
      } else {
        this.$store.commit('cleanStatusAll')
      }
    },
    // 清除已完成任务
    clean () {
      this.$store.commit('cleanDone')
    },
    // 修改页面上展示的列表数据
    changeList (key) {
      this.$store.commit('changeViewKey', key)
    }
  },
  watch: {
    infolist: function (val) {
      this.checked()
    }
  }
}
</script>

<style>
#app {
  width: 610px;
  margin: 0 auto;
}
.td-input {
  width: 500px;
  margin-right: 10px;
}
.td-main {
  width: 500px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #DCDFE6;
}
.btn-close {
  float: right;
}
.el-button--mini.is-circle {
  padding: 3px;
}
.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 15px;
}
</style>

```




```

# store.js

```css
import Vue from 'vue'
import Vuex from 'vuex'
import axios from 'axios'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    // 列表清单
    list: [],
    // 文本框输入内容
    inputValue: '',
    // 下一个Id
    nextId: 6,
    viewKey: '全部'
  },
  mutations: {
    initList (state, list) {
      state.list = list
    },
    // 为inputValue赋值到state中
    setInputValue (state, val) {
      state.inputValue = val
    },
    addItem (state) {
      const obj = {
        id: state.nextId,
        info: state.inputValue.trim(),
        // 未完成状态默认是false
        done: false
      }
      // 将对象追加到list中
      state.list.push(obj)
      // id自动递增，保证不重复
      state.nextId++
      // 清空文本框
      state.inputValue = ''
    },
    // 根据Id删除对应的任务事项
    removeItem (state, id) {
      // 根据Id查找对应项的索引
      const i = state.list.findIndex(x => x.id === id)
      // 根据索引，删除对应的元素
      if (i !== -1) {
        state.list.splice(i, 1)
      }
    },
    // 修改列表项的选中状态
    changeStatus (state, param) {
      const i = state.list.findIndex(x => x.id === param.id)
      if (i !== -1) {
        state.list[i].done = param.status
      }
    },
    // 修改全选后的状态
    changeStatusAll (state) {
      // console.log(params)
      state.list.forEach(row => {
        row.done = true
      })
    },
    // 取消全选后的状态
    cleanStatusAll (state) {
      state.list.forEach(row => {
        row.done = false
      })
    },
    cleanDone (state) {
      state.list = state.list.filter(x => x.done === false)
    },
    // 修改视图的关键字
    changeViewKey (state, key) {
      state.viewKey = key
    }
  },
  actions: {
    // 查询数据
    getList (context) {
      axios.get('/list.json').then(({ data }) => {
        context.commit('initList', data)
      })
    },
    // 增加数据
    addList (context, response) {
      console.log(context)
      console.log(response)
      axios.post('/list.json', response).then(({ data }) => {
        context.commit('initList', data)
      })
    }
  },
  getters: {
    // 统计未完成的任务数量
    unDoneLength (state) {
      return state.list.filter(x => x.done === false).length
    },
    infolist (state) {
      if (state.viewKey === '全部') {
        return state.list
      }
      if (state.viewKey === '未完成') {
        return state.list.filter(x => !x.done)
      }
      if (state.viewKey === '已完成') {
        return state.list.filter(x => x.done)
      }
      return state.list
    }
  },
  modules: {
  }
})

```


# 运行结果

> 需要源代码关注公众号前端小歌谣
