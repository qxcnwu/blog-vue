<script lang="js">
// 引入 css
import '@wangeditor/editor/dist/css/style.css'

// 导入必要的包文件
import {onBeforeUnmount, ref, shallowRef, onMounted} from 'vue'
import {Editor, Toolbar} from '@wangeditor/editor-for-vue'
import attachmentModule from '@wangeditor/plugin-upload-attachment'
import {Boot} from '@wangeditor/editor'
import formulaModule from '@wangeditor/plugin-formula'
import axios from 'axios';

// 注册。要在创建编辑器之前注册，且只能注册一次，不可重复注册。
Boot.registerModule(formulaModule)
// 注册。要在创建编辑器之前注册，且只能注册一次，不可重复注册。
Boot.registerModule(attachmentModule)

export default {
  // 内部组件
  components: {Toolbar, Editor},
  // 启动函数
  setup() {
    // 编辑器实例，必须用 shallowRef
    const editorRef = shallowRef()

    // 内容 HTML
    const valueHtml = ref('<p>hello</p>')
    // 用户token
    const token = ref("")
    // 左侧列表
    const leftTitleList = ref([])
    // 当前标题
    const title = ref("")
    // 组件配置
    const toolbarConfig = {
      insertKeys: {
        index: 0,
        keys: [
          'insertFormula', // “插入公式”菜单
          'uploadAttachment',
          'editFormula' // “编辑公式”菜单
        ],
      },
    }
    // 编辑器配置
    const editorConfig = {
      placeholder: 'Type here...',
      scroll: false, // 禁止编辑器滚动
      // 在编辑器中，点击选中“附件”节点时，要弹出的菜单
      hoverbarKeys: {
        attachment: {
          menuKeys: ['downloadAttachment'], // “下载附件”菜单
        },
        formula: {
          menuKeys: ['editFormula'], // “编辑公式”菜单
        },
      },
      MENU_CONF: {
        // “上传附件”菜单的配置
        uploadAttachment: {
          server: 'http://127.0.0.1:8001/upload/attachment', // 服务端地址
          timeout: 5 * 1000, // 5s
          fieldName: 'attachment',
          meta: {
            "token": token
          }, // 请求时附加的数据
          metaWithUrl: false, // meta 拼接到 url 上
          headers: {},
          maxFileSize: 50 * 1024 * 1024, // 最大文件限制50M
        },
        uploadImage: {
          server: 'http://127.0.0.1:8001/upload/image',
          fieldName: 'image',
          // 小于该值就插入 base64 格式（而不上传），默认为 0
          base64LimitSize: 5 * 1024, // 5kb
          // 单个文件的最大体积限制，默认为 2M
          maxFileSize: 100 * 1024 * 1024,
          // 最多可上传几个文件，默认为 100
          maxNumberOfFiles: 10,
          // 选择文件时的类型限制，默认为 ['image/*'] 。如不想限制，则设置为 []
          allowedFileTypes: ['image/*'],
          // 自定义上传参数，例如传递验证的 token 等。参数会被添加到 formData 中，一起上传到服务端。
          meta: {
            "token": token
          }, // 请求时附加的数据
          // 将 meta 拼接到 url 参数中，默认 false
          metaWithUrl: false,
          // 自定义增加 http  header
          headers: {},
          // 跨域是否传递 cookie ，默认为 false
          withCredentials: true,
          // 超时时间，默认为 10 秒
          timeout: 5 * 1000, // 5 秒
        }
      }
    }
    // 当前是否可以编辑
    const canChange = ref(false)
    // 默认布局
    const mode = ref('default')

    // 组件销毁时，也及时销毁编辑器
    onBeforeUnmount(() => {
      const editor = editorRef.value
      if (editor == null) return
      editor.destroy()
    })
    onMounted(() => {
    })
    const handleCreated = (editor) => {
      editorRef.value = editor // 记录 editor 实例，重要！
    }

    return {
      editorRef,
      valueHtml,
      mode,
      toolbarConfig,
      editorConfig,
      handleCreated,
      token,
      leftTitleList,
      title,
      canChange
    };
  },
  // 自定义方法
  methods: {
    // 点击左侧空间出现的活动
    handleNodeClick() {

    },
    // 获取当前空间下所有文章
    getLeftList(current_title, node) {
      const url = "http://127.0.0.1:8001/channel/get/" + current_title;
      axios.get(url).then(response => {
        if (response.data["errno"] === 0) {
          // 当前成功获取那么需要更新当前节点
          response.data["data"].forEach(function (item) {
            console.log("node:" + node)
            node.children.push({
              label: item.title,
              sonidx: item.sonidx,
              children: []
            })
          })
          console.log(node)
        } else if (response.data["errno"] === -1) {
          // todo 弹出登录页面

        }
      })
    },
    initButton(root, token) {
      console.log(token)
      console.log(root)
      this.getLeftList("base", root)
      console.log(this.root)
    },
  },
  // 全局数据
  data() {
    return {
      // 当前用户的信息
      token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MTc5NDU4NTgsImlhdCI6MTcxNzk0NTg1OCwidXNlcm5hbWUiOiJxeGMifQ.WeldEIXUXl1H_o2xUrvEy-zgbf1WS7gT1Dd-NeF9Tj8",
      root: [{label: "基础空间", sonidx: "123123", children: []}],
      // root: [
      //   {
      //     label: 'Level one 1',
      //     children: [
      //       {
      //         label: 'Level two 1-1',
      //         children: [
      //           {
      //             label: 'Level three 1-1-1',
      //           },
      //         ],
      //       },
      //     ],
      //   },
      //   {
      //     label: 'Level one 2',
      //     children: [
      //       {
      //         label: 'Level two 2-1',
      //         children: [
      //           {
      //             label: 'Level three 2-1-1',
      //           },
      //         ],
      //       },
      //       {
      //         label: 'Level two 2-2',
      //         children: [
      //           {
      //             label: 'Level three 2-2-1',
      //           },
      //         ],
      //       },
      //     ],
      //   },
      //   {
      //     label: 'Level one 3',
      //     children: [
      //       {
      //         label: 'Level two 3-1',
      //         children: [
      //           {
      //             label: 'Level three 3-1-1',
      //           },
      //         ],
      //       },
      //       {
      //         label: 'Level two 3-2',
      //         children: [
      //           {
      //             label: 'Level three 3-2-1',
      //           },
      //         ],
      //       },
      //     ],
      //   },
      // ],
      defaultProps: {
        children: 'children',
        label: 'label',
      }
    }
  },
}
</script>


<template>
  <el-container>
    <el-header style="background-color: #222222">Header</el-header>
    <el-container>
      <el-aside id="el-main-aside" width=15% style="background-color: rgb(238, 241, 246); overflow-y: auto">
        <el-row style="height: 10%">
          <el-col>
            qweqeqwweq
          </el-col>
        </el-row>
        <el-row id="el-aside-tree">
          <el-col :span="24">
            <el-tree
                style="max-width: 600px"
                :data="root"
                :props="defaultProps"
                @node-click="handleNodeClick"
            />
          </el-col>
        </el-row>
      </el-aside>
      <el-main id="el-main-blog" style="height: 90%;overflow-y: auto;padding: 0">
        <div>
          <el-button @click="initButton(root[0],token)">
            test
          </el-button>
        </div>
        <div style="border-bottom: 1px solid #e8e8e8;">
          <div id="editor-toolbar">
            <Toolbar
                style="border-bottom: 1px solid #ccc"
                :editor="editorRef"
                :defaultConfig="toolbarConfig"
                :mode="mode"
            />
          </div>
        </div>
        <div id="content">
          <div id="editor-container">
            <div id="title-container">
              <input v-model="title" placeholder="Page title...">
            </div>
            <div id="editor-text-area">
              <Editor
                  style="height: 70vh;overflow-y: auto;"
                  v-model="valueHtml"
                  :defaultConfig="editorConfig"
                  :mode="mode"
                  @onCreated="handleCreated"
              />
            </div>
          </div>
        </div>
      </el-main>
    </el-container>
  </el-container>
</template>

<style>
#el-main-blog {
  position: absolute;
  left: 15%;
  right: 0;
  top: 60px;
  bottom: 0;
  overflow-y: scroll;
}

#el-main-aside {
  position: absolute;
  left: 0;
  right: 0;
  top: 60px;
  bottom: 0;
  overflow-y: scroll;
  height: calc(100vh - 250px);
}

#el-aside-tree {
  height: 100%
}

::-webkit-scrollbar {
  display: none;
}

#editor-toolbar {
  width: 80%;
  height: 10%;
  background-color: #FCFCFC;
  margin: 0 auto;
}

#content {
  height: calc(100vh - 250px);
  width: 100%;
  background-color: rgb(245, 245, 245);
  overflow-y: auto;
  position: relative;
}

#editor-container {
  width: 80%;
  margin: 20px auto 0px auto;
  background-color: #fff;
  padding: 20px 50px 0px 50px;
  border: 1px solid #e8e8e8;
  box-shadow: 0 2px 10px rgb(0 0 0 / 12%);
}

#title-container {
  padding: 20px 0;
  border-bottom: 1px solid #e8e8e8;
}

#title-container input {
  font-size: 30px;
  border: 0;
  outline: none;
  width: 100%;
  line-height: 1;
}

#editor-text-area {
  min-height: 900px;
  margin-top: 20px;
}
</style>
