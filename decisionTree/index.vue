<template>
  <div class="decision-tree-container" :style="{ backgroundImage: `url(${ASSETS.pageBg})` }">
    <!-- 左侧：决策树主区域 -->
    <div class="left-panel">
      <!-- 页面标题 -->
      <div class="top-header">
        <img :src="ASSETS.pageTitle" alt="天气决策树" class="header-img" />
      </div>

      <!-- 决策树画布（包含背景框、节点、连线、数据动画） -->
      <div class="tree-area-container">
        <img :src="ASSETS.treeFrameBg" class="tree-area-bg" />
        
        <div class="tree-area" @click="openMenuNodeId = ''">
          
          <!-- 顶部：待分类数据容器 -->
          <div class="waiting-box">
            <img :src="ASSETS.waitingBoxBg" class="waiting-box-bg" />
            <img :src="ASSETS.waitingLabel" class="waiting-badge" />
          </div>
        
        <!-- 连接线背景（用于指引动画滚动轨迹） -->
        <img :src="ASSETS.pathLevel1" class="path-bg path-level-1" />
        <!-- 下层连接线背景（复用同一张图，按位置/层级摆放） -->
        <img :src="ASSETS.pathLevel2" class="path-bg path-level-2 path-left" v-if="treeNodes['node_2_yes']" />
        <img :src="ASSETS.pathLevel2" class="path-bg path-level-2 path-right" v-if="treeNodes['node_2_no']" />

        <!-- 节点：筛选依据选择入口（点击弹出下拉菜单） -->
        <!-- 第 1 层（根节点） -->
        <div class="filter-node node-l1" @click="toggleNodeMenu('root')">
          <img :src="getFilterImage(treeNodes['root'].selectedFilter)" class="filter-img" />
        </div>

        <!-- 第 2 层（左右两个节点，分别独立选择筛选依据） -->
        <div class="filter-node node-l2 node-l2-left" v-if="treeNodes['node_2_yes'] && treeNodes['root'].status === 'completed'" @click="toggleNodeMenu('node_2_yes')">
          <img :src="getFilterImage(treeNodes['node_2_yes'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l2 node-l2-right" v-if="treeNodes['node_2_no'] && treeNodes['root'].status === 'completed'" @click="toggleNodeMenu('node_2_no')">
          <img :src="getFilterImage(treeNodes['node_2_no'].selectedFilter)" class="filter-img" />
        </div>

        <!-- 第 3 层（4 个节点，分别独立选择筛选依据） -->
        <div class="filter-node node-l3 node-l3-ll" v-if="treeNodes['node_3_ll'] && treeNodes['node_2_yes'].status === 'completed'" @click="toggleNodeMenu('node_3_ll')">
          <img :src="getFilterImage(treeNodes['node_3_ll'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-lr" v-if="treeNodes['node_3_lr'] && treeNodes['node_2_yes'].status === 'completed'" @click="toggleNodeMenu('node_3_lr')">
          <img :src="getFilterImage(treeNodes['node_3_lr'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-rl" v-if="treeNodes['node_3_rl'] && treeNodes['node_2_no'].status === 'completed'" @click="toggleNodeMenu('node_3_rl')">
          <img :src="getFilterImage(treeNodes['node_3_rl'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-rr" v-if="treeNodes['node_3_rr'] && treeNodes['node_2_no'].status === 'completed'" @click="toggleNodeMenu('node_3_rr')">
          <img :src="getFilterImage(treeNodes['node_3_rr'].selectedFilter)" class="filter-img" />
        </div>

        <div
          v-if="openMenuNodeId"
          class="node-menu"
          :style="getNodeMenuStyle(openMenuNodeId)"
          @click.stop
        >
          <div
            v-for="opt in getNodeMenuOptions(openMenuNodeId)"
            :key="opt"
            class="node-menu-item"
            :class="{ active: treeNodes[openMenuNodeId]?.selectedFilter === opt }"
            @click="selectNodeFilter(openMenuNodeId, opt)"
          >
            {{ getFilterText(opt) }}
          </div>
        </div>

        <!-- 容器框：用于承载数据图标（每一层的“是/否”分支容器） -->
        <!-- 第 2 层容器（左右两个大框） -->
        <div class="box-container box-l2-left">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge" />
        </div>
        <div class="box-container box-l2-right">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge" />
        </div>

        <!-- 第 3 层容器（4 个中框） -->
        <div class="box-container box-l3-ll box-medium">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge" />
        </div>
        <div class="box-container box-l3-lr box-medium">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge" />
        </div>
        <div class="box-container box-l3-rl box-medium">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge" />
        </div>
        <div class="box-container box-l3-rr box-medium">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge" />
        </div>

        <!-- 第 4 层：8 个叶子结果框（最终落点，显示结果标签） -->
        <div class="box-container box-leaf box-leaf-1">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_1')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-2">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_2')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-3">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_3')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-4">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_4')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-5">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_5')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-6">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_6')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-7">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_7')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-8">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_8')" class="box-result-label" v-if="isTreeCompleted" />
        </div>

          <!-- 12 个待分类数据图标（通过坐标/缩放控制动画） -->
          <div
            v-for="item in items"
            :key="item.id"
            class="data-item"
            :style="{
              left: `${item.x}px`,
              top: `${item.y}px`,
              transform: `translate(-50%, -50%) scale(${item.scale})`,
              opacity: item.visible ? 1 : 0
            }"
          >
            <img :src="item.img" class="item-icon" />
          </div>

          <!-- 预测图标（问号，预测时沿着树路径滚动，结束后替换为对应天气图标） -->
          <div
            v-if="predictItem.visible"
            class="data-item predict-item"
            :style="{
              left: `${predictItem.x}px`,
              top: `${predictItem.y}px`,
              transform: `translate(-50%, -50%) scale(${predictItem.scale})`,
              zIndex: 100
            }"
          >
            <img :src="predictItem.img" class="item-icon" />
          </div>
        </div>
      </div>

      <!-- 底部操作区 -->
      <div class="bottom-controls">
        <img :src="ASSETS.btnRestart" class="btn btn-restart" @click="resetAll" />
        <div class="center-buttons">
          <img :src="ASSETS.btnPrev" class="btn btn-prev" @click="goBack" />
          <img
            :src="canStart ? ASSETS.btnStartFilled : ASSETS.btnStartOutline"
            class="btn btn-start"
            :class="{ disabled: !canStart }"
            @click="startClassification"
          />
        </div>
      </div>
    </div>

    <!-- 右侧：天气预测区 -->
    <div class="right-panel">
      <img :src="ASSETS.rightPanelBg" class="right-panel-bg" />
      <div class="right-panel-content">
        <div class="panel-header">
          <img :src="ASSETS.rightPanelTitle" alt="天气预测区" class="panel-title-img" />
        </div>
        
        <div class="panel-body">
          <div class="option-row">
            <span class="option-label"><img :src="ASSETS.optionDot" class="option-label-icon" />是否降水:</span>
            <div class="toggle-group">
              <img :src="predictOptions.rain === true ? ASSETS.toggleYesActive : ASSETS.toggleYesInactive" class="toggle-img" @click="predictOptions.rain = true" />
              <img :src="predictOptions.rain === false ? ASSETS.toggleNoActive : ASSETS.toggleNoInactive" class="toggle-img" @click="predictOptions.rain = false" />
            </div>
          </div>
          <div class="option-row">
            <span class="option-label"><img :src="ASSETS.optionDot" class="option-label-icon" />是否有阳光:</span>
            <div class="toggle-group">
              <img :src="predictOptions.sun === true ? ASSETS.toggleYesActive : ASSETS.toggleYesInactive" class="toggle-img" @click="predictOptions.sun = true" />
              <img :src="predictOptions.sun === false ? ASSETS.toggleNoActive : ASSETS.toggleNoInactive" class="toggle-img" @click="predictOptions.sun = false" />
            </div>
          </div>
          <div class="option-row">
            <span class="option-label"><img :src="ASSETS.optionDot" class="option-label-icon" />是否零下:</span>
            <div class="toggle-group">
              <img :src="predictOptions.zero === true ? ASSETS.toggleYesActive : ASSETS.toggleYesInactive" class="toggle-img" @click="predictOptions.zero = true" />
              <img :src="predictOptions.zero === false ? ASSETS.toggleNoActive : ASSETS.toggleNoInactive" class="toggle-img" @click="predictOptions.zero = false" />
            </div>
          </div>
          <div class="divider"></div>
          <div class="option-row result-row">
            <span class="option-label"><img :src="ASSETS.optionDot" class="option-label-icon" />预测结果:</span>
            <span class="result-text">{{ predictResultText }}</span>
          </div>
        </div>
        <div class="panel-footer">
          <img :src="ASSETS.btnPredict" class="btn btn-predict" @click="startPrediction" />
        </div>
      </div>
    <!-- 最右侧：知识卡片/思考问题入口 -->
    <div class="sidebar">
      <img :src="ASSETS.sidebarKnowledge" class="sidebar-card" />
      <img :src="ASSETS.sidebarThinking" class="sidebar-card" />
    </div>

  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

/**
 * 静态资源映射表
 * 说明：
 * - 使用 new URL(..., import.meta.url).href 让 Vite 在构建时能正确处理资源路径
 * - 模板中尽量通过该映射引用图片，避免相对路径在不同构建模式下失效
 */
const ASSETS = {
  pageBg: new URL('./image/Slice 176.png', import.meta.url).href,
  pageTitle: new URL('./image/Rectangle 886.png', import.meta.url).href,
  treeFrameBg: new URL('./image/Rectangle 880.png', import.meta.url).href,
  waitingBoxBg: new URL('./image/Group 2186.png', import.meta.url).href,
  waitingLabel: new URL('./image/label_waiting.png', import.meta.url).href,
  pathLevel1: new URL('./image/Group 2174.png', import.meta.url).href,
  pathLevel2: new URL('./image/path_blue.png', import.meta.url).href,
  filterRain: new URL('./image/label_precipitation.png', import.meta.url).href,
  filterSun: new URL('./image/label_sunlight.png', import.meta.url).href,
  filterZero: new URL('./image/label_below_zero.png', import.meta.url).href,
  iconSnow: new URL('./image/icon_snow.png', import.meta.url).href,
  iconRain: new URL('./image/icon_rain.png', import.meta.url).href,
  iconSun: new URL('./image/icon_sun.png', import.meta.url).href,
  iconCloud: new URL('./image/icon_cloud.png', import.meta.url).href,
  iconQuestion: new URL('./image/icon_question.png', import.meta.url).href,
  resultSnow: new URL('./image/result_snow.png', import.meta.url).href,
  resultRain: new URL('./image/result_rain.png', import.meta.url).href,
  resultSun: new URL('./image/result_sun.png', import.meta.url).href,
  resultCloud: new URL('./image/result_cloud.png', import.meta.url).href,
  resultNone: new URL('./image/result_none.png', import.meta.url).href,
  boxYes: new URL('./image/box_yes.png', import.meta.url).href,
  boxNo: new URL('./image/box_no.png', import.meta.url).href,
  badgeYes: new URL('./image/Group 2255.png', import.meta.url).href,
  badgeNo: new URL('./image/Group 2256.png', import.meta.url).href,
  btnRestart: new URL('./image/btn_restart.png', import.meta.url).href,
  btnPrev: new URL('./image/btn_prev.png', import.meta.url).href,
  btnStartOutline: new URL('./image/btn_start_outline.png', import.meta.url).href,
  btnStartFilled: new URL('./image/btn_start_filled.png', import.meta.url).href,
  rightPanelBg: new URL('./image/Rectangle 881.png', import.meta.url).href,
  rightPanelTitle: new URL('./image/Group 2243.png', import.meta.url).href,
  btnPredict: new URL('./image/btn_restart_predict.png', import.meta.url).href,
  sidebarKnowledge: new URL('./image/Group 2241.png', import.meta.url).href,
  sidebarThinking: new URL('./image/Group 2242.png', import.meta.url).href,
  optionDot: new URL('./image/Group 2251.png', import.meta.url).href,
  toggleYesActive: new URL('./image/Group 2246.png', import.meta.url).href,
  toggleYesInactive: new URL('./image/Group 2247.png', import.meta.url).href,
  toggleNoActive: new URL('./image/Group 2248.png', import.meta.url).href,
  toggleNoInactive: new URL('./image/Group 2249.png', import.meta.url).href
};

// ====================
// 数据与枚举定义
// ====================
/**
 * 待分类数据原型（每种天气 3 个，一共 12 个）
 * 字段说明：
 * - rain：是否降水
 * - sun：是否有阳光
 * - zero：是否零下
 * - img：展示图标（已映射为可解析的资源 URL）
 * - type：类型标识（用于调试/辅助）
 */
const baseData = [
  { rain: true, sun: false, zero: true, img: ASSETS.iconSnow, type: 'snow' },
  { rain: true, sun: false, zero: false, img: ASSETS.iconRain, type: 'rain' },
  { rain: false, sun: true, zero: false, img: ASSETS.iconSun, type: 'sun' },
  { rain: false, sun: false, zero: false, img: ASSETS.iconCloud, type: 'cloud' }
];

/**
 * 12 个数据点的运行态数组
 * 每一项会在 initData() 中被扩展为：坐标、缩放、是否可见、当前所在容器等信息
 */
const items = ref([]);

/**
 * 三种筛选条件的枚举
 */
const FILTER_TYPES = ['rain', 'sun', 'zero'];

/**
 * 筛选条件 -> 节点标签图片
 */
const FILTER_IMAGES = {
  rain: ASSETS.filterRain,
  sun: ASSETS.filterSun,
  zero: ASSETS.filterZero
};

/**
 * 筛选条件 -> 中文显示文案（用于下拉菜单）
 */
const FILTER_TEXT = {
  rain: '是否降水',
  sun: '是否有阳光',
  zero: '是否零下'
};

// ====================
// 坐标与布局定义（相对决策树区域）
// ====================
/**
 * 布局坐标表（所有坐标均相对于 .tree-area）
 * 说明：
 * - root 表示顶部“待分类”容器的中心参考点
 * - lineX 表示每一层连接线的中点（用于两段式动画的中间点）
 * - node_xx / leaf_x 表示各层容器/叶子框的中心参考点（用于布局和落点）
 */
const POS = {
  root: { x: 500, y: 120 }, // 顶部初始区中心
  line1: { x: 500, y: 220 }, // 第一层连接线中点
  
  node_2_yes: { x: 260, y: 320 }, // 第二层左大框中心
  node_2_no: { x: 740, y: 320 }, // 第二层右大框中心
  
  line2_l: { x: 260, y: 440 }, // 左侧大框下连线中点
  line2_r: { x: 740, y: 440 }, // 右侧大框下连线中点

  node_3_ll: { x: 140, y: 520 }, // 第三层 左-左中框中心
  node_3_lr: { x: 380, y: 520 }, // 第三层 左-右中框中心
  node_3_rl: { x: 620, y: 520 }, // 第三层 右-左中框中心
  node_3_rr: { x: 860, y: 520 }, // 第三层 右-右中框中心

  line3_ll: { x: 140, y: 600 },
  line3_lr: { x: 380, y: 600 },
  line3_rl: { x: 620, y: 600 },
  line3_rr: { x: 860, y: 600 },

  // 最终的 8 个叶子框坐标
  leaf_1: { x: 80, y: 680 },
  leaf_2: { x: 200, y: 680 },
  leaf_3: { x: 320, y: 680 },
  leaf_4: { x: 440, y: 680 },
  leaf_5: { x: 560, y: 680 },
  leaf_6: { x: 680, y: 680 },
  leaf_7: { x: 800, y: 680 },
  leaf_8: { x: 920, y: 680 },
};

// ====================
// 决策树状态（响应式状态）
// ====================
/**
 * 决策树节点状态表
 * 字段说明：
 * - id / level：节点标识与层级
 * - selectedFilter：该节点选择的筛选依据（rain/sun/zero）
 * - availableFilters：该节点可选的筛选依据列表（会排除上层已选的条件）
 * - status：节点是否已完成分流（pending/completed）
 * - yesChild/noChild：该节点“是/否”分支指向的下一个节点/叶子
 * - linePos：该节点连接线中点位置（用于动画的中间点）
 */
const treeNodes = reactive({
  root: {
    id: 'root',
    level: 1,
    selectedFilter: '',
    availableFilters: [...FILTER_TYPES],
    status: 'pending',
    yesChild: 'node_2_yes', noChild: 'node_2_no', linePos: POS.line1
  },
  // 第二层节点将在根节点完成时动态初始化
});

/**
 * 当前分类执行到的层级
 * - 0：根节点未分流（数据还在顶部）
 * - 1：已完成第 1 层分流（数据进入左右大框）
 * - 2：已完成第 2 层分流（数据进入 4 个中框）
 * - 3：已完成第 3 层分流（数据进入 8 个叶子框）
 */
const currentStage = ref(0); // 0：未分流，1：完成第1层，2：完成第2层，3：完成第3层

/**
 * 是否正在执行动画
 * 用于：
 * - 禁止在动画过程中切换节点筛选条件
 * - 禁止重复触发“开始分类”
 */
const isAnimating = ref(false);

/**
 * 是否已完成整棵树的三层分类
 */
const isTreeCompleted = computed(() => currentStage.value === 3);

/**
 * “开始分类”按钮是否可用
 * 规则（与需求一致）：
 * - 同一层所有可操作节点都选好筛选依据后才允许点击
 */
const canStart = computed(() => {
  if (isAnimating.value) return false;
  if (currentStage.value === 0) return !!treeNodes.root?.selectedFilter;
  if (currentStage.value === 1) return !!treeNodes.node_2_yes?.selectedFilter && !!treeNodes.node_2_no?.selectedFilter;
  if (currentStage.value === 2) {
    return (
      !!treeNodes.node_3_ll?.selectedFilter &&
      !!treeNodes.node_3_lr?.selectedFilter &&
      !!treeNodes.node_3_rl?.selectedFilter &&
      !!treeNodes.node_3_rr?.selectedFilter
    );
  }
  return false;
});

// ====================
// 预测区状态
// ====================
/**
 * 右侧预测面板选择的条件值
 * 说明：预测时将读取该对象，并沿决策树路径滚动“？”图标
 */
const predictOptions = reactive({
  rain: true,
  sun: false,
  zero: false
});

/**
 * 右侧展示的预测结果文本
 */
const predictResultText = ref('');

/**
 * 预测用的动态图标（默认问号，结束后替换为对应天气图标）
 */
const predictItem = reactive({
  x: POS.root.x, y: POS.root.y, scale: 1, visible: false, img: ASSETS.iconQuestion
});

/**
 * 当前处于展开状态的节点下拉菜单对应的节点 ID
 * - 为空：表示没有任何节点菜单展开
 */
const openMenuNodeId = ref('');

/**
 * 获取筛选条件对应的中文文案
 * @param {string} filter 筛选条件 key（rain/sun/zero）
 * @returns {string} 中文文案
 */
const getFilterText = (filter) => FILTER_TEXT[filter] || '';

/**
 * 获取某个节点“下拉菜单”当前可选项（文字模式）
 * 规则：
 * - 菜单项 = 当前已选项 + 可选项（去重）
 * - 如果只有 1 个可选项，则不弹出菜单
 * @param {string} nodeId 节点ID
 * @returns {string[]} 可选的筛选条件 key 列表
 */
const getNodeMenuOptions = (nodeId) => {
  const node = treeNodes[nodeId];
  if (!node) return [];
  const options = [];
  if (node.selectedFilter) options.push(node.selectedFilter);
  node.availableFilters.forEach(f => {
    if (!options.includes(f)) options.push(f);
  });
  return options;
};

/**
 * 打开/关闭某个节点的下拉菜单
 * 规则：只允许在当前层级（currentStage）选择本层节点的依据
 * @param {string} nodeId 节点ID
 */
const toggleNodeMenu = (nodeId) => {
  if (isAnimating.value || currentStage.value === 3) return;
  const node = treeNodes[nodeId];
  if (!node || node.status === 'completed') return;
  if (nodeId === 'root' && currentStage.value !== 0) return;
  if ((nodeId === 'node_2_yes' || nodeId === 'node_2_no') && currentStage.value !== 1) return;
  if (nodeId.startsWith('node_3_') && currentStage.value !== 2) return;
  if (getNodeMenuOptions(nodeId).length <= 1) return;
  openMenuNodeId.value = openMenuNodeId.value === nodeId ? '' : nodeId;
};

/**
 * 选择节点的筛选依据
 * @param {string} nodeId 节点ID
 * @param {string} filter 筛选条件 key（rain/sun/zero）
 */
const selectNodeFilter = (nodeId, filter) => {
  const node = treeNodes[nodeId];
  if (!node) return;
  node.selectedFilter = filter;
  node.availableFilters = FILTER_TYPES.filter(f => f !== filter && !getUsedFiltersForNode(nodeId).includes(f));
  openMenuNodeId.value = '';
};

/**
 * 获取下拉菜单的显示位置
 * 说明：菜单默认显示在节点的下方
 * @param {string} nodeId 节点ID
 * @returns {Record<string, string>} 用于绑定到 style 的定位对象
 */
const getNodeMenuStyle = (nodeId) => {
  const pos = POS[nodeId];
  if (!pos) return {};
  return {
    left: `${pos.x}px`,
    top: `${pos.y + 40}px`
  };
};

/**
 * 计算某个节点所在“路径”已使用的筛选条件
 * 目的：保证“上层选择过的筛选依据，下层不再出现该筛选依据”
 * @param {string} nodeId 节点ID
 * @returns {string[]} 已使用的筛选条件 key 列表
 */
const getUsedFiltersForNode = (nodeId) => {
  if (nodeId === 'root') return [];
  if (nodeId === 'node_2_yes' || nodeId === 'node_2_no') return [treeNodes.root.selectedFilter].filter(Boolean);
  if (nodeId.startsWith('node_3_')) {
    const parentId = nodeId.startsWith('node_3_l') ? 'node_2_yes' : 'node_2_no';
    return [treeNodes.root.selectedFilter, treeNodes[parentId]?.selectedFilter].filter(Boolean);
  }
  return [];
};


// ====================
// 数据初始化与通用工具函数
// ====================
/**
 * 初始化 12 个待分类数据点
 * 行为：
 * - 生成 12 个数据点（每种天气 3 个）
 * - 随机打乱顺序，模拟“待分类”区的随机排列
 * - 将数据点布局到顶部“待分类”容器
 */
const initData = () => {
  items.value = [];
  let id = 1;
  baseData.forEach(d => {
    for (let i = 0; i < 3; i++) {
      items.value.push({ ...d, id: id++, x: POS.root.x, y: POS.root.y, scale: 1, visible: true, currentNode: 'root' });
    }
  });
  // 打乱
  items.value.sort(() => Math.random() - 0.5);
  layoutItemsInNode('root', items.value);
};

/**
 * 在指定节点容器内，将若干数据点按网格方式排布
 * 说明：
 * - 本项目使用“固定坐标 + 网格排布”方案，保证动画结束后能稳定落在框内
 * - 同一个节点内的数据越多，网格会自动扩展行列
 * @param {string} nodeId 目标节点/容器 ID
 * @param {Array} itemArray 要布局的数据点数组
 */
const layoutItemsInNode = (nodeId, itemArray) => {
  if (!itemArray.length) return;
  const center = POS[nodeId];
  if (!center) return;
  // 简单的网格布局
  const cols = Math.ceil(Math.sqrt(itemArray.length * 1.5));
  const dx = 40;
  const dy = 40;
  const startX = center.x - ((cols - 1) * dx) / 2;
  
  itemArray.forEach((item, index) => {
    const row = Math.floor(index / cols);
    const col = index % cols;
    item.x = startX + col * dx;
    item.y = center.y + row * dy - 20; // 稍微偏上一点
    item.currentNode = nodeId;
    item.scale = 1;
  });
};

/**
 * 根据筛选条件返回节点标签图片
 * @param {string} filter 筛选条件 key（rain/sun/zero）
 * @returns {string} 图片资源 URL
 */
const getFilterImage = (filter) => {
  return FILTER_IMAGES[filter] || ASSETS.waitingLabel;
};

/**
 * 判断某个数据点是否满足指定筛选条件（用于是/否分流）
 * @param {Object} data 数据点（包含 rain/sun/zero）
 * @param {string} filterType 筛选条件 key（rain/sun/zero）
 * @returns {boolean} 是否满足条件
 */
const checkCondition = (data, filterType) => {
  return !!data[filterType]; // 对应数据上的三个布尔字段
};

/**
 * 根据（降水、阳光、零下）三要素推断天气类型
 * 真值表规则：
 * - 降水=是 且 阳光=是 -> 无
 * - 降水=是 且 阳光=否 且 零下=是 -> 雪
 * - 降水=是 且 阳光=否 且 零下=否 -> 雨
 * - 降水=否 且 阳光=是 -> 晴（忽略零下）
 * - 降水=否 且 阳光=否 -> 阴（忽略零下）
 * @param {{rain:boolean, sun:boolean, zero:boolean}} attrs 三个条件值
 * @returns {'none'|'snow'|'rain'|'sun'|'cloud'} 天气类型
 */
const resolveWeatherType = (attrs) => {
  const { rain, sun, zero } = attrs;
  if (rain && sun) return 'none';
  if (rain && !sun && zero) return 'snow';
  if (rain && !sun && !zero) return 'rain';
  if (!rain && sun) return 'sun';
  return 'cloud';
};

/**
 * 将天气类型映射为底部叶子框“结果标签”图片
 * @param {'none'|'snow'|'rain'|'sun'|'cloud'} type 天气类型
 * @returns {string} 图片资源 URL
 */
const resolveWeatherImage = (type) => {
  if (type === 'none') return ASSETS.resultNone;
  if (type === 'snow') return ASSETS.resultSnow;
  if (type === 'rain') return ASSETS.resultRain;
  if (type === 'sun') return ASSETS.resultSun;
  return ASSETS.resultCloud;
};

/**
 * 将结果类型映射为预测图标要显示的图片文件名
 * @param {string} type 结果类型（none/snow/rain/sun/cloud）
 * @returns {string} 图片文件名
 */
const resolveWeatherPredictIcon = (type) => {
  if (type === 'none') return ASSETS.resultNone;
  if (type === 'snow') return ASSETS.iconSnow;
  if (type === 'rain') return ASSETS.iconRain;
  if (type === 'sun') return ASSETS.iconSun;
  return ASSETS.iconCloud;
};

/**
 * 获取某个叶子框对应的“路径定义”
 * 说明：
 * - 每个叶子框都对应一条从根节点开始的三层路径
 * - 路径项格式：[nodeId, branchYes]，branchYes 表示在该节点走“是(true)”还是“否(false)”分支
 * @param {string} leafId 叶子框 ID（leaf_1 ~ leaf_8）
 * @returns {Array<[string, boolean]>} 路径数组
 */
const getLeafPath = (leafId) => {
  const paths = {
    leaf_1: [['root', true], ['node_2_yes', true], ['node_3_ll', true]],
    leaf_2: [['root', true], ['node_2_yes', true], ['node_3_ll', false]],
    leaf_3: [['root', true], ['node_2_yes', false], ['node_3_lr', true]],
    leaf_4: [['root', true], ['node_2_yes', false], ['node_3_lr', false]],
    leaf_5: [['root', false], ['node_2_no', true], ['node_3_rl', true]],
    leaf_6: [['root', false], ['node_2_no', true], ['node_3_rl', false]],
    leaf_7: [['root', false], ['node_2_no', false], ['node_3_rr', true]],
    leaf_8: [['root', false], ['node_2_no', false], ['node_3_rr', false]]
  };
  return paths[leafId] || [];
};

/**
 * 根据叶子框路径，反推该叶子框对应的（降水/阳光/零下）三要素取值
 * 说明：
 * - 叶子框的路径只描述“每层走是/否”，并不固定哪个条件在哪一层
 * - 因此需要读取每个节点当前选择的筛选依据（selectedFilter），再把是/否映射到对应条件上
 * @param {string} leafId 叶子框 ID
 * @returns {{rain:boolean, sun:boolean, zero:boolean}} 三要素值
 */
const getLeafAttrs = (leafId) => {
  const attrs = { rain: false, sun: false, zero: false };
  const path = getLeafPath(leafId);
  path.forEach(([nodeId, branchYes]) => {
    const node = treeNodes[nodeId];
    if (!node?.selectedFilter) return;
    attrs[node.selectedFilter] = !!branchYes;
  });
  return attrs;
};

/**
 * 计算叶子框的“结果标签图片”
 * 说明：通过叶子框的路径 -> 三要素 -> 真值表 -> 标签图片 的链路得到最终结果
 * @param {string} leafId 叶子框 ID
 * @returns {string} 结果标签图片 URL
 */
const getLeafResultImage = (leafId) => {
  const attrs = getLeafAttrs(leafId);
  return resolveWeatherImage(resolveWeatherType(attrs));
};

// ====================
// 分类与动画执行
// ====================

/**
 * 点击“开始分类”
 * 规则：
 * - 当前层所有可操作节点选择完成后才能触发（由 canStart 控制）
 * - 每次点击推进一层分类：第 1 层 -> 第 2 层 -> 第 3 层
 */
const startClassification = () => {
  if (!canStart.value || isTreeCompleted.value) return;
  
  if (currentStage.value === 0) {
    executeLevel1();
  } else if (currentStage.value === 1) {
    executeLevel2();
  } else if (currentStage.value === 2) {
    executeLevel3();
  }
};

/**
 * 执行第 1 层分流（根节点 -> 左右两个大框）
 * 动画：
 * 1) 顶部待分类数据滚动到连接线中点并缩小
 * 2) 再滚动到左右大框并放大
 */
const executeLevel1 = () => {
  isAnimating.value = true;
  const root = treeNodes['root'];
  
  // 1. 移动到连接线，缩小
  items.value.forEach(item => {
    item.x = root.linePos.x;
    item.y = root.linePos.y;
    item.scale = 0.5;
  });

  // 2. 移动到第2层大框，放大
  setTimeout(() => {
    const yesItems = [];
    const noItems = [];
    
    items.value.forEach(item => {
      if (checkCondition(item, root.selectedFilter)) {
        yesItems.push(item);
      } else {
        noItems.push(item);
      }
    });

    layoutItemsInNode('node_2_yes', yesItems);
    layoutItemsInNode('node_2_no', noItems);

    // 3. 初始化第二层节点
    const level2Available = FILTER_TYPES.filter(f => f !== root.selectedFilter);
    treeNodes['node_2_yes'] = {
      id: 'node_2_yes', level: 2, 
      selectedFilter: '',
      availableFilters: [...level2Available],
      status: 'pending', yesChild: 'node_3_ll', noChild: 'node_3_lr', linePos: POS.line2_l
    };
    treeNodes['node_2_no'] = {
      id: 'node_2_no', level: 2, 
      selectedFilter: '',
      availableFilters: [...level2Available],
      status: 'pending', yesChild: 'node_3_rl', noChild: 'node_3_rr', linePos: POS.line2_r
    };
    
    root.status = 'completed';
    currentStage.value = 1;
    isAnimating.value = false;
  }, 1000);
};

/**
 * 执行第 2 层分流（两个大框 -> 4 个中框）
 * 说明：左右两个节点分别使用各自选择的筛选依据，互不影响
 */
const executeLevel2 = () => {
  isAnimating.value = true;
  const nodeYes = treeNodes['node_2_yes'];
  const nodeNo = treeNodes['node_2_no'];

  items.value.forEach(item => {
    if (item.currentNode === 'node_2_yes') {
      item.x = nodeYes.linePos.x; item.y = nodeYes.linePos.y; item.scale = 0.5;
    } else if (item.currentNode === 'node_2_no') {
      item.x = nodeNo.linePos.x; item.y = nodeNo.linePos.y; item.scale = 0.5;
    }
  });

  setTimeout(() => {
    const lists = { ll: [], lr: [], rl: [], rr: [] };
    
    items.value.forEach(item => {
      if (item.currentNode === 'node_2_yes') {
        checkCondition(item, nodeYes.selectedFilter) ? lists.ll.push(item) : lists.lr.push(item);
      } else if (item.currentNode === 'node_2_no') {
        checkCondition(item, nodeNo.selectedFilter) ? lists.rl.push(item) : lists.rr.push(item);
      }
    });

    layoutItemsInNode('node_3_ll', lists.ll);
    layoutItemsInNode('node_3_lr', lists.lr);
    layoutItemsInNode('node_3_rl', lists.rl);
    layoutItemsInNode('node_3_rr', lists.rr);

    const leftAvailable = FILTER_TYPES.filter(f => ![treeNodes.root.selectedFilter, nodeYes.selectedFilter].includes(f));
    const rightAvailable = FILTER_TYPES.filter(f => ![treeNodes.root.selectedFilter, nodeNo.selectedFilter].includes(f));

    ['node_3_ll', 'node_3_lr'].forEach(id => {
      treeNodes[id] = {
        id,
        level: 3,
        selectedFilter: leftAvailable[0] || '',
        availableFilters: leftAvailable.slice(1),
        status: 'pending',
        yesChild: id === 'node_3_ll' ? 'leaf_1' : 'leaf_3',
        noChild: id === 'node_3_ll' ? 'leaf_2' : 'leaf_4',
        linePos: POS[`line3_${id.split('_')[2]}`]
      };
    });
    ['node_3_rl', 'node_3_rr'].forEach(id => {
      treeNodes[id] = {
        id,
        level: 3,
        selectedFilter: rightAvailable[0] || '',
        availableFilters: rightAvailable.slice(1),
        status: 'pending',
        yesChild: id === 'node_3_rl' ? 'leaf_5' : 'leaf_7',
        noChild: id === 'node_3_rl' ? 'leaf_6' : 'leaf_8',
        linePos: POS[`line3_${id.split('_')[2]}`]
      };
    });

    nodeYes.status = 'completed';
    nodeNo.status = 'completed';
    currentStage.value = 2;
    isAnimating.value = false;
  }, 1000);
};

/**
 * 执行第 3 层分流（4 个中框 -> 8 个叶子框）
 * 说明：四个节点分别独立使用各自的筛选依据，把数据分配到最终落点
 */
const executeLevel3 = () => {
  isAnimating.value = true;
  
  const nodes = ['node_3_ll', 'node_3_lr', 'node_3_rl', 'node_3_rr'];
  
  items.value.forEach(item => {
    const node = treeNodes[item.currentNode];
    if (node) {
      item.x = node.linePos.x; item.y = node.linePos.y; item.scale = 0.5;
    }
  });

  setTimeout(() => {
    const leafLists = { leaf_1:[], leaf_2:[], leaf_3:[], leaf_4:[], leaf_5:[], leaf_6:[], leaf_7:[], leaf_8:[] };

    items.value.forEach(item => {
      const node = treeNodes[item.currentNode];
      if (node) {
        if (checkCondition(item, node.selectedFilter)) {
          leafLists[node.yesChild].push(item);
        } else {
          leafLists[node.noChild].push(item);
        }
      }
    });

    Object.keys(leafLists).forEach(leafId => {
      layoutItemsInNode(leafId, leafLists[leafId]);
    });

    nodes.forEach(id => treeNodes[id].status = 'completed');
    currentStage.value = 3;
    isAnimating.value = false;
  }, 1000);
};


/**
 * 重置：回到初始状态
 * 行为：
 * - 清空第 2/3 层节点状态，仅保留根节点
 * - 清空预测图标和预测结果
 * - 重新生成并排列 12 个待分类数据
 */
const resetAll = () => {
  if (isAnimating.value) return;
  // 重置状态
  currentStage.value = 0;
  
  Object.keys(treeNodes).forEach(key => {
    if (key !== 'root') delete treeNodes[key];
  });
  
  treeNodes['root'] = {
    id: 'root',
    level: 1,
    selectedFilter: '',
    availableFilters: [...FILTER_TYPES],
    status: 'pending',
    yesChild: 'node_2_yes', noChild: 'node_2_no', linePos: POS.line1
  };

  predictItem.visible = false;
  predictResultText.value = '';
  initData();
};

/**
 * 上一步
 * 说明：
 * - 当前实现采用“直接重置”策略，以避免多层回退的状态还原复杂度
 * - 如果后续需要精确回退到上一层，可在此处扩展为“回退到 currentStage-1”并回收/重排数据
 */
const goBack = () => {
  resetAll();
};


// ====================
// 右侧预测功能
// ====================

/**
 * 简单的延时工具（用于串联动画步骤）
 * @param {number} ms 毫秒
 * @returns {Promise<void>} Promise
 */
const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

/**
 * 根据右侧面板选择值，计算预测最终会落在哪一个叶子框
 * @returns {string} 叶子框 ID（leaf_1 ~ leaf_8）
 */
const getPredictionLeafId = () => {
  const root = treeNodes.root;
  const rootYes = checkCondition(predictOptions, root.selectedFilter);
  const node2Id = rootYes ? 'node_2_yes' : 'node_2_no';
  const node2 = treeNodes[node2Id];
  const node2Yes = checkCondition(predictOptions, node2.selectedFilter);
  const node3Id = node2Id === 'node_2_yes' ? (node2Yes ? 'node_3_ll' : 'node_3_lr') : (node2Yes ? 'node_3_rl' : 'node_3_rr');
  const node3 = treeNodes[node3Id];
  const node3Yes = checkCondition(predictOptions, node3.selectedFilter);
  return node3Yes ? node3.yesChild : node3.noChild;
};

/**
 * 让预测图标穿过某一个节点（执行“上->线->下”的两段式动画）
 * @param {string} nodeId 节点ID（root / node_2_x / node_3_x）
 * @returns {Promise<string>} 下一个节点/叶子ID
 */
const animatePredictThroughNode = async (nodeId) => {
  const node = treeNodes[nodeId];
  if (!node) return nodeId;
  const isYes = checkCondition(predictOptions, node.selectedFilter);
  const nextNodeId = isYes ? node.yesChild : node.noChild;

  predictItem.x = node.linePos.x;
  predictItem.y = node.linePos.y;
  predictItem.scale = 0.5;
  await sleep(900);

  const targetPos = POS[nextNodeId];
  predictItem.x = targetPos.x;
  predictItem.y = targetPos.y;
  predictItem.scale = 1;
  await sleep(900);

  return nextNodeId;
};

/**
 * 触发预测
 * 规则：
 * - 只有在整棵树分类完成后才能预测（确保路径完整、节点均有筛选依据）
 * - 预测图标从顶部出发，依次穿过 3 层节点，到达叶子框后替换为对应天气图标
 */
const startPrediction = () => {
  if (!isTreeCompleted.value) {
    alert("请先完成左侧决策树的完整分类");
    return;
  }
  if (isAnimating.value) return;
  isAnimating.value = true;
  predictResultText.value = '预测中...';
  predictItem.img = ASSETS.iconQuestion;
  predictItem.visible = true;
  predictItem.x = POS.root.x;
  predictItem.y = POS.root.y - 40;
  predictItem.scale = 1;

  (async () => {
    let next = 'root';
    next = await animatePredictThroughNode(next);
    next = await animatePredictThroughNode(next);
    await animatePredictThroughNode(next);

    const leafId = getPredictionLeafId();
    const attrs = getLeafAttrs(leafId);
    const type = resolveWeatherType(attrs);
    predictItem.img = resolveWeatherPredictIcon(type);
    predictResultText.value = type === 'none' ? '无' : type === 'snow' ? '雪' : type === 'rain' ? '雨' : type === 'sun' ? '晴' : '阴';
    isAnimating.value = false;
  })();
};

/**
 * 组件挂载后初始化待分类数据
 */
onMounted(() => {
  initData();
});

</script>

<style scoped>
.decision-tree-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  background-size: cover;
  overflow: hidden;
  font-family: sans-serif;
}

.left-panel {
  flex: 1;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.tree-area-container {
  position: relative;
  width: 1340px;
  height: 774px;
  margin-top: 40px;
}

.tree-area-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: fill;
  z-index: 1;
}

.tree-area {
  position: relative;
  z-index: 2;
  width: 1000px;
  height: 700px;
  margin: 0 auto;
}

.waiting-box {
  position: absolute;
  top: 40px;
  left: 50%;
  transform: translateX(-50%);
  width: 500px;
  display: flex;
  justify-content: center;
  z-index: 20;
}

.waiting-box-bg {
  width: 100%;
}

.waiting-badge {
  position: absolute;
  top: -20px;
  width: 160px;
}

.right-panel {
  position: relative;
  width: 440px;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.sidebar {
  width: 100px;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding-right: 20px;
  z-index: 10;
}

.sidebar-card {
  width: 60px;
  cursor: pointer;
  transition: transform 0.2s;
}
.sidebar-card:hover {
  transform: scale(1.05);
}

.right-panel-bg {
  position: absolute;
  top: 0;
  right: 0;
  width: 100%;
  height: 100%;
  object-fit: fill;
  z-index: 1;
}

.right-panel-content {
  position: relative;
  z-index: 2;
  width: 100%;
  height: 100%;
  padding: 80px 40px;
  display: flex;
  flex-direction: column;
}

.top-header {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 10;
}

.header-img {
  width: 260px;
}

/* Background Paths */
.path-bg {
  position: absolute;
  pointer-events: none;
}
.path-level-1 {
  top: 170px; left: 500px; transform: translateX(-50%); width: 520px;
}
.path-level-2 {
  top: 370px; width: 260px;
}
.path-left {
  left: 260px; transform: translateX(-50%);
}
.path-right {
  left: 740px; transform: translateX(-50%);
}

/* Filter Nodes */
.filter-node {
  position: absolute;
  transform: translateX(-50%);
  cursor: pointer;
  z-index: 30;
  transition: transform 0.2s;
}
.filter-node:hover {
  transform: translateX(-50%) scale(1.05);
}
.filter-img {
  width: 160px;
  filter: drop-shadow(0 4px 6px rgba(0,0,0,0.1));
}
.node-l1 { top: 150px; left: 500px; }
.node-l2-left { top: 350px; left: 260px; }
.node-l2-right { top: 350px; left: 740px; }

.node-l3 {
  top: 520px;
  transform: translateX(-50%) scale(0.8);
}
.node-l3:hover {
  transform: translateX(-50%) scale(0.85);
}
.node-l3-ll { left: 140px; }
.node-l3-lr { left: 380px; }
.node-l3-rl { left: 620px; }
.node-l3-rr { left: 860px; }


/* Box Containers */
.box-container {
  position: absolute;
  transform: translateX(-50%);
  display: flex;
  justify-content: center;
  z-index: 10;
}
.box-bg {
  width: 100%;
}
.box-badge {
  position: absolute;
  top: -15px;
  width: 50px;
}
.box-badge-small {
  position: absolute;
  top: -10px;
  width: 40px;
}

/* Level 2 Boxes */
.box-l2-left { top: 260px; left: 260px; width: 440px; }
.box-l2-right { top: 260px; left: 740px; width: 440px; }

/* Level 3 Boxes (Medium) */
.box-medium { width: 220px; top: 440px; }
.box-l3-ll { left: 140px; }
.box-l3-lr { left: 380px; }
.box-l3-rl { left: 620px; }
.box-l3-rr { left: 860px; }

/* Level 4 Boxes (Leaves) */
.box-leaf { width: 100px; top: 590px; }
.box-leaf-1 { left: 80px; }
.box-leaf-2 { left: 200px; }
.box-leaf-3 { left: 320px; }
.box-leaf-4 { left: 440px; }
.box-leaf-5 { left: 560px; }
.box-leaf-6 { left: 680px; }
.box-leaf-7 { left: 800px; }
.box-leaf-8 { left: 920px; }

.box-result-label {
  position: absolute;
  bottom: -20px;
  width: 60px;
}

/* Data Items */
.data-item {
  position: absolute;
  transition: all 1s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
  z-index: 20;
}
.item-icon {
  width: 36px;
  height: 36px;
  object-fit: contain;
  filter: drop-shadow(0 2px 4px rgba(0, 50, 150, 0.2));
}

.bottom-controls {
  position: absolute;
  bottom: 20px;
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  padding: 0 40px;
  z-index: 50;
}

.center-buttons {
  display: flex;
  gap: 20px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.btn {
  cursor: pointer;
  height: 50px;
  object-fit: contain;
  transition: transform 0.2s;
}
.btn:hover {
  transform: scale(1.05);
}
.btn.disabled {
  cursor: not-allowed;
  opacity: 0.6;
  transform: none;
}

.btn-restart {
  width: 140px;
}

/* Right Panel Styles */
.panel-header {
  margin-bottom: 40px;
}
.panel-title-img {
  width: 200px;
}

.panel-body {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.option-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.option-label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #2b579a;
  font-weight: bold;
  font-size: 18px;
}
.option-label-icon {
  width: 20px;
}

.toggle-group {
  display: flex;
  gap: 10px;
}

.toggle-img {
  width: 60px;
  cursor: pointer;
  transition: transform 0.1s;
}
.toggle-img:hover {
  transform: scale(1.05);
}

.divider {
  width: 100%;
  height: 1px;
  background-image: linear-gradient(to right, #ccc 50%, transparent 50%);
  background-size: 10px 1px;
  margin: 20px 0;
}

.result-row {
  margin-top: 10px;
}

.result-text {
  font-size: 24px;
  color: #4a90e2;
  font-weight: bold;
}

.panel-footer {
  margin-top: auto;
  display: flex;
  justify-content: center;
  padding-bottom: 40px;
}

.btn-predict {
  width: 200px;
  cursor: pointer;
  transition: transform 0.2s;
}
.btn-predict:hover {
  transform: scale(1.05);
}

.node-menu {
  position: absolute;
  transform: translateX(-50%);
  min-width: 180px;
  padding: 8px;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.92);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
  border: 1px solid rgba(0, 0, 0, 0.05);
  z-index: 1000;
}

.node-menu-item {
  padding: 8px 10px;
  border-radius: 8px;
  font-size: 14px;
  color: #2b579a;
  cursor: pointer;
  user-select: none;
}

.node-menu-item:hover {
  background: rgba(74, 144, 226, 0.12);
}

.node-menu-item.active {
  background: rgba(74, 144, 226, 0.18);
  font-weight: 700;
}
</style>
