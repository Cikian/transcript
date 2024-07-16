<template>
  <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px">
    <div style="width: 10%; font-size: 18px">
      标题：
    </div>
    <el-input
        v-model="form.title"
        style="width: 90%; height: 50px; font-size: 22px; font-weight: bold "
        placeholder="请输入标题"
        clearable
        size="large"
    />
  </div>
  <el-scrollbar ref="scoRef" max-height="60vh"
                style="width: 45vw; padding: 30px 20px; border: #409eff solid 1px; border-radius: 10px; box-shadow: #409eff 0 0 5px; margin-bottom: 50px">
    <div v-for="item in content" :key="item.index" style="margin-bottom: 30px;">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px">
        <div style="width: 10%; font-size: 18px">
          问题{{ item.index }}
        </div>
        <el-input
            v-model="item.question"
            style="width: 80%; height: 50px; font-size: 18px; font-weight: bold "
            placeholder="请输入问题"
            clearable
            size="large"
        />
        <div style="width: 10%;">
          <el-button type="danger" size="small" @click="reduceLine(item.index)">删除</el-button>
        </div>
      </div>

      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px">
        <div style="width: 10%; font-size: 16px">答</div>
        <el-input
            v-model="item.answer"
            style="width: 80%"
            :autosize="{ minRows: 2 }"
            type="textarea"
            placeholder="请输入回答"
            size="large"
        />
        <div style="width: 10%;">
        </div>
      </div>
    </div>
  </el-scrollbar>

  <div style="display: flex; justify-content: right; align-items: center; margin-bottom: 20px">
    <div style="width: 10%; font-size: 16px">
      记录人：
    </div>
    <el-input
        v-model="form.recorder"
        style="width: 30%; height: 30px; font-size: 16px; font-weight: bold "
        placeholder="请输入记录人"
        clearable
        size="large"
    />
  </div>

  <el-button size="large" type="primary" @click="addLine" style="font-size: 16px">添加1行</el-button>
  <el-button size="large" type="success" @click="submit" style="font-size: 16px">导出Word</el-button>

</template>

<script setup lang="ts">
import {ref} from "vue";
import axios, {options} from "axios";

const scoRef = ref();

const form = ref({
  title: '',
  content: [],
  recorder: ''
})

const content: qa[] = ref([
  {
    index: 1,
    question: '',
    answer: ''
  },
  {
    index: 2,
    question: '',
    answer: ''
  },
  {
    index: 3,
    question: '',
    answer: ''
  }
])

const submit = () => {
  if (form.value.title === '') {
    ElNotification({
      title: '标题不能为空！',
      message: '请输入标题',
      type: 'error',
    })
    return
  }

  for (let i = 0; i < content.value.length; i++) {
    if (content.value[i].question === '' || content.value[i].answer === '') {
      ElNotification({
        title: '问题或答案不能为空！',
        message: '存在空问题或空答案，请检查问答记录，补全内容',
        type: 'error',
      })
      return
    }
  }

  if (form.value.recorder === '') {
    ElNotification({
      title: '记录人不能为空！',
      message: '请输入记录人',
      type: 'error',
    })
    return
  }

  form.value.content = content.value
  console.log(form.value)
  axios.post('/api/ts', {
    title: form.value.title,
    content: form.value.content,
    recorder: form.value.recorder
  },{
    responseType: 'blob'
  },{
     headers: {
       "content-type":"multipart/from-data"
    }
  }).then((res)=>{
    console.log(res)
    console.log(res.data.data)
    const link=document.createElement('a');
    try {
      //如果文件类型不确定的时候，可以不设置type
      let blob = new Blob([res.data],{type: 'application/vnd.ms-word'});
      console.log(res.headers['filename'])
      let _fileName = res.headers['filename'].split(';')[1].split('=')
          [1];//文件名，中文无法解析的时候会显示 _(下划线),生产环境获取不到
      // 解码
      _fileName = decodeURIComponent(_fileName);
      link.style.display='none';
      // 兼容不同浏览器的URL对象
      const url = window.URL || window.webkitURL || window.moxURL;

      link.href=url.createObjectURL(blob);
      link.setAttribute('download', _fileName.substring(_fileName.lastIndexOf('_')+1));
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      url.revokeObjectURL(link.href);//销毁url对象
    }catch (e) {
      console.log('下载的文件出错',e)
    }

  }).catch((err)=>{
    console.log('请求出错',err.response.data.error);
  })

}


const addLine = () => {
  content.value.push({
    index: content.value.length + 1,
    question: '',
    answer: ''
  })
  scoRef.value!.setScrollTop(999999999999999999)
}

const reduceLine = (index: number) => {
  if (content.value.length === 1) {
    ElNotification({
      title: '不可删除！',
      message: '至少保留一组问答。',
      type: 'error',
    })

    return
  }
  content.value = content.value.filter(item => item.index !== index)
  content.value.forEach((item, index) => item.index = index + 1)
}
</script>

<style scoped>

</style>
