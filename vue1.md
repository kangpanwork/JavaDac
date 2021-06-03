
#### 页面格式
```
<template>这里写HTML元素</template>
<script>
import 组件 from '文件路径'
export default {
 components:{组件引入},
 computed:{计算属性},
 watch:{监听},
 data(){return {属性定义及初始化}},
 created() {实例创建完之后的钩子函数},
 mounted(){页面完成后执行的钩子函数，实时监控数据，数据变化实时更新DOM},
 destroyed(){实例销毁执行的钩子函数},
 methods:{函数}
 }
 </script>
 <style scoped>样式</style>
 
}
</script>
```
#### 父子传值及全局事件
父组件中引入子组件，例如：
```
<template>
    <div>
        <子组件 :value='值' ref='引用' @function='事件' ></子组件>
    </div>
</template>
<script>
import 子组件 from '子组件路径'
export default {
    components:{
        子组件：子组件
    }
}
</script>
```
`function`事件是子组件发送过来的，通过 `this.$emit('function',传值)`
`value`属性是父组件传值给子组件，在子组件接收
```
export default {
 props: {
     value: {
         type:Boolean,
         default:function(){
             return false
         }
    }
 }
}
```
`ref` 引用，父组件可以通过子组件的引用调子组件的方法
`const data = this.$refs['引用'].getData()`
全局事件发送及接收，事件`ID`必须唯一
```
this.$evenBus.$emit(id,value)
this.$evenBus.$on(id,value => {do something})
```
#### 时间格式化
`new Date(window._Hae.Date.format(时间,'yyyy/MM/dd hh:mm:ss')).valueOf()`
#### map
```
let newList = this.list.map(ele => {
    return {
        新属性:ele.属性
    }
})
```
#### 路由跳转
```
this.$router.push({
    path:'路径',
    query:{属性:属性值}
}）
```
#### 缓存
```
window.sessinStorage.setItem(key,value)
JSON.parse(window.sessionStorage.getItem(key))
window.sessionStorage.removeItem(key)
```
#### 服务请求
```
const url = '请求URL'
const params = {'属性':属性值}
this.$service.network.post(url,params).then(res = > {
    if(res.data.resultCode === 200){do something}
    else{do something}
}).catch(() => {异常处理})

```
#### 异步请求
```
return new Promise((resolve,reject) => {调服务,resolve(结果)})
```
#### 下拉、文本切换
`v-lookUpName` 自定义指令，用于数据转义
```
<aui-dropdown v-if='页面权限' v-bind="dropDownOp"></aui-dropdown>
<span v-else v-lookUpName="{}"></span>
```
#### 列表字段加链接
`#data` 为列表行数据，通过`a`标签链接，链接的方法为`link`，` grid.getRowData(target)` 获取行数据
```
content:'{{:~renderUtil.get(#data)}}'
renderUtil：{
    get:data => {
        return '<a eno="link"></a>'
    }
}
onComplete:(grid,op)= {grid.setEvent('link',(target,grid,e) => {
    let rowData = grid.getRowData(target)
}
)}
```
#### 重新渲染附件列表表格
深度克隆`Op`（表格配置），然后赋值再渲染
```
const grid = this.$refs.gridRef.widget
let newOp = _.cloneDeep(this.Op)
newOp.dataset.value = grid.getValue()
grid.reInit(newOp)
```
#### Grid 三部曲
```
获取grid：const grid = this.$refs.gridRef.widget
获取row：const row = grid.getFocusRow()
获取数据: const rowData = grid.getRowData(row)
```
#### 列表经常用的函数
```
 对选项的数据进行处理，返回true表示可以选
 onBeforeSelected:(data) => {return true}
 编辑之后数据处理，一般用来处理渲染到列表的数据
 onAfterEdit:(value,rowData,td,col,oldValue) => {
 }
 是否可以编辑
 onBeforeEdit:(cellValue,rowData,td,col) => {return true}
```
#### 搜索区域 涉及分页
```
this.$service.business.getData = args => {
    const {curPage,pageSize} = args.pageVO
    const url = ''
    const params = {
        pageVO: {curPage,pageSize},
        ...this.searchData
    }
    retrun new Promise((resolve,reject) => 
    this.$service.network.post(url,params).then(res => {
        resolve(res.data)
    })
    )
}
```
#### 搜索重置，重置按钮
```
this.$refs.xxxRef.widget.search(this.searchData)
resetButton() {
    for(let key in this.searchData){
        this.searchData[key] = ''
    }
    this.$refs.xxxRef.widget.rebuild()
    this.$nextTick(()= > {
        this.$resf.xxxRefs.widget.search(this.searchData)
    })
}
```
#### lookUp 请求
```
this.$service.business.getLookUpItem({classifyCode:'lookUp配置的key'}).then(ele => {do something})
```
#### 页面多次请求服务
```
Timer:null 定时器
let totalTime = 40 * 1000 倒计时总时间，这里设置请求四次
const interval = 10 * 1000 每 10 秒请求一次
const getApi = () => {
    totalTime -= interval
    if(totalTime > -1) {
        do something
    }
}
getApi() 先执行一次
this.Timer = setInterval(getApi,interval) 每 10 请求一次
```
#### 集合根据某个字段判断是否有重复项
```
let map = data.reduce((all,ele) => {
    let key =  ele.属性
    let list = all.get(key)
    if(!list) {
        list = []
        all.set(key,list)
    }
    list.push(ele)
    return all
    },new Map())
查找
let find = Array.from(map.entries()).filter(([des,list]) => list.length > 1) 
```
#### 列表是否可以多选
```
const op = _.cloneDeep(this.gridOp)
op.columns.find(ele => ele.field = 'multi').hideable = false
const grid = this.$resf.gridRef.widget
op.dataset.value = grid.getValue()
grid.reInit(op)
```
#### 表单和Grid必填校验
```
window._Hae.validForm(window._$('#id'),valid => {do something})
grid.valid(valid => {do something})
```
#### 多个方法异步调用
```
Promise.all([方法,other])
```
#### 异步请求表格中人员数据
```
refreshGridUserData
```
#### 利用routeEvent 页面权限控制
```
index.js 文件
router.beforeEach((to,from,next) => {do something}) 
```
#### 去重
```
distint(list) {
    let result = []
    const map = new Map()
    result = list.filter(ele => !map.has(ele.属性) && res.set(ele.属性,1)).map(item => ({
    ...item,
    添加的字段:字段值
    }))
    return result
}
```
#### 过滤非法字段
```
const pureGridDataList = (dataList=[]) => {
return dataList.map(item => {
        return _.reduce(item,(result,value,key) => {
            if(!_.startsWith(key,'_$') && _.includes(key,'__') && !_includes('$$$')) {
                result[kye] = value
            }
            return result
        },{})
    })
}
```
#### 延时
```
setTimeout(() => {
    do something
},100)
```
#### Excel文件转换JSON
```
const parseFileToJson = (file) => {
    return new Promise((resolve,reject) => {
        if(file.typ === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') {
           // 兼容ie
           if(typeof(FileReader.prototype.readAsBinaryString) !== 'function') {
               FileReader.prototype.readAsBinaryString = function(fileData) {
                   let binary = ''
                   let pt = this
                   const fileReader = new FileReader()
                   fileReader.onload = function(e) {
                       var bytes = new Uint8Array(fileReader.result)
                       var length = bytes.byteLength
                       for(var i = 0; i < length; i++) {
                           binary +=
                           String.fromCharCode(bytes[i])
                       }
                       pt.currentTarget = {}
                       pt.currentTarget.result = binary
                       pt.onload(pt)
                   }
                   fileReader.readArrayBuffer(fileData)
               }
           }
           const fileReader = new FileReader()
           fileReader.onload = (e) => {
               const data = e.currentTarget.result
               const workbook = xlsx.read(data,{type:'binary'})
               const sheet = xlsx.utils.sheet_to_json(workbook.Sheets[workbook.SheetName[0]])
               resolve(sheet)
           }
           return fileReader.readAsBinaryString(file)
        }
    })
}
```
#### 循坏
```
_.forEach(list,(item,index) => {do something})
```
#### 获取当前用户
```
this.$service.base.getUserInfoSync().userId
this.$service.base.getUserInfoSync().userCN
this.$service.base.getUserInfoSync().userAccount
```
#### 文本域权限控制
```
<aui-textarea v-readonly='true'></aui-textarea>
```
#### 列表单元格数据显示
```
window.__$(grid.getRowCell(curRow,'字段').text(字段值))
```
#### 列表字段高亮
```
onRenderRow:(tr,rowData,rowIdx) => {
    this.$service.business.getLookUpItem({classifyCode:'XXX'}).then(item => {tr.find(`td[field = "&{字段}"]`).addClass('背景颜色')})
}
```