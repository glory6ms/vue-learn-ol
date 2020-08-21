<template>
  <el-upload
    class="upload-demo"
    drag
    action="https://jsonplaceholder.typicode.com/posts/"
    multiple
  >
    <i class="el-icon-upload" />
    <div class="el-upload__text">将海图文件拖到此处，或<em>点击上传</em></div>
    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件</div>
  </el-upload>
</template>

<script>
export default {
  name: 'Charts'
}
</script>

<style scoped>

</style>
