<template>
  <div class="app-container">
    <el-card>
      <el-form ref="dataForm" :rules="rules" :model="temp" class="form-container">
        <sticky :z-index="10" style="padding-left: 80%;height: 60px; padding-top: 10px">
          <el-button style="margin-left: 10px;" type="success" @click="handleCreate('published')">
            Publush
          </el-button>
          <el-button type="warning" @click="handleCreate('draft')">
            Draft
          </el-button>
        </sticky>
        <el-row style="padding-top: 10px">
          <el-form-item label-width="70px" label="编号：" prop="noticeId">
            <el-input v-model="temp.noticeId" :disabled="this.showFlag" style="width: 20%" />
          </el-form-item>
          <el-form-item label-width="100px" label="发布部门：" class="postInfo-container-item" prop="noticePublish">
            <el-select v-model="temp.noticePublish.publishId" placeholder="选择发布部门">
              <el-option v-for="item in noticePublish" :key="item.publishId" :label="item.typePublish" :value="item.publishId" />
            </el-select>
          </el-form-item>
          <el-form-item label-width="100px" label="发布日期： " class="postInfo-container-item" prop="noticeDate">
            <el-date-picker v-model="temp.noticeDate" value-format="yyyy-MM-dd" type="date" placeholder="Please pick a date" />
          </el-form-item>
        </el-row>
        <el-form-item style="margin-bottom: 40px;" label-width="70px" label="标题：" prop="noticeTitle">
          <el-input v-model="temp.noticeTitle" :rows="1" type="textarea" class="article-textarea" autosize placeholder="请输入标题" />
        </el-form-item>
        <el-form-item style="margin-bottom: 30px;">
          <Tinymce ref="editor" v-model="temp.noticeContent" :height="400" />
        </el-form-item>
      </el-form>
    </el-card>

  </div>
</template>
<script>
import Tinymce from '@/components/Tinymce'
import Sticky from '@/components/Sticky' // 粘性header组件
import { mapState, mapActions } from 'vuex'

const noticePublish = [
  { publishId: 1, typePublish: '人力资源部' },
  { publishId: 2, typePublish: '办公厅' },
  { publishId: 3, typePublish: '综合部' },
  { publishId: 4, typePublish: '监察室' }
]

export default {
  components: { Tinymce, Sticky },
  data() {
    return {
      noticePublish,
      temp: {
        noticeId: '',
        noticeDate: '',
        noticePublish: { publishId: '', typePublish: '' },
        noticeTitle: '',
        noticeContent: '',
        noticeReadNum: 0,
        noticeStatus: 'draft'
      },
      showFlag: false,
      rules: {
        noticeId: [
          { required: true, message: '请输入编号', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        noticePublish: [
          { required: true, message: '请选择发布部门', trigger: 'change' }
        ],
        noticeDate: [
          { required: true, message: '请选择日期', trigger: 'change' }
        ],
        noticeTitle: [
          { required: true, message: '请输入标题', trigger: 'blur' },
          { min: 7, max: 14, message: '长度在 7到 14 个字符', trigger: 'blur' }
        ]
      },
      newFlag: true
    }
  },
  computed: {
    ...mapState(['notice'])
  },
  mounted() {
    if (this.$route.query.id > 0) {
      for (const v of this.notice.notice) {
        if (v.noticeId === this.$route.query.id) {
          this.temp = v
          break
        }
      }
      this.showFlag = true
    }
  },

  methods: {
    ...mapActions(['addNotice', 'updateNotice']),
    resetTemp() {
      this.temp = {
        noticeId: '',
        noticeDate: '',
        noticePublish: { publishId: -1, typePublish: '' },
        noticeTitle: '',
        noticeContent: '',
        noticeReadNum: '',
        noticeStatus: 'published'
      }
    },
    handleCreate(dd) {
      this.newFlag = true
      if (this.temp.noticePublish.publishId !== '' && this.temp.noticeContent !== '' && this.temp.noticeTitle !== '' && this.temp.noticeDate !== null && this.temp.noticeId !== '') {
        if (!this.showFlag) {
          for (const a of this.notice.notice) {
            if (a.noticeId == this.temp.noticeId) {//eslint-disable-line
              this.newFlag = false
              this.temp.noticeId = ''
              alert('编号已存在，请重新输入')
              break
            }
          }
        }
        if (this.newFlag) {
          this.temp.editIf = true
          this.temp.noticePublish.typePublish = noticePublish[this.temp.noticePublish.publishId - 1].typePublish
          this.temp.noticeReadNum = 0
          this.temp.noticeStatus = dd
          if (this.showFlag) {
            this.updateNotice(this.temp)
          } else {
            this.addNotice(this.temp)
          }
          this.showFlag = false
          this.$notify({
            title: '成功',
            message: dd + '成功',
            type: 'success',
            duration: 2000
          })
          this.resetTemp()
          this.$router.push({ path: '/notice/page' })
        }
      } else {
        alert('请输入完整信息！')
      }
    }
  }

}
</script>
<style lang="scss" scoped>
  @import '../../styles/mixin.scss';

  .createPost-container {
    position: relative;

    .createPost-main-container {
      padding: 40px 45px 20px 50px;

      .postInfo-container {
        position: relative;
        @include clearfix;
        margin-bottom: 10px;

        .postInfo-container-item {
          float: left;
        }
      }
    }

    .word-counter {
      width: 40px;
      position: absolute;
      right: 10px;
      top: 0px;
    }
  }

  .article-textarea /deep/ {
    textarea {
      padding-right: 40px;
      resize: none;
      border: none;
      border-radius: 0px;
      border-bottom: 1px solid #bfcbd9;
    }
  }
</style>
