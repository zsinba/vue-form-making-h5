<template>
  <el-container>
    <el-aside style="wdith: 250px;">

      <div class="components-list">
        <div class="widget-cate">基础字段</div>
        <draggable element="ul" :list="basicComponents" 
          :options="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
          @end="handleMoveEnd"
          @start="handleMoveStart"
          :move="handleMove"
        >
          
          <li class="form-edit-widget-label" v-for="(item, index) in basicComponents" :key="index">
            <a>
              <icon class="icon" :name="item.icon"></icon>
              <span>{{item.name}}</span>
            </a>
          </li>
        </draggable>

        <div class="widget-cate">高级字段</div>
        <draggable element="ul" :list="advanceComponents" 
          :options="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
          @end="handleMoveEnd"
          @start="handleMoveStart"
          :move="handleMove"
        >
          
          <li class="form-edit-widget-label" v-for="(item, index) in advanceComponents" :key="index">
            <a>
              <icon class="icon" :name="item.icon"></icon>
              <span>{{item.name}}</span>
            </a>
          </li>
        </draggable>
        
        <div class="widget-cate">布局字段</div>
        <draggable element="ul" :list="layoutComponents" 
          :options="{group:{ name:'people', pull:'clone',put:false},sort:false, ghostClass: 'ghost'}"
          @end="handleMoveEnd"
          @start="handleMoveStart"
          :move="handleMove"
        >
          
          <li class="form-edit-widget-label data-grid" v-for="(item, index) in layoutComponents" :key="index">
            <a>
              <icon class="icon" :name="item.icon"></icon>
              <span>{{item.name}}</span>
            </a>
          </li>
        </draggable>
      </div>
      
    </el-aside>
    <el-container class="center-container" direction="vertical">
      <el-header class="btn-bar" style="height: 45px;">
        <!-- <el-button type="text" size="medium" @click="handleGoGithub">GitHub</el-button> -->
        <el-button type="text" size="medium" icon="el-icon-upload2" @click="handleGenerateJson2">导入JSON</el-button>
        <el-button type="text" size="medium" icon="el-icon-view" @click="handlePreview">PC预览</el-button>
        <el-button type="text" size="medium" icon="el-icon-view" @click="handlePreviewH5">H5预览</el-button>
        <el-button type="text" size="medium" icon="el-icon-tickets" @click="handleGenerateJson">生成JSON</el-button>
        <el-button type="text" size="medium" icon="el-icon-document" @click="handleGenerateCode">生成代码</el-button>
      </el-header>
      <el-main :class="{'widget-empty': widgetForm.list.length == 0}">
        
        <widget-form ref="widgetForm" :data="widgetForm" :select.sync="widgetFormSelect"></widget-form>
      </el-main>
    </el-container>
    
    <el-aside class="widget-config-container">
      <el-container>
        <el-header height="45px">
          <div class="config-tab" :class="{active: configTab=='widget'}" @click="handleConfigSelect('widget')">字段属性</div>
          <div class="config-tab" :class="{active: configTab=='form'}" @click="handleConfigSelect('form')">表单属性</div>
        </el-header>
        <el-main class="config-content">
          <widget-config v-show="configTab=='widget'" :data="widgetFormSelect"></widget-config>
          <form-config v-show="configTab=='form'" :data="widgetForm.config"></form-config>
        </el-main>
      </el-container>
      
    </el-aside>
    
    <!-- PC预览 -->
    <cus-dialog
      :visible="previewVisible"
      @on-close="previewVisible = false"
      ref="widgetPreview"
      @on-submit="handleTest"
      width="1000px"
      form
    >
      <generate-form v-if="previewVisible" :data="widgetForm" :remote="remoteFuncs" :value="widgetModels" ref="generateForm">

        <template slot="blank" slot-scope="scope">
          宽度：<el-input v-model="scope.model.blank.width" style="width: 100px"></el-input>
          高度：<el-input v-model="scope.model.blank.height" style="width: 100px"></el-input>
        </template>
      </generate-form>
    </cus-dialog>

    <!-- H5预览 -->
    <cus-dialog
      :visible="previewH5Visible"
      @on-close="previewH5Visible = false"
      ref="widgetPreview"
      @on-submit="handleTest"
      width="425px"
      form
      title="H5初步预览效果"
    > 
      <p>注：如果想获得更真实的预览效果，请通过路由访问预览，并切换移动端模拟器</p>
      <fm-generate-form-h5 v-if="previewH5Visible" :data="widgetForm" :remote="remoteFuncs" :value="widgetModels" ref="generateForm">

        <!-- <template slot="blank" slot-scope="scope">
          宽度：<el-input v-model="scope.model.blank.width" style="width: 100px"></el-input>
          高度：<el-input v-model="scope.model.blank.height" style="width: 100px"></el-input>
        </template> -->
      </fm-generate-form-h5>
    </cus-dialog>

    <cus-dialog
      :visible="jsonVisible"
      @on-close="jsonVisible = false"
      ref="jsonPreview"
      width="800px"
      form
    >
      <div id="jsoneditor" style="height: 400px;width: 100%;">{{jsonTemplate}}</div>
      
      <template slot="action">
        <el-button id="copybtn" data-clipboard-target=".ace_text-input">双击复制</el-button>
      </template>
    </cus-dialog>
    <!-- 导入json -->
    <cus-dialog
      :visible="importJsonVisible"
      @on-close="importJsonVisible = false"
      ref="jsonPreview"
      width="800px"
      form
    > 
      <p>JSON格式如下，直接复制生成的json覆盖此处代码点击确定即可</p>
      <div id="jsoneditor2" style="height: 400px;width: 100%;">{{importJsonTemplate}}</div>
      
      <template slot="action">
        <el-button @click="setImportJson">确定</el-button>
      </template>
    </cus-dialog>

    <cus-dialog
      :visible="codeVisible"
      @on-close="codeVisible = false"
      ref="codePreview"
      width="800px"
      form
      :action="false"
    >
      <div id="codeeditor" style="height: 500px; width: 100%;">{{htmlTemplate}}</div>
    </cus-dialog>
  </el-container>
</template>

<script>
import Draggable from 'vuedraggable'
import WidgetConfig from './WidgetConfig'
import FormConfig from './FormConfig'
import WidgetForm from './WidgetForm'
import CusDialog from './CusDialog'
import GenerateForm from './GenerateForm'
// import JSONEditor from 'jsoneditor'
// import 'jsoneditor/dist/jsoneditor.min.css'
import Clipboard from 'clipboard'
import {basicComponents, layoutComponents, advanceComponents} from './componentsConfig.js'
import {loadJs, loadCss} from '../util/index.js'
import request from '../util/request.js'

//return html，生成html代码
import generateCode from './generateCode.js'

/*设计器模板fm-making-form组件注册*/
export default {
  name: 'fm-making-form',
  components: {
    Draggable,
    WidgetConfig,
    FormConfig,
    WidgetForm,
    CusDialog,
    GenerateForm
  },
  data () {
    return {
      basicComponents,
      layoutComponents,
      advanceComponents,
      widgetForm: {
        list: [],
        config: {
          labelWidth: 100,
          labelPosition: 'top'
        },
      },
      configTab: 'widget',
      widgetFormSelect: null,
      previewVisible: false,
      previewH5Visible: false,
      jsonVisible: false,
      importJsonVisible: false,
      codeVisible: false,
      remoteFuncs: {
        func_test (resolve) {
          setTimeout(() => {
            const options = [
              {id: '1', name: '1111'},
              {id: '2', name: '2222'},
              {id: '3', name: '3333'}
            ]

            resolve(options)
          }, 2000)
        },
        funcGetToken (resolve) {
          request.get('http://tools-server.xiaoyaoji.cn/api/uptoken').then(res => {
            resolve(res.uptoken)
          })
        }
      },
      widgetModels: {},
      blank: '',
      htmlTemplate: '',
      jsonTemplate: '',
      importJsonTemplate: {"list":[{"type":"input","name":"单行文本","icon":"icon-input","options":{"width":"100%","defaultValue":"","required":false,"dataType":"string","pattern":"","placeholder":"","remoteFunc":"func_1540908864000_94322"},"key":"1540908864000_94322","model":"input_1540908864000_94322","rules":[{"type":"string","message":"单行文本格式不正确"}]},{"type":"textarea","name":"多行文本","icon":"icon-diy-com-textarea","options":{"width":"100%","defaultValue":"","required":false,"pattern":"","placeholder":"","remoteFunc":"func_1540908876000_19435"},"key":"1540908876000_19435","model":"textarea_1540908876000_19435","rules":[]}],"config":{"labelWidth":100,"labelPosition":"top","size":"small"}}
    }
  },
  mounted () {
    // loadCss('https://unpkg.com/jsoneditor/dist/jsoneditor.min.css')
    // loadJs('https://unpkg.com/jsoneditor/dist/jsoneditor.min.js')
    loadJs('lib/ace/src/ace.js')
  },
  methods: {
    handleGoGithub () {
      window.location.href = 'https://github.com/GavinZhuLei/vue-form-making'
    },
    handleConfigSelect (value) {
      this.configTab = value
    },
    handleMoveEnd (evt) {
      console.log('end', evt)
    },
    handleMoveStart ({oldIndex}) {
      console.log('start', oldIndex, this.basicComponents)
    },
    handleMove () {
      return true
    },
    handlePreview () {
      this.previewVisible = true
    },
    handlePreviewH5 () {
      this.previewH5Visible = true
    },
    handleTest () {
      this.$refs.generateForm.getData().then(data => {
        this.$alert(data, '').catch(e=>{})
        this.$refs.widgetPreview.end()
      }).catch(e => {
        this.$refs.widgetPreview.end()
      })
    },
    handleGenerateJson () {
      this.jsonVisible = true
      this.jsonTemplate = this.widgetForm
      console.log('----json----',this.widgetForm)
      this.$nextTick(() => {

        const editor = ace.edit('jsoneditor')
        editor.session.setMode("ace/mode/json")

        const btnCopy = new Clipboard('#copybtn')
      })
    },
    handleGenerateJson2 () {
      this.importJsonVisible = true
      //this.importJsonTemplate = this.widgetForm
      console.log('----json----',this.widgetForm)
      this.$nextTick(() => { 

        const editor = ace.edit('jsoneditor2')
        editor.session.setMode("ace/mode/json")

        //const btnCopy = new Clipboard('#copybtn')
      })
    },
    setImportJson (){//debugger
      console.log(this.widgetForm)
      const editor = ace.edit('jsoneditor2')
            editor.session.setMode("ace/mode/json")
      let _importJsonTemplate = editor.getValue()
      console.log('获取json',JSON.parse(_importJsonTemplate)) 

      this.widgetForm = JSON.parse(_importJsonTemplate)

      this.importJsonVisible = false
    },
    handleGenerateCode () {
      this.codeVisible = true
      //widgetForm表单json转成字符串传进去，再解析
      this.htmlTemplate = generateCode(JSON.stringify(this.widgetForm))
      this.$nextTick(() => {
        const editor = ace.edit('codeeditor')
        editor.session.setMode("ace/mode/html")
      })
    }
  },
  watch: {
    widgetForm: {
      deep: true,
      handler: function (val) {
        console.log(this.$refs.widgetForm)
      }
    }
  }
}
</script>

<style lang="scss">
@import '../styles/cover.scss';
@import '../styles/index.scss';

.widget-empty{
  background: url('../assets/form_empty.png') no-repeat;
  background-position: 50%;
}

</style>
