## 1. Vue 父组件 <--> 子组件通信 (Vue 2)

### 父组件 --> 子组件 (Props)

父组件在使用子组件时，通过 `v-bind` 来绑定一个自定义属性，并把要传递的值赋值给它。

子组件在自身的 `props` 中，声明接受的属性名称。

数据从父组件 --> 子组件，子组件不应该自行修改 `props` 声明的属性值。

### 子组件 --> 父组件 (Emit)

子组件无法直接修改父组件数据，但是可以通过 `emit` 通知父组件，`emit` 可以传递数据。

父组件通过 `v-on` 监听这个自定义事件。

### 父组件 <--> 子组件 (sync)

在 Vue 2 中，数据的双向流动可以通过 `v-model` 或者 `.sync` 来实现。`.sync` 是一种语法糖，其语法为：`:<prop_name>.sync="value"` ，例如 `:visible.sync="xxx"`。父组件的 `xxx` 发生变化时，会导致子组件的 `visible` 发生变化，子组件的 `visible` 发生变化时，会触发一个事件 `update:visible`，父组件通过 `@update:visible` 来接收。

## 2. 新增 / 修改组件复用 (Vue 2)

在开发后台管理系统中，常见的需求是：点击新增，弹出一个新增对话框，填写相应信息后点击确定，然后发送新增请求。在表格条目的右侧存在一个修改按钮，点击修改按钮，弹出一个修改对话框，==对话框的布局与新增对话框一致==，只是里面会事先填好相应的内容。

解决这个需求的最佳实践是：**创建一个可复用的“新增/修改”对话框组件**。

### 子组件示例

```vue
<template>
  <el-dialog
    :title="title"
    :visible.sync="localVisible"
    width="50%"
    @close="handleClose"
  >
    <el-form ref="form" :model="form" :rules="rules" label-width="80px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="form.email" placeholder="请输入邮箱"></el-input>
      </el-form-item>
      </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="handleClose">取 消</el-button>
      <el-button type="primary" @click="handleConfirm">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  name: 'UserFormDialog',
  props: {
    // 控制对话框是否可见
    visible: {
      type: Boolean,
      default: false
    },
    // 对话框的模式：'add' 或 'edit'
    mode: {
      type: String,
      default: 'add',
      validator: (value) => ['add', 'edit'].includes(value)
    },
    // “修改”模式下的初始数据
    initialData: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      form: {
        id: null,
        username: '',
        email: ''
        // ...其他字段
      },
      rules: {
        username: [{ required: true, message: '用户名为必填项', trigger: 'blur' }],
        email: [
          { required: true, message: '邮箱为必填项', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱格式', trigger: ['blur', 'change'] }
        ]
      }
    };
  },
  computed: {
    // 1. 根据模式动态计算标题
    title() {
      return this.mode === 'add' ? '新增用户' : '修改用户';
    },
    // 使用一个本地变量来同步prop，避免直接修改prop
    localVisible: {
        get() {
            return this.visible;
        },
        set(val) {
            this.$emit('update:visible', val);
        }
    }
  },
  watch: {
    // 2. 监听 visible 的变化，当对话框弹出时，初始化表单
    visible(newVal) {
      if (newVal) {
        this.resetForm(); // 先重置表单
        if (this.mode === 'edit') {
          // 修改模式：用初始数据填充表单
          // 使用深拷贝，避免直接修改父组件传来的对象
          this.form = JSON.parse(JSON.stringify(this.initialData));
        }
      }
    }
  },
  methods: {
    // 3. 点击“确定”按钮的逻辑
    handleConfirm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          // 表单验证通过，通过 $emit 将数据发送给父组件
          this.$emit('submit', this.form);
        } else {
          console.log('表单验证失败');
          return false;
        }
      });
    },
    // 4. 关闭/取消的逻辑
    handleClose() {
      this.$emit('update:visible', false);
    },
    // 重置表单状态
    resetForm() {
      this.form = {
        id: null,
        username: '',
        email: ''
      };
      // 清除上一次的校验信息
      this.$nextTick(() => {
        if (this.$refs.form) {
            this.$refs.form.clearValidate();
        }
      });
    }
  }
};
</script>
```

### 父组件示例

```vue
<template>
  <div>
    <el-button type="primary" @click="handleAdd">新增</el-button>

    <el-table :data="tableData">
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="handleEdit(scope.row)">修改</el-button>
        </template>
      </el-table-column>
    </el-table>

    <user-form-dialog
      :visible.sync="dialogVisible"
      :mode="dialogMode"
      :initial-data="currentRow"
      @submit="handleSubmit"
    ></user-form-dialog>

  </div>
</template>

<script>
import UserFormDialog from './UserFormDialog.vue';
// 假设你有一个 api.js 文件来管理所有后端请求
import * as api from '@/api/user'; 

export default {
  components: {
    UserFormDialog
  },
  data() {
    return {
      tableData: [], // 表格数据
      dialogVisible: false, // 控制对话框的显示
      dialogMode: 'add', // 对话框的模式
      currentRow: null // 当前正在编辑的行数据
    };
  },
  methods: {
    // 6. 点击“新增”按钮
    handleAdd() {
      this.dialogMode = 'add';
      this.currentRow = null; // 新增模式下没有初始数据
      this.dialogVisible = true;
    },
    // 7. 点击“修改”按钮
    handleEdit(row) {
      this.dialogMode = 'edit';
      this.currentRow = row; // 传入当前行的数据
      this.dialogVisible = true;
    },
    // 8. 监听对话框的 submit 事件
    async handleSubmit(formData) {
      try {
        if (this.dialogMode === 'add') {
          // 调用新增接口
          await api.addUser(formData);
          this.$message.success('新增成功');
        } else {
          // 调用修改接口
          await api.updateUser(formData);
          this.$message.success('修改成功');
        }
        
        // 不论是新增还是修改，成功后都关闭对话框并刷新表格
        this.dialogVisible = false;
        this.fetchTableData(); // 重新加载表格数据
      } catch (error) {
        console.error('操作失败', error);
        // 可以在这里显示错误信息
      }
    },
    fetchTableData() {
      // 获取表格数据的方法
    }
  },
  created() {
    this.fetchTableData();
  }
};
</script>
```

## 3. 利用 Vuex 缓存数据或者多层传递数据

在开发中有两种常见需求：

1. 有一些固定的数据需要在初始化时从后端获取，随后便不再发生变化。
2. 子组件需要向自己的上级的上级的上级... 传递数据。

对于第一个需求，自然会想到利用缓存：每次需要数据时查一下缓存，如果缓存命中则直接获取数据，否则从后端读取数据。

对于第二个需求，如果嵌套层数不多，可以通过之间介绍过的 `emit()` 的方式进行传递，如果嵌套层数过多，子组件可以直接把数据放到一个数据中心 (缓存) 中，然后父组件需要的时候就去读取即可。

### 具体写法

#### Store

在 `/stores` 目录下创建一个 `.js` 文件，表示一个 `Store` 模块。下面给出一个样例写法 (`WageItemDropdown.js`) ：

```javascript
import Vue from "vue";

const state = {
  wageItems: [],
};

const mutations = {
  SET_WAGE_ITEMS(state, items) {
    state.wageItems = items;
  },
};

const actions = {
  fetchWageItems({ commit, state }) {
    if (state.wageItems.length > 0) return;
    Vue.prototype.post(
      {
        url: "otherSalaryItemsList",
      },
      (res) => {
        if (res.code === 200) {
          commit("SET_WAGE_ITEMS", res.data);
        }
      }
    );
  },
};

const getters = {
  getWageItems: (state) =>
    [...state.wageItems].sort((a, b) => a.itemOrder - b.itemOrder),
};

export default {
  namespaced: true,
  state,
  mutations,
  actions,
  getters,
};

```

- `state` 用于定义需要存储的数据
- `mutations` 用于定义修改数据的方法，在 Vuex 中，我们约定只有 `mutations` 中的方法才能修改 `state` 中的数据
- `actions` 用于定义一般性的方法，例如获取数据，在获取完成，需要修改 `state` 中的数据时，需要用 `commit()` 配合 `mutations` 中的方法来完成，这是固定写法
- `getters` 可以看作是 Vue 中的计算属性
- 最后在 `export` 的时候，如果 `namespaced: true`，那么在使用这个 `store` 时，需要带上命名空间

在定义完成之后，需要在 Vuex 中注册该 `Store` 模块。
#### 使用

在使用时，通过 `getters` 获取数据，如果需要一些其他操作，则通过 `actions` 完成，Vue 提供了 `mapGetters()` 和 `mapActions()` 来辅助进行这两种操作。

```vue
<script>
import { mapActions, mapGetters } from "vuex/dist/vuex.common.js";

export default {
	methods: {
		// WageItemDropdown 为命名空间
		// 可以像使用普通方法一样使用 fetchWageItems，方法名称一致
		...mapActions("WageItemDropdown", ["fetchWageItems"]),
	},
	computed: {
		// 可以像使用普通的计算属性一样使用 getter，方法名为 wageItems
		...mapGetters("WageItemDropdown", { wageItems: "getWageItems" }),
	}
};
</script>
```