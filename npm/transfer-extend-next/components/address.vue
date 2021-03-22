<template>
  <div class="component-address">
    <!-- 左侧穿梭框 原料框 -->
    <div class="transfer-left">
      <h3 class="transfer-title">
        <span>{{ fromTitle }}</span>
      </h3>
      <!-- 内容区 -->
      <div class="transfer-main">
        <slot name="from"></slot>
        <el-tree
          ref="from-tree"
          :indent="indent"
          :draggable="draggable"
          :allow-drag="allowDrag"
          :allow-drop="allowDrop"
          :icon-class="iconClass"
          :node-key="node_key"
          :props="defaultProps"
          :data="self_from_data"
          @node-click="handleNodeClick"
          :default-expand-all="openAll"
          :highlight-current="highLight"
          :render-content="renderContentLeft"
          :check-on-click-node="checkOnClickNode"
          :render-after-expand="renderAfterExpand"
          :expand-on-click-node="expandOnClickNode"
          :default-checked-keys="defaultCheckedKeys"
        ></el-tree>
      </div>
    </div>
    <!-- 穿梭区 按钮框 -->
    <div class="transfer-center address-list-center">
      <p
        class="transfer-center-item address-only-item"
      >
        <el-button
          type="primary"
          @click="addressListTransfer"
          icon="el-icon-arrow-right"
          circle
          class="address-first-btn"
          :disabled="from_disabled"
        ></el-button>
      </p>
    </div>
    <!-- 右侧列表区 -->
    <div class="transfer-right">
      <div
        class="transfer-right-item transfer-right-only"
      >
        <h3 class="transfer-title">
          <span>{{ toTitle }}</span>
          <span class="u-clear" @click="clearToList('all')">清空</span>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <slot name="to"></slot>
          <el-input
            v-if="filter"
            :placeholder="placeholder"
            v-model="filterList"
            size="small"
            class="filter-tree"
          ></el-input>
          <ul class="address-list-ul">
            <li class="address-list-li" v-for="item of to_list" :key="item[node_key]">
              <label>
                {{ item.data.path }}
<!--                {{ item[options.suffix] }}-->
<!--                {{ options.connector }}-->
<!--                {{ item[defaultProps.label] }}-->
              </label>
              <i
                class="address-list-del el-icon-delete"
                @click="clearToList(item[node_key])"
              ></i>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { arrayToTree } from "wl-core";

export default {
  name: "AddressBook",
  data() {
    return {
      from_disabled: true, // 添加按钮是否禁用
      from_checked: [], // 源数据选中的节点
      from_check_keys: [], // 源数据选中key数组 以此属性关联穿梭按钮，总全选、半选状态
      to_path: '', // 保存右侧的路径
      filterList: "", // 通讯录模式 右1筛选
      archive: [], // 存档右侧筛选前数据
      to_list: [], // 收件人列表
      move_up: false, // 通讯录模式 切换右侧
    };
  },
  props: {
    // 标题
    title: {
      type: Array,
      default: () => ["源列表", "目标列表"],
    },
    // 源数据
    from_data: {
      type: Array,
      default: () => [],
    },
    // el-tree 配置项
    defaultProps: {
      type: Object,
      default: () => {
        return { label: "label", children: "children" };
      },
    },
    // el-tree node-key 必须唯一
    node_key: {
      type: String,
      default: "id",
    },
    // 自定义 pid参数名
    pid: {
      type: String,
      default: "pid",
    },
    // 是否启用筛选
    filter: {
      type: Boolean,
      default: false,
    },
    // 是否展开所有节点
    openAll: {
      type: Boolean,
      default: false,
    },
    // 左侧自定义树节点
    renderContentLeft: Function,
    // 通讯录模式配置项 num-> 所需右侧通讯录个数 suffix-> label后想要拼接的字段（如id，即取此条数据的id拼接在后方）connector -> 连接符（字符串）
    options: {
      type: Object,
      default: () => {
        return {
          num: 1,
          suffix: "suffix",
          connector: "-",
        };
      },
    },
    // 源数据 默认选中节点
    defaultCheckedKeys: {
      type: Array,
      default: () => [],
    },
    // 源数据 默认展开节点
    defaultExpandedKeys: {
      type: Array,
      default: () => [],
    },
    // 筛选placeholder
    placeholder: {
      type: String,
      default: "输入关键字进行过滤",
    },
    // 自定义筛选函数
    filterNode: Function,
    // 是否开启arrayToTree
    arrayToTree: {
      type: Boolean,
      default: false,
    },
    // 是否启用懒加载
    lazy: {
      type: Boolean,
      default: false,
    },
    // 懒加载的回调函数
    lazyFn: Function,
    // 是否高亮当前选中节点，默认值是 false。
    highLight: {
      type: Boolean,
      default: false,
    },
    // 是否遵循父子不关联
    checkStrictly: {
      type: Boolean,
      default: false,
    },
    // 是否每次只打开一个同级树节点
    accordion: {
      type: Boolean,
      default: false,
    },
    // 是否在第一次展开某个树节点后才渲染其子节点
    renderAfterExpand: {
      type: Boolean,
      default: true,
    },
    // 是否在点击节点的时候展开或者收缩节点
    expandOnClickNode: {
      type: Boolean,
      default: true,
    },
    // 是否在点击节点的时候选中节点
    checkOnClickNode: {
      type: Boolean,
      default: false,
    },
    // 相邻级节点间的水平缩进，单位为像素
    indent: {
      type: Number,
      default: 16,
    },
    // 	自定义树节点的图标
    iconClass: String,
    // 是否开启拖拽节点功能
    draggable: Boolean,
    // 判断节点能否被拖拽
    allowDrag: Function,
    // 拖拽时判定目标节点能否被放置
    allowDrop: Function,
  },
  computed: {
    // 左侧数据
    self_from_data() {
      let from_array = [...this.from_data];
      return !this.arrayToTree
        ? from_array
        : arrayToTree(from_array, {
            id: this.node_key,
            pid: this.pid,
            children: this.defaultProps.children,
          });
    },
    // 左侧菜单名
    fromTitle() {
      let [text] = this.title;
      return text;
    },
    // 右侧菜单名
    toTitle() {
      let [, text] = this.title;
      return text;
    }
    // checkedPath () {
    //   let path = ''
    //   let node = this.from_checked[0]
    //   while(node.parent) {
    //     console.log(node)
    //     path = node.parent.data.name + '-' + path
    //     node = node.parent
    //     console.log(path)
    //   }
    //   return path
    // }
  },
  watch: {
    // 左侧 状态监测
    from_checked(node) {
      // 当type=3或4, 即被选中节点是岗位或人员的时候, 允许点击穿梭按钮
      if (node.childNodes == null || node.childNodes.length === 0) {
        this.from_disabled = false
      } else {
        this.from_disabled = true
      }
      // this.from_disabled = node.childNodes.length !== 0;
    },
    // 通讯录模式筛选
    filterList(newval, oldval) {
      if (oldval === "") {
        this.archiveFirst = this.to_list;
      }
      if (newval === "") {
        this.to_list = this.archiveFirst;
      }
      let reg = RegExp(newval);
      this.to_list = this.to_list.filter((item) => reg.test(item.label));
    }
  },
  methods: {
    // ---------------------提供输出函数---------------------
    handleNodeClick (data, node, comp) {
      this.from_checked = [node]
    },
    // 通讯录模式 穿梭操作
    addressListTransfer() {
      // 添加到选中
      this.to_list = [...this.to_list, ...this.from_checked];
      // 处理完毕取消选中
      this.from_checked = []
      // 传递信息给父组件
      this.$emit("add-btn", this.to_list);
    },
    // 清理 通讯录选中 数据
    clearToList(id) {
      this.to_list =
        id === "all" ? [] : this.to_list.filter((item) => item[this.node_key] !== id);
      // 传递信息给父组件
      this.$emit("remove-btn", this.to_list);
    },
    // 右侧 通讯录 上下自动
    moveUp(type) {
      this.move_up = type === "up";
    }
  }
};
</script>
