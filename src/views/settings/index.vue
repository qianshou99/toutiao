<template>
  <div class="settings-container">
      <el-card class="box-card">
        <div slot="header" class="clearfix">
            <!-- 面包屑路径导航 -->
            <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item to="/">首页</el-breadcrumb-item>
            <el-breadcrumb-item>个人设置</el-breadcrumb-item>
            </el-breadcrumb>
            <!-- /面包屑路径导航 -->
        </div>
        <el-row>
            <el-col :span="15">
            <el-form ref="form" :model="user" label-width="80px" :rules="formRules">
                <el-form-item label="编号" >
                {{ user.id }}
                </el-form-item>
                <el-form-item label="手机">
                {{ user.mobile }}
                </el-form-item>
                <el-form-item label="媒体名称" prop="name">
                <el-input v-model="user.name"></el-input>
                </el-form-item>
                <el-form-item label="媒体介绍" prop="intro">
                <el-input type="textarea" v-model="user.intro"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                <el-input v-model="user.email"></el-input>
                </el-form-item>
                <el-form-item>
                <el-button
                type="primary"
                :loading="updateProfileLoading"
                @click="onUpdateUser"
                >保存</el-button>
                </el-form-item>
            </el-form>
            </el-col>
            <!-- offset偏移 -->
            <el-col :offset="2" :span="4">
            <label for="file">
                <el-avatar
                    shape="square"
                    :size="150"
                    fit="cover"
                    :src="user.photo"
                ></el-avatar>
            </label>
            <!-- <p @click="$refs.file.click()">点击修改头像</p> -->
            <input
                id="file"
                type="file"
                hidden
                ref="file"
                @change="onFileChange"
            >
            </el-col>
        </el-row>
    </el-card>
    <!-- Dialog 打开动画结束时的回调 -->
    <el-dialog
      title="修改头像"
      :visible.sync="dialogVisible"
      append-to-body
      @opened="onDialogOpened"
      @closed="onDialogClosed"
    >
      <div class="preview-image-wrap">
        <img
        class="preview-image"
        :src="previewImage"
        ref="preview-image"
        >
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button
        type="primary"
        :loading="updatePhotoLoading"
        @click="onUpdatePhoto"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  getUserProfile,
  updateUserPhoto,
  updateUserProfile
} from '@/api/user'
import 'cropperjs/dist/cropper.css'
import Cropper from 'cropperjs'
import globalBus from '@/utils/global-bus'
export default {
  name: 'SettingsIndex',
  components: {},
  props: {},
  data () {
    return {

      user: {
        email: '',
        id: null,
        intro: '',
        mobile: '',
        name: '',
        photo: ''
      }, // 用户资料
      dialogVisible: false, // 弹层
      previewImage: '', // 预览图片
      cropper: null, // 裁切器实例
      updatePhotoLoading: false, // 更新用户头像 loading 状态
      updateProfileLoading: false, // 点击保存 loading 初始状态
      formRules: {
        name: [
          { required: true, message: '请输入媒体名称', trigger: 'blur' },
          { min: 5, max: 30, message: '长度在 5 到 30 个字符', trigger: 'blur' }
        ],
        intro: [
          { required: true, message: '请输入内容', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {},
  watch: {},
  created () {
    //   初始化数据
    this.loadUser()
  },
  mounted () {},
  methods: {
    // 更新用户信息
    onUpdateUser () {
      // 表单验证
      // 验证通过，提交表单
      // 开启 loading 状态
      this.updateProfileLoading = true
      updateUserProfile({
        name: this.user.name,
        intro: this.user.intro,
        email: this.user.email
      }).then(res => {
        this.$message({
          type: 'success',
          message: '保存成功'
        })
        this.updateProfileLoading = false
        // 更新头部当前登录用户的信息
        // 用户信息已修改发布通知
        globalBus.$emit('update-user', this.user)
      })
    },
    loadUser () {
      getUserProfile().then(res => {
        // console.log(res)
        this.user = res.data.data
      })
    },
    onFileChange () {
      console.log('file change')
      // 弹层显示
      this.dialogVisible = true
      // 处里图片预览
      const file = this.$refs.file

      const blobData = window.URL.createObjectURL(file.files[0])

      this.previewImage = blobData
      // 解决选择相同文件不触发 change 事件问题
      this.$refs.file.value = ''
    },
    onDialogOpened () {
      // 获取图片 DOM 节点
      const image = this.$refs['preview-image']
      // 第1次初始化好以后，如果预览裁切的图片发生了变化，裁切器默认不会更新
      // 如果需要预览图片发生变化更新裁切器：
      //    方式一：裁切器.replace 方法
      //    方式二：销毁裁切器，重新初始化
      // 初始化裁切器
      if (this.cropper) {
        this.cropper.replace(this.previewImage) // 替换剪裁图片的路径
        return
      }
      this.cropper = new Cropper(image, {
        // aspectRatio: 16 / 9,
        viewMode: 1,
        dragMode: 'none',
        aspectRatio: 1,
        cropBoxResizable: false
        // crop (event) {
        //   console.log(event.detail.x)
        //   console.log(event.detail.y)
        //   console.log(event.detail.width)
        //   console.log(event.detail.height)
        //   console.log(event.detail.rotate)
        //   console.log(event.detail.scaleX)
        //   console.log(event.detail.scaleY)
        // }
      })
    },
    onDialogClosed () {
      // 对话框关闭，销毁裁切器
      // this.cropper.destroy()
    },
    onUpdatePhoto () {
      // 点确定按钮开启loadding
      this.updatePhotoLoading = true
      // 获取裁切的图片对象
      this.cropper.getCroppedCanvas().toBlob(file => {
        const fd = new FormData()
        fd.append('photo', file)
        // 请求提交 fd
        updateUserPhoto(fd).then(res => {
          // 关闭对话框
          this.dialogVisible = false
          // 直接把裁切结果的文件对象转为 blob 数据本地预览
          this.user.photo = window.URL.createObjectURL(file)
          // 把服务端返回的图片进行展示有点慢
          // this.user.photo = res.data.data.photo
          // 关闭确定按钮的 loading
          this.updatePhotoLoading = false
          this.$message({
            type: 'success',
            message: '更新头像成功'
          })
          // 更新顶部登录用户的信息
          globalBus.$emit('update-user', this.user)
        })
      })
      // 请求更新用户头像
      // 关闭对话框
      // 更新视图展示
    }
  }
}
</script>

<style scoped lang="less">
.preview-image-wrap {
  /* Ensure the size of the image fit the container perfectly */
  .preview-image {
    display: block;

    /* This rule is very important, please don't ignore this */
    max-width: 100%;
    height: 200px;
  }
}
</style>
