> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7017957779176423454) 是不是比原生的滚动条美观多了，使用方法也非常简单：

```
<el-scrollbar>
  <div class="box">
    <p v-for="item in 15" :key="item">欢迎使用 el-scrollbar {{item}}</p>
  </div>
</el-scrollbar>

<style scoped>
.el-scrollbar {
  border: 1px solid #ddd;
  height: 200px;
}
.el-scrollbar ::v-deep  .el-scrollbar__wrap {
    overflow-y: scroll;
    overflow-x: hidden;
  }
</style>
复制代码
```

只要 scrollbar 内部盒子的高度超过 scrollbar 的高度就会出现滚动条，横向滚动条同理。

### el-upload 模拟点击

有时候我们想用 el-upload 的上传功能，但又不想用 el-upload 的样式，如何实现呢？方法也很简单，隐藏 el-upload，然后再模拟点击就可以了。

```
<button @click="handleUpload">上传文件</button>
<el-upload
  v-show="false"
  class="upload-resource"
  multiple
  action=""
  :http-request="clickUploadFile"
  ref="upload"
  :on-success="uploadSuccess"
>
    上传本地文件
</el-upload>

<script>
export default {
    methods: {
        // 模拟点击
        handleUpload() {
            document.querySelector(".upload-resource .el-upload").click()
        },
        // 上传文件
        async clickUploadFile(file) {
            const formData = new FormData()
            formData.append('file', file.file)
            const res = await api.post(`xxx`, formData)
        }
        // 上传成功后，清空组件自带的文件列表
        uploadSuccess() {
            this.$refs.upload.clearFiles()
        }
    }
}
</script>
复制代码
```

### el-select 下拉框选项过长

很多时候下拉框的内容是不可控的，如果下拉框选项内容过长，势必会导致页面非常不协调，解决办法就是，单行省略加文字提示。

```
<el-select popper-class="popper-class" :popper-append-to-body="false" v-model="value" placeholder="请选择">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value"
    >
        <el-tooltip
          placement="top"
          :disabled="item.label.length<17"
        >
            <div slot="content">
                <span>{{item.label}}</span>
            </div>
            <div class="iclass-text-ellipsis">{{ item.label }}</div>
        </el-tooltip>
    </el-option>
</el-select>

<script>
  export default {
    data() {
      return {
        options: [{
          value: '选项1',
          label: '黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕'
        }, {
          value: '选项2',
          label: '双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶'
        }, {
          value: '选项3',
          label: '蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎'
        }],
        value: ''
      }
    }
  }
</script>

<style scoped>
.el-select {
  width: 300px;
}
.el-select ::v-deep .popper-class {
  width: 300px;
}
.iclass-text-ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
复制代码
```

效果如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c87b9654ff484f46a6a3a176f0c2c347~tplv-k3u1fbpfcp-watermark.awebp?)

### el-input 首尾不能为空格

我们在使用 input 输入框时，大多不希望用户在前后输入空格，有没有简单的校验方法呢，当然是有的。

```
<el-form :rules="rules" :model="form" label-width="80px">
    <el-form-item label="活动名称" prop="name">
        <el-input v-model="form.name"></el-input>
    </el-form-item>
</el-form>

<script>
export default {
    data() {
        return {
            form: {
                name: ''
            },
            rules: {
                name: [
                        { required: true, message: '请输入活动名称', trigger: 'blur'},
                        { pattern: /^(?!\s+).*(?<!\s)$/,  message: '首尾不能为空格', trigger: 'blur' }
                ]
            }
        }
    }
}
</script>
复制代码
```

效果如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9f5694eca6b456e988f6c5880654f1a~tplv-k3u1fbpfcp-watermark.awebp?)

### el-input type=number 输入中文，焦点上移

当 el-input 设置 type="number" 时，输入中文，虽然中文不会显示出来，但焦点会上移。 ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b114ec274264869a12d97e35b9ae47e~tplv-k3u1fbpfcp-watermark.awebp?) 解决办法：

```
<style scoped>
::v-deep .el-input__inner {
    line-height: 1px !important;
}
</style>
复制代码
```

### el-input type=number 去除聚焦时的上下箭头

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d715302792b43258c253cfa449e14b7~tplv-k3u1fbpfcp-watermark.awebp?) 解决办法：

```
<el-input class="clear-number-input" type="number"></el-input>

<style scoped>
.clear-number-input ::v-deep input[type="number"]::-webkit-outer-spin-button,
.clear-number-input ::v-deep input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none !important;
}
</style>
复制代码
```

### el-form 只校验表单其中一个字段

有时候我们需要单独校验一些字段，比如发送验证码，单独对手机号进行校验，可以这样做：

```
this.$refs.form.validateField('name', valid => {
    if (valid) { 
        console.log('send!'); 
    } else { 
        console.log('error send!'); 
        return false; 
    }
})
复制代码
```

### el-dialog 重新打开弹窗，清除表单信息

有人会在打开弹窗时，在 $nextTick 里重置表单，而我选择在关闭弹窗后进行重置：

```
<el-dialog @closed="resetForm">
    <el-form ref="form">
    </el-form>
</el-dialog>

<script>
export default {
    methods: {
        resetForm() {
            this.$refs.form.resetFields()
        }
    }
}
</script>
复制代码
```

### el-dialog 的 destroy-on-close 属性设置无效

destroy-on-close 设置为 true 后发现弹窗关闭后 DOM 元素仍在，没有被销毁。

解决办法：在 el-dialog 上添加 v-if。

```
<el-dialog :visible.sync="visible" v-if="visible" destroy-on-close>
</el-dialog>
复制代码
```

### el-table 表格内容超出省略

当表格内容过长时，手动添加样式比较麻烦，偷偷告诉你，只需要添加一个 show-overflow-tooltip 就可以搞定。

```
<el-table-column
  prop="address"
  label="地址"
  width="180"
  show-overflow-tooltip
>
</el-table-column>
复制代码
```

效果如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c41012695f0c43d6a6f0e5df018fc2cf~tplv-k3u1fbpfcp-watermark.awebp?)

欢迎在评论区留言，掘金官方将在[掘力星计划](https://juejin.cn/post/7012210233804079141 "https://juejin.cn/post/7012210233804079141")活动结束后，在评论区抽送 100 份掘金周边。如果文章对你有所帮助，不要忘了点个赞~

听说点赞的今年桃花运贼旺😍