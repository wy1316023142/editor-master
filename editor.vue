<template>
  <div class="wang" :id="id"></div>
</template>

<script>
  import WangEditor from 'wangeditor'

  export default {
    name: '',
    props: {
      value: {
        type: String
      },
      id: {
        type: String,
        default: 'editor'
      },
      disabled: {
        type: Boolean,
        default: false
      },
      menus: {
        type: Array
      }
    },
    data () {
      return {
        editor: null,
        editorContent: ''
      }
    },
    computed: {
      baseUrl () {
        return this.config.ossUrl
      },
      baseApi () {
        return this.$store.getters.gApiUrl
      },
      accessInfo () {
        return this.$store.state.accessInfo
      }
    },
    watch: {
      value (newValue, oldValue) {
        if (this.editorContent !== newValue) {
          this.editor.txt.html(this.escape2Html(newValue))
        }
      },
      disabled (value) {
        this.editor.$textElem.attr('contenteditable', !value)
        this.editor.$toolbarElem[0].style.display = value ? 'none' : 'flex'
      }
    },
    mounted () {
      const editor = new WangEditor('#' + this.id)
      // editor.customConfig.uploadImgServer = this.accessInfo.host
      // editor.customConfig.uploadImgMaxLength = 1
      editor.customConfig.zIndex = 10
      // editor.customConfig.uploadImgMaxSize = 5 * 1024 * 1024
      // editor.customConfig.withCredentials = true
      // editor.customConfig.uploadFileName = 'file'
      // editor.customConfig.uploadImgParams = this.accessInfo
      if (this.menus) {
        editor.customConfig.menus = this.menus
      }
      let _this = this
      editor.customConfig.customUploadImg = async function (files, insert) {
        // files 是 input 中选中的文件列表
        // insert 是获取图片 url 后，插入到编辑器的方法
        let xhr = new XMLHttpRequest()
        var formData = new FormData()
    /* if (files.length > 1) {
          _this.$message.error('一次只能上传一张图片')
          return
        }
    */
        if (!/\.(jpg|jpeg|png|bmp|gif)$/i.test(files[0].name)) {
          // 后缀名不合法，不是图片
          _this.$message.error('请上传图片文件')
          return
        }
        if (files[0].size > 1024 * 1024 * 1) {
          // 上传图片过大
          _this.$message.error('图片文件过大')
          return
        }
        let imgUrl = ''
        let imgLoad = new Promise((resolve, reject) => {
          var reader = new FileReader()
          reader.readAsDataURL(files[0])
          reader.onload = function () {
            var img = new Image()
            console.log(this)
            img.src = this.result
            img.onload = function () {
              resolve(this)
              console.log(`width=${this.width},height=${this.height}`)
              if (this.width > 1230 || this.width < 600 || this.height > 960) {
                _this.$message.error('商品图片尺寸宽度在600px至1230px之间,高度不超过960px')
                reject(new Error('商品图片尺寸宽度在600px至1230px之间,高度不超过960px'))
              } else {
                imgUrl = this.src
              }
            }
            img.onerror = function (err) {
              reject(err)
            }
          }
        })
        await imgLoad
        // 添加签名
        let access = Object.deepAssign({}, _this.accessInfo)
        access.key = _this.accessInfo.key + files[0].name
        for (let key in access) {
          formData.append(key, access[key])
        }
        // 
   /* 添加文件 单张图片
        files.forEach(file => {
          formData.append('file', file)
        })
        xhr.open('POST', _this.accessInfo.host)
        xhr.onreadystatechange = () => {
          // 阿里云返回201
          if (xhr.readyState === 4 && xhr.status === 201 && imgUrl) {
            insert(xhr.responseXML.querySelector('Location').textContent)
          } else if (xhr.readyState === 4 && xhr.status !== 201) {
            _this.$message.error('未知错误，状态码：' + xhr.status)
          }
        }
         xhr.send(formData)
       }
     */
    // 添加文件 多张图片 利用回调的方式
        ;(function iterator (i) {
          if (i === files.length) {
            return
          }
          let name = files[i].name.split('.')[0] + '_' + Math.round(new Date().getTime() / 1000) + '.' + files[i].name.split('.')[1]
          // 异步请求
          formData.set('file', files[i], name)
          formData.set('key', _this.accessInfo.key + name)
          xhr.open('POST', _this.accessInfo.host)
          xhr.onreadystatechange = () => {
            // 阿里云返回201
            if (xhr.readyState === 4 && xhr.status === 201 && imgUrl) {
              insert(xhr.responseXML.querySelector('Location').textContent)
              setTimeout(() => {
                iterator(i + 1)
              }, 500)
            } else if (xhr.readyState === 4 && xhr.status !== 201) {
              _this.$message.error('未知错误，状态码：' + xhr.status)
            }
          }
          xhr.send(formData)
        })(0)
      }
  // 打印出错误
      editor.customConfig.customAlert = info => {
        this.$message.error(info)
      }
      editor.customConfig.showLinkImg = false
 /* 回调即插入图片的地址
      editor.customConfig.linkImgCallback = function (url) {}
      editor.customConfig.linkImgCheck = function (src) {
         console.log(src) // 图片的链接
         var img2 = new Image()
         img2.src = src
         img2.onload = function () {
           if (img2.width > 1230 || img2.width < 600 || img2.height > 960) {
             _this.$message.error('商品图片尺寸宽度在600px至1230px之间,高度不超过960px')
             return '' // 返回字符串，即校验失败的提示信息
           }
         }
      }
 */
      let timer = null
      // editor.customConfig.uploadImgShowBase64 = true // 使用 base64 保存图片
      editor.customConfig.onchange = html => {
        this.editorContent = html
        clearTimeout(timer)
        timer = setTimeout(() => {
          if (html === '<p><br></p>') {
            html = ''
          }
          this.$emit('input', html)
        }, 100)
      }
      editor.create()
      if (this.value) {
        editor.txt.html(this.escape2Html(this.value))
      }
      this.editor = editor
    },
    methods: {
      escape2Html (str) {
        var arrEntities = {'lt': '<', 'gt': '>', 'nbsp': ' ', 'amp': '&', 'quot': '"'}
        return str.replace(/&(lt|gt|nbsp|amp|quot);/ig, (all, t) => {
          return arrEntities[t]
        })
      }
    }
  }
</script>

<style>
  .wang {
    border-top: 1px solid #ccc;
  }
  .w-e-toolbar {
    margin-top: -1px;
  }
  .w-e-text-container {
    z-index: 100;
  }
  .w-e-toolbar .w-e-menu {
    z-index: 101;
  }
</style>
