<template>
  <div>
    <div class='auth' v-if="!token" @click="OAuth">
      <img src="../images/auth.png">
      <span>&nbsp;授权</span>
    </div>
    <h1>获取GitHub仓库的关注者信息</h1>
    <p class='search-bar'>
      <el-input v-model="user" size="medium" @keyup.enter.native="getRepoInfo" placeholder="请输入仓库作者"></el-input>
      <el-input v-model="repo" size="medium" @keyup.enter.native="getRepoInfo" placeholder="请输入仓库名称"></el-input>
      <el-button type="primary" size="medium" icon="el-icon-search" @click="getRepoInfo">搜索</el-button>
    </p>
    <ul v-if="Object.keys(repoInfo).length>0" class='repo'>
      <li><b>项目名: </b><a :href="repoInfo.html_url" target="_blank">{{repoInfo.name}}</a></li>
      <li><b>是否原创: </b><span>{{repoInfo.fork?'否':'是'}}</span></li>
      <li><b>Watch: </b><span>{{repoInfo.subscribers_count}}</span></li>
      <li><b>Star: </b><span>{{repoInfo.stargazers_count}}</span></li>
      <li><b>Fork: </b><span>{{repoInfo.forks_count}}</span></li>
      <li><b>创建时间: </b><span>{{repoInfo.created_at | date}}</span></li>
      <li><b>上次更新: </b><span>{{repoInfo.updated_at | date}}</span></li>
    </ul>
    <el-table :data="tableData" v-loading="loading" size="medium" border style="width: 100%">
      <el-table-column type="index" label="序号" :index="indexMethod" align="center" header-align="center" width="80"></el-table-column>
      <el-table-column align="center" header-align="center" label="头像" width="100">
        <template slot-scope="scope">
          <img class='avatar' :src="scope.row.avatar_url">
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="账号" width="150">
        <template slot-scope="scope">
          <a :href="scope.row.html_url" target="_blank">{{scope.row.login}}</a>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" sortable prop='public_repos' label="公开仓库数" width="120">
        <template slot-scope="scope">
          <a :href="scope.row.html_url + '?tab=repositories'" target="_blank">{{scope.row.public_repos}}</a>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" sortable prop='followers' label="粉丝" width="100">
        <template slot-scope="scope">
          <a :href="scope.row.html_url + '?tab=followers'" target="_blank">{{scope.row.followers}}</a>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" sortable prop='following' label="关注" width="100">
        <template slot-scope="scope">
          <a :href="scope.row.html_url + '?tab=following'" target="_blank">{{scope.row.following}}</a>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="博客" min-width="180">
        <template slot-scope="scope">
          <a v-if="scope.row.blog" :href="scope.row.blog" target="_blank">{{scope.row.blog}}</a>
          <span v-else>无博客链接</span>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="邮箱" min-width="160">
        <template slot-scope="scope">
          <a v-if="scope.row.email" :href="'mailto:'+scope.row.email">{{scope.row.email}}</a>
          <span v-else>{{token?'无公开邮箱':'尚未授权,无法获取邮箱地址'}}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" label="所在地" width="120">
        <template slot-scope="scope">
          {{scope.row.location?scope.row.location:'不详'}}
        </template>
      </el-table-column>
      <el-table-column align="center" header-align="center" sortable prop='created_at' label="账号创建时间" width="180">
        <template slot-scope="scope">
          <span>{{scope.row.created_at | date}}</span>
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination">
      <el-pagination background layout="prev, pager, next, sizes, jumper" :current-page.sync="page" :page-size="per_page" :page-sizes="[10, 20, 30, 50, 100]" :total="total" @size-change="pageSizeChange" @current-change="getStargazers"></el-pagination>
    </div>
    <p class='des'>Star过万的项目,调转到最后几页可能会报错</p>
  </div>
</template>

<script>
import axios from 'axios'
import { Notification } from 'element-ui'

const CLIENT_ID = '2a7f21547241f40029d1'
const REDIRECT_URI = 'https://haochuan9421.github.io/stargazers/'

axios.interceptors.response.use(function (response) {
  return response
}, function (error) {
  let errorObj = JSON.parse(JSON.stringify(error))
  if (errorObj.response.data.message === 'Bad credentials') {
    window.localStorage.removeItem('github_token')
  }
  Notification({
    type: 'error',
    title: '错误',
    duration: 10000,
    dangerouslyUseHTMLString: true,
    message: `<a href="${errorObj.response.data.documentation_url}" target="_blank" style="color:#409EFF;">${errorObj.response.data.message}</a>`
  })
  return Promise.reject(error)
})
export default {
  data () {
    return {
      repoInfo: {},
      user: 'ElemeFE',
      repo: 'element',
      token: '',
      page: 1,
      per_page: 10,
      total: 0,
      tableData: [],
      loading: false
    }
  },
  created () {
    this.checkAuth()
  },
  methods: {
    checkAuth () {
      const isCode = new RegExp(/\?code=([0-9A-Za-z]{20})/)
      const codeMatch = isCode.exec(window.location.href)
      const token = window.localStorage.getItem('github_token')
      if (token) {
        this.token = token
        this.getRepoInfo()
      } else if (codeMatch) {
        window.history.replaceState(null, null, window.location.href.replace(isCode, ''))
        window.history.go(1)
        this.getToken(codeMatch[1])
      } else {
        this.$msgbox({
          title: '温馨提示',
          message: 'Github的API在无access_token的情况下,调用接口时会针对IP进行次数限制,而且无法获取其他用户的公开邮箱,是否授权获取更完整体验',
          showCancelButton: true,
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(() => {
          this.OAuth()
        }).catch(() => {
          this.$message({
            type: 'warning',
            showClose: true,
            message: '您可以稍后点击右上角的登录授权按钮重新解锁权限',
            duration: 6000
          })
          this.getRepoInfo()
        })
      }
    },
    OAuth () {
      window.location.href = `https://github.com/login/oauth/authorize?client_id=${CLIENT_ID}&redirect_uri=${encodeURIComponent(REDIRECT_URI)}`
    },
    getToken (code) {
      const loading = this.$loading({
        lock: true,
        text: '获取access_token',
        background: 'rgba(0, 0, 0, 0.7)'
      })
      axios({
        baseURL: 'https://www.zhc.im/token',
        params: { code }
      }).then((res) => {
        if (res.data.code === 200) {
          this.token = res.data.data
          window.localStorage.setItem('github_token', this.token)
          this.$message.success('授权成功')
          this.getRepoInfo()
        } else {
          this.$message.error('授权失败')
        }
      }).catch(() => {
        this.$message.error('授权失败')
      }).finally(() => {
        loading.close()
      })
    },
    getRepoInfo () {
      this.page = 1

      let options = {
        method: 'get',
        baseURL: `https://api.github.com/repos/${this.user}/${this.repo}`
      }
      if (this.token) {
        options.params = {
          access_token: this.token
        }
      }
      axios(options).then((res) => {
        this.repoInfo = res.data
        this.total = res.data.stargazers_count
        this.getStargazers()
      })
    },
    getStargazers () {
      if (this.user === '' || this.repo === '') {
        this.$message('搜素条件不完整')
        return
      }
      this.loading = true
      this.tableData = []
      let options = {
        method: 'get',
        baseURL: `https://api.github.com/repos/${this.user}/${this.repo}/stargazers`,
        params: {
          page: this.page,
          per_page: this.per_page
        }
      }
      if (this.token) {
        options.params.access_token = this.token
      }

      axios(options).then((res) => {
        if (res.data.length) {
          let allRequest = []
          res.data.forEach((item) => {
            allRequest.push(this.getUserDetail(item.login))
          })
          Promise.all(allRequest).then((userData) => {
            this.tableData = userData.map(item => item.data)
            Notification({
              message: `成功获取了${userData.length}条关注者信息`,
              type: 'success',
              duration: 2000
            })
          }).finally(() => {
            this.loading = false
          })
        } else {
          Notification({
            message: `无关注者信息`,
            type: 'warning',
            duration: 2000
          })
          this.loading = false
          this.tableData = []
        }
      }).catch(() => {
        this.loading = false
        this.tableData = []
      })
    },
    getUserDetail (user) {
      let options = {
        method: 'get',
        baseURL: `https://api.github.com/users/${user}`
      }
      if (this.token) {
        options.params = {
          access_token: this.token
        }
      }
      return axios(options)
    },
    pageSizeChange (size) {
      console.log(size)
      this.per_page = size
      this.getStargazers()
    },
    indexMethod (index) {
      return (this.page - 1) * this.per_page + index + 1
    }
  },
  filters: {
    date (time = Date.now(), fmt = 'yyyy-MM-dd HH:mm') {
      const date = new Date(time)
      if (date === 'Invalid Date') return time
      const obj = {
        'y': date.getFullYear(),
        'M': date.getMonth() + 1,
        'd': date.getDate(),
        'q': Math.floor((date.getMonth() + 3) / 3),
        'w': date.getDay(),
        'H': date.getHours(),
        'h': date.getHours() % 12 === 0 ? 12 : date.getHours() % 12,
        'm': date.getMinutes(),
        's': date.getSeconds(),
        'S': date.getMilliseconds()
      }
      const week = ['天', '一', '二', '三', '四', '五', '六']
      for (const i in obj) {
        fmt = fmt.replace(new RegExp(i + '+', 'g'), function (m) {
          let val = obj[i] + ''
          if (i === 'w') return (m.length > 2 ? '星期' : '周') + week[val]
          for (let j = 0, len = val.length; j < m.length - len; j++) val = '0' + val
          return m.length === 1 ? val : val.substring(val.length - m.length)
        })
      }
      return fmt
    }
  }
}
</script>

<style lang="scss" scoped>
h1, h2 {
  text-align: center;
  color: #42b983;
}
.search-bar{
  display: flex;
  justify-content: center;
  align-items: center;
  .el-input{
    width: 240px;
    margin-right: 10px;
  }
}
.repo{
  margin: 30px 0;
  list-style: none;
  text-align: center;
  li{
    display: inline-block;
    margin-right: 10px;
    span{
      color: #666;
    }
  }
}
.avatar{
  width: 60px;
  height: 60px;
  border-radius: 5px;
}
a{
  color: #42b983;
}
.pagination{
  width: 100%;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.des{
  color: #aaaaaa;
  font-size: 12px;
  text-align: center;
}
.auth{
  position: fixed;
  top: 10px;
  right: 20px;
  display: flex;
  align-items: center;
  cursor: pointer;
  color: #1296db;
  img{
    width: 20px;
    height: 20px;
  }
}
</style>
