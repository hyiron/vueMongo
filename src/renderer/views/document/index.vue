<template>
  <div class="page">
    <div class="toolbar">
      <el-button size="small" type="success" @click="goback">返回</el-button>
      <el-button size="small" type="primary" @click="save">保存</el-button>
    </div>

    <div class="editor">
      <JSONEditor v-model="data" />
    </div>
  </div>
</template>

<script>
import JSONEditor from '@/components/json-editor.vue';

export default {
  name: 'Document',

  components: {
    JSONEditor
  },

  beforeRouteLeave(to, from, next) {
    if ( // 数据变动且未保存
      this.originData !== this.data && !this.isSaved
    ) {
       this.$confirm('数据有变动且未保存，是否保存?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          showInput: false
        }).then(async () => {
          await this.save();
          next()
        }).catch(() => next());
    } else {
      next();
    }
  },

  data() {
    return {
      originData: {}, // 原本的数据，用于比对是否改动了数据
      data: {},
      isSaved: false,
    }
  },

  created() {
    this.fetchDocument();
  },

  methods: {
    async fetchDocument() {
      this.$loading.start();
      try {
        const { db, collection, document } = this.$router.currentRoute.params;
        const data = await this.$http.query.findById(db, collection, document);
        this.data = data;
        this.$nextTick(_ => this.originData = this.data); // 用于比对，JSON编辑器会字符串话数据，故如此
      } catch (err) {
        this.$message.error(err.message)
      } finally {
        this.$loading.close();
      }
    },

    async save() {
      this.$loading.start();
      try {
        const { db, collection, document } = this.$router.currentRoute.params;
        await this.$http.query.updateById(db, collection, document, this.data);
        this.$message.success('数据更新成功');
        this.isSaved = true;
      } catch (err) {
        this.$message.error(err.message);
      } finally {
        this.$loading.close();
      }
    },

    goback() {
      const { db, collection } = this.$router.currentRoute.params;
      this.$router.push(`/db/${db}/collection/${collection}`);
    },
  },
}
</script>

<style lang="scss" scoped>
.page {
  height: 100%;
  padding: 15px;
}

div.toolbar {
  height: 30px;
  margin-bottom: 15px;
  padding: 0 !important;
}

.text-right {
  text-align: right;
}

.editor {
  height: calc(100% - 80px);
}
</style>
