> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7020205053147349000) npm install vue-cropper 复制代码

### 使用

```
import { VueCropper } from 'vue-cropper'
复制代码
```

### 代码实现

以 element-ui + vue-cropper 为例实现头像裁剪

src/App.vue

```
<template>
  <div>
    <el-button @click="dialogVisible = true">上传头像</el-button>
    <avatar-cropper :dialogVisible.sync="dialogVisible" @closeAvatarDialog="closeAvatarDialog"></avatar-cropper>
  </div>
</template>

<script>
  import avatarCropper from '@/components/avatarCropper'
  export default {
    components: {
      avatarCropper
    },
    data() {
      return {
        dialogVisible: false
      }
    },
    methods: {
      closeAvatarDialog(data) {
	console.log(data)
        this.dialogVisible = false
      }
    }
  }
</script>
复制代码
```

src/components/avatarCropper.vue

```
<template>
  <el-dialog
    title="裁剪头像"
    :visible.sync="dialogVisible"
    :show-close="false"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    @close="closeDialog"
    width="600px"
  >
    <div class="avatar-container">
      <!-- 待上传图片 -->
      <div v-show="!options.img">
        <el-upload
          class="upload"
          ref="upload"
          action=""
          :on-change="upload"
          accept="image/png, image/jpeg, image/jpg"
          :show-file-list="false"
          :auto-upload="false"
        >
          <el-button slot="trigger" size="small" type="primary" ref="uploadBtn">
            选择图片
          </el-button>
        </el-upload>
        <div>支持jpg、png格式的图片，大小不超过5M</div>
      </div>
      <!-- 已上传图片 -->
      <div v-show="options.img" class="avatar-crop">
        <vueCropper
          v-if="dialogVisible"
          class="crop-box"
          ref="cropper"
          :img="options.img"
          :autoCrop="options.autoCrop"
          :fixedBox="options.fixedBox"
          :canMoveBox="options.canMoveBox"
          :autoCropWidth="options.autoCropWidth"
          :autoCropHeight="options.autoCropHeight"
          :centerBox="options.centerBox"
          :fixed="options.fixed"
          :fixedNumber="options.fixedNumber"
          :canMove="options.canMove"
          :canScale="options.canScale"
        ></vueCropper>
      </div>
    </div>
    <span slot="footer" class="dialog-footer">
      <div class="reupload" @click="reupload">
        <span v-show="options.img">重新上传</span>
      </div>
      <div>
        <el-button @click="closeDialog">取 消</el-button>
        <el-button type="primary" @click="getCrop">确 定</el-button>
      </div>
    </span>
  </el-dialog>
</template>

<script>
import { VueCropper } from 'vue-cropper'
export default {
  components: {
    VueCropper
  },
  name: 'avatarCropper',
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // vueCropper组件 裁剪配置信息
      options: {
        img: '', // 原图文件
        autoCrop: true, // 默认生成截图框
        fixedBox: false, // 固定截图框大小
        canMoveBox: true, // 截图框可以拖动
        autoCropWidth: 200, // 截图框宽度
        autoCropHeight: 200, // 截图框高度
        fixed: true, // 截图框宽高固定比例
        fixedNumber: [1, 1], // 截图框的宽高比例
        centerBox: true, // 截图框被限制在图片里面
        canMove: false, // 上传图片不允许拖动
        canScale: false // 上传图片不允许滚轮缩放
      }
    }
  },

  methods: {
    // 读取原图
    upload(file) {
      const isIMAGE = file.raw.type === 'image/jpeg' || file.raw.type === 'image/png'
      const isLt5M = file.raw.size / 1024 / 1024 < 5
      if (!isIMAGE) {
        this.$message({
          showClose: true,
          message: '请选择 jpg、png 格式的图片',
          type: 'warning'
        })
        return false
      }
      if (!isLt5M) {
        this.$message({
          showClose: true,
          message: '图片大小不能超过 5MB',
          type: 'warning'
        })
        return false
      }
      let reader = new FileReader()
      reader.readAsDataURL(file.raw)
      reader.onload = e => {
        this.options.img = e.target.result // base64
      }
    },
    // 获取截图信息
    getCrop() {
      // 获取截图的 base64 数据
      // this.$refs.cropper.getCropData((data) => {
      //   this.$emit('closeAvatarDialog', data)
      //   this.closeDialog()
      // });
      // 获取截图的 blob 数据
      this.$refs.cropper.getCropBlob(data => {
        this.$emit('closeAvatarDialog', data)
        this.closeDialog()
      })
    },
    // 重新上传
    reupload() {
      this.$refs.uploadBtn.$el.click()
    },
    // 关闭弹框
    closeDialog() {
      this.$emit('update:dialogVisible', false)
      this.options.img = ''
    }
  }
}
</script>

<style lang="scss" scoped>
.dialog-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  .reupload {
    color: #409eff;
    cursor: pointer;
  }
}
.avatar-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 560px;
  height: 400px;
  background-color: #f0f2f5;
  margin-right: 10px;
  border-radius: 4px;
  .upload {
    text-align: center;
    margin-bottom: 24px;
  }
  .avatar-crop {
    width: 560px;
    height: 400px;
    position: relative;
    .crop-box {
      width: 100%;
      height: 100%;
      border-radius: 4px;
      overflow: hidden;
    }
  }
}
</style>
复制代码
```

### 总结

裁剪完成之后可以获取到 base64 和 blob 数据，然后上传至后端。vue-cropper 还有众多属性和方法，用起来都很方便，有兴趣的同学可以实现一下实时预览。

文档地址：[github.com/xyxiao001/v…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxyxiao001%2Fvue-cropper "https://github.com/xyxiao001/vue-cropper")

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d875019d2b114f47af46a95c96bd61a7~tplv-k3u1fbpfcp-watermark.awebp?) ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2ad45fb75f04077b74bb86759c1b0a3~tplv-k3u1fbpfcp-watermark.awebp?)

欢迎在评论区留言，掘金官方将在[掘力星计划](https://juejin.cn/post/7012210233804079141 "https://juejin.cn/post/7012210233804079141")活动结束后，在评论区抽送 100 份掘金周边。如果文章对你有所帮助，不要忘了点个赞呦~

听说点赞的来年升职加薪，爱情事业双丰收😃