<template lang="jade">
#admin-posts.admin
  .title
    h1 {{$route.meta.title}}
    el-button(type='text', @click="$router.push('/push/new')") 添加推送
  el-table(:data='listData.json' border)
    el-table-column(prop='content', label='内容')
    el-table-column(prop='created_at', label='创建时间', width="180")
    el-table-column(prop='time_send_at', label='推送时间', width="180")
    el-table-column(prop='redirect', label='内容ID', width="100")
      template(scope='scope')
        span.preview(@click='preview(scope.row)') {{scope.row.redirect}}
    //- el-table-column(prop='', label='iOS点击', width="150")
    //- el-table-column(prop='', label='Android点击', width="150")
    //- el-table-column(label='操作', width="100")
      //- template(scope='scope')
      //-   el-button(type='text',
      //-             @click='handleDestroy(scope.$index, scope.row, listData.posts)') 删除
  el-pagination(@size-change='handleSizeChange',
                @current-change='handleCurrentChange',
                :current-page='currentPage',
                :page-size='listData.meta.limit_value',
                layout='total, prev, pager, next, jumper',
                :total='listData.meta.total_count')
</template>

<script>
import tool from 'tools'
import api from 'stores/api'
import config from '../../config.js'

const url = 'api/v1/broadcasts'

export default {
  data () {
    return {
      params: {
        // content: ''
      },
      currentPage: 1,
      listData: {
        list: [],
        meta: {
          total_count: 0,
          limit_value: 0
        }
      }
    }
  },
  methods: {
    preview (row) {
      window.open(`${config.main}/news/${row.redirect}`)
    },
    search () {
      // api.acc.get(url, {params: this.params}).then(result => {
      //   console.log(result)
      //   this.listData = result.data
      // }).catch((err) => {
      //   console.log(err)
      // })
    },
    handleSizeChange (index, val) {
      console.log(`每页 ${index} 条`)
    },
    handleCurrentChange (index, val) {
      this.currentPage = index
      this.fetch()
      console.log(`当前页: ${index}`)
    },
    fetch () {
      const params = Object.assign({page: this.currentPage}, this.params)
      api.account.get(url, {params: params}).then((result) => {
        console.log(result)
        this.listData = result.data
      }).catch((err) => {
        console.log(err)
        this.$message.error(err.toString())
      })
    },
    handleDestroy (index, val, list) {
      // const destroy = () => {
      //   api.delete(`${url}/${val.id}`).then((result) => {
      //     this.$message.success('success')
      //     this.fetch()
      //   }).catch((err) => {
      //     console.log(err)
      //     this.$message.error(err.toString())
      //   })
      // }
      // tool.deleteConfirm(this, destroy)
    }
  },
  watch: {
    'listData.json': function (val) {
      if (val === undefined) {
        return
      }
      val.forEach(el => {
        el.created_at = tool.moment(new Date(el.created_at))
        if (el.time_send_at) {
          el.time_send_at = timetrans(el.time_send_at)
        }
      })
    },
    'params.state': function () {
      this.search()
    }
  },
  beforeMount () {
    this.fetch()
  }
}
function timetrans (date) {
  date = new Date(date * 1000)
  let Y = date.getFullYear() + '-'
  let M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-'
  let D = (date.getDate() < 10 ? '0' + (date.getDate()) : date.getDate()) + ' '
  let h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':'
  let m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':'
  let s = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds())
  return Y + M + D + h + m + s
}
</script>

<style lang="stylus" scoped>
.preview
  cursor pointer
</style>
