<template>
  <div class="quiz-admin">
    <header class="page-title">Quiz后台管理</header>
    <div class="workspace">
      <aside class="side-nav">
        <div class="nav-group-title"><i class="el-icon-message"></i><span>系统信息管理</span><i class="el-icon-arrow-up"></i></div>
        <el-menu :default-active="activeMenu" class="nav-menu" @select="switchPage">
          <el-menu-item index="users"><span>用户管理</span></el-menu-item>
          <el-menu-item index="questions"><span>题目管理</span></el-menu-item>
        </el-menu>
      </aside>

      <main class="main-content">
        <section class="search-bar">
          <span class="search-label">{{ isUserPage ? '用户名' : '题目' }}</span>
          <el-input v-model.trim="keyword" class="query-input" :placeholder="isUserPage ? '用户名' : '题目关键字'" clearable @keyup.enter.native="search"></el-input>
          <el-button type="primary" @click="search">查询{{ isUserPage ? '用户' : '题目' }}</el-button>
          <el-button type="success" @click="openCreate">添加{{ isUserPage ? '用户' : '题目' }}</el-button>
        </section>

        <section class="data-area">
          <el-table v-if="isUserPage" :data="pagedRows" class="simple-table" :header-cell-style="headerStyle" empty-text="暂无用户数据">
            <el-table-column label="日期" width="145"><template slot-scope="scope"><i class="el-icon-time date-icon"></i>{{ scope.row.date }}</template></el-table-column>
            <el-table-column label="用户名" width="155"><template slot-scope="scope"><el-tag size="small" type="info">{{ scope.row.name }}</el-tag></template></el-table-column>
            <el-table-column prop="password" label="密码" width="140"></el-table-column>
            <el-table-column label="操作" min-width="180"><template slot-scope="scope"><el-button size="mini" @click="openEdit(scope.row)">编辑</el-button><el-button size="mini" type="danger" @click="remove(scope.row)">删除</el-button></template></el-table-column>
          </el-table>

          <el-table v-else :data="pagedRows" class="simple-table" :header-cell-style="headerStyle" empty-text="暂无题目数据">
            <el-table-column type="index" label="序号" width="95"></el-table-column>
            <el-table-column prop="title" label="题目" min-width="230"></el-table-column>
            <el-table-column label="选项" min-width="220"><template slot-scope="scope"><span class="option-text">{{ scope.row.options.join('，') }}</span></template></el-table-column>
            <el-table-column prop="answer" label="答案" min-width="120"></el-table-column>
            <el-table-column label="操作" min-width="160"><template slot-scope="scope"><el-button size="mini" @click="openEdit(scope.row)">编辑</el-button><el-button size="mini" type="danger" @click="remove(scope.row)">删除</el-button></template></el-table-column>
          </el-table>
          <el-pagination class="pagination" background layout="prev, pager, next" :total="filteredRows.length" :page-size="pageSize" :current-page.sync="page"></el-pagination>
        </section>
      </main>
    </div>

    <el-dialog :title="`${editingId ? '编辑' : '添加'}${isUserPage ? '用户' : '题目'}`" :visible.sync="dialogVisible" width="480px" :close-on-click-modal="false">
      <el-form ref="form" :model="form" :rules="rules" label-width="72px">
        <template v-if="isUserPage">
          <el-form-item label="用户名" prop="name"><el-input v-model.trim="form.name" placeholder="请输入用户名"></el-input></el-form-item>
          <el-form-item label="密码" prop="password"><el-input v-model.trim="form.password" placeholder="请输入密码"></el-input></el-form-item>
          <el-form-item label="日期"><el-date-picker v-model="form.date" type="date" value-format="yyyy-MM-dd" placeholder="选择日期"></el-date-picker></el-form-item>
        </template>
        <template v-else>
          <el-form-item label="题目" prop="title"><el-input v-model.trim="form.title" type="textarea" :rows="2" placeholder="请输入题目"></el-input></el-form-item>
          <el-form-item label="选项"><el-input v-model="form.optionsText" placeholder="用中文逗号分隔选项"></el-input></el-form-item>
          <el-form-item label="答案" prop="answer"><el-input v-model.trim="form.answer" placeholder="请输入答案"></el-input></el-form-item>
        </template>
      </el-form>
      <span slot="footer"><el-button @click="dialogVisible=false">取消</el-button><el-button type="primary" @click="save">确定</el-button></span>
    </el-dialog>
  </div>
</template>

<script>
const users = [
  { id: 1, date: '2016-05-02', name: '王小虎', password: '123456' },
  { id: 2, date: '2016-05-04', name: '王小虎', password: '123456' },
  { id: 3, date: '2016-05-01', name: '王小虎', password: '123456' },
  { id: 4, date: '2016-05-03', name: '王小虎', password: '123456' },
  { id: 5, date: '2016-05-06', name: '张小明', password: '123456' },
  { id: 6, date: '2016-05-09', name: '李小红', password: '123456' },
  { id: 7, date: '2016-05-10', name: '刘小云', password: '123456' }
]
const questions = [
  { id: 1, title: '法国的首都是哪座城市？', options: ['巴黎', '伦敦', '罗马', '巴塞罗那'], answer: '巴黎' },
  { id: 2, title: '地球上面积最大的海洋是？', options: ['太平洋', '大西洋', '印度洋', '北冰洋'], answer: '太平洋' },
  { id: 3, title: '以下哪个是 JavaScript 框架？', options: ['Vue', 'MySQL', 'Linux', 'Nginx'], answer: 'Vue' },
  { id: 4, title: '水的化学分子式是什么？', options: ['H2O', 'CO2', 'O2', 'NaCl'], answer: 'H2O' },
  { id: 5, title: '中国的首都是哪座城市？', options: ['北京', '上海', '广州', '深圳'], answer: '北京' },
  { id: 6, title: '一年有几个季节？', options: ['四个', '三个', '五个', '两个'], answer: '四个' },
  { id: 7, title: 'HTML 的主要作用是什么？', options: ['页面结构', '数据库', '服务器', '图片处理'], answer: '页面结构' }
]

export default {
  data() {
    return {
      activeMenu: 'users', keyword: '', page: 1, pageSize: 4, dialogVisible: false, editingId: null, form: {},
      userRows: this.readStorage('simple-quiz-users', users), questionRows: this.readStorage('simple-quiz-questions', questions),
      rules: {
        name: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        title: [{ required: true, message: '请输入题目', trigger: 'blur' }],
        answer: [{ required: true, message: '请输入答案', trigger: 'blur' }]
      }
    }
  },
  computed: {
    isUserPage() { return this.activeMenu === 'users' },
    sourceRows() { return this.isUserPage ? this.userRows : this.questionRows },
    filteredRows() {
      const keyword = this.keyword.toLowerCase()
      if (!keyword) return this.sourceRows
      return this.sourceRows.filter(row => this.isUserPage ? row.name.toLowerCase().includes(keyword) : row.title.toLowerCase().includes(keyword))
    },
    pagedRows() { return this.filteredRows.slice((this.page - 1) * this.pageSize, this.page * this.pageSize) },
    headerStyle() { return { background: '#fff', color: '#606266', fontWeight: '500', fontSize: '14px' } }
  },
  watch: { keyword() { this.page = 1 } },
  methods: {
    readStorage(key, fallback) { try { return JSON.parse(localStorage.getItem(key)) || fallback } catch { return fallback } },
    persist() { localStorage.setItem('simple-quiz-users', JSON.stringify(this.userRows)); localStorage.setItem('simple-quiz-questions', JSON.stringify(this.questionRows)) },
    switchPage(key) { this.activeMenu = key; this.keyword = ''; this.page = 1 },
    search() { this.page = 1 },
    newForm() { return this.isUserPage ? { name: '', password: '', date: '2016-05-02' } : { title: '', optionsText: '', answer: '' } },
    openCreate() { this.editingId = null; this.form = this.newForm(); this.dialogVisible = true; this.$nextTick(() => this.$refs.form && this.$refs.form.clearValidate()) },
    openEdit(row) { this.editingId = row.id; this.form = { ...row, optionsText: row.options ? row.options.join('，') : '' }; this.dialogVisible = true },
    save() {
      this.$refs.form.validate(valid => {
        if (!valid) return
        const list = this.sourceRows
        const record = { ...this.form }
        if (!this.isUserPage) { record.options = record.optionsText.split(/[，,]/).map(v => v.trim()).filter(Boolean); delete record.optionsText }
        if (this.editingId) Object.assign(list.find(item => item.id === this.editingId), record)
        else list.push({ ...record, id: Date.now() })
        this.persist(); this.dialogVisible = false; this.$message.success('保存成功')
      })
    },
    remove(row) {
      this.$confirm('确定删除该条数据吗？', '提示', { type: 'warning' }).then(() => {
        if (this.isUserPage) this.userRows = this.userRows.filter(item => item.id !== row.id)
        else this.questionRows = this.questionRows.filter(item => item.id !== row.id)
        this.persist(); this.$message.success('删除成功')
      }).catch(() => {})
    }
  }
}
</script>

<style src="./style.css"></style>
