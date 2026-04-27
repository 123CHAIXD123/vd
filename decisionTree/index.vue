<template>
  <div class="decision-tree-container" :style="{ backgroundImage: `url(${ASSETS.pageBg})` }">
    <!-- Left Panel: Main Tree Area -->
    <div class="left-panel">
      <!-- Title -->
      <div class="top-header">
        <img :src="ASSETS.pageTitle" alt="天气决策树" class="header-img" />
      </div>

      <!-- Decision Tree Canvas -->
      <div class="tree-area-container">
        <img :src="ASSETS.treeFrameBg" class="tree-area-bg" />
        
        <div class="tree-area" @click="openMenuNodeId = ''">
          
          <!-- Top Waiting Box -->
          <div class="waiting-box">
            <img :src="ASSETS.waitingBoxBg" class="waiting-box-bg" />
            <img :src="ASSETS.waitingLabel" class="waiting-badge" />
          </div>
        
        <!-- Connection Paths (Backgrounds) -->
        <img :src="ASSETS.pathLevel1" class="path-bg path-level-1" />
        <!-- Re-using smaller paths or scaled paths for lower levels -->
        <img :src="ASSETS.pathLevel2" class="path-bg path-level-2 path-left" v-if="treeNodes['node_2_yes']" />
        <img :src="ASSETS.pathLevel2" class="path-bg path-level-2 path-right" v-if="treeNodes['node_2_no']" />

        <!-- Nodes Configuration (Buttons) -->
        <!-- Level 1 (Root) -->
        <div class="filter-node node-l1" @click="toggleNodeMenu('root')">
          <img :src="getFilterImage(treeNodes['root'].selectedFilter)" class="filter-img" />
        </div>

        <!-- Level 2 Nodes (Left and Right) -->
        <div class="filter-node node-l2 node-l2-left" v-if="treeNodes['node_2_yes'] && treeNodes['root'].status === 'completed'" @click="toggleNodeMenu('node_2_yes')">
          <img :src="getFilterImage(treeNodes['node_2_yes'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l2 node-l2-right" v-if="treeNodes['node_2_no'] && treeNodes['root'].status === 'completed'" @click="toggleNodeMenu('node_2_no')">
          <img :src="getFilterImage(treeNodes['node_2_no'].selectedFilter)" class="filter-img" />
        </div>

        <!-- Level 3 Nodes (4 Nodes) -->
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

        <!-- Container Boxes -->
        <!-- Level 2 Boxes -->
        <div class="box-container box-l2-left">
          <img :src="ASSETS.boxYes" class="box-bg" />
          <img :src="ASSETS.badgeYes" class="box-badge" />
        </div>
        <div class="box-container box-l2-right">
          <img :src="ASSETS.boxNo" class="box-bg" />
          <img :src="ASSETS.badgeNo" class="box-badge" />
        </div>

        <!-- Level 3 Boxes (Medium) -->
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

        <!-- Level 4 Boxes (Leaves) -->
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

          <!-- Data Items (Icons) -->
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

          <!-- Prediction Item (The ? mark) -->
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

      <!-- Bottom Controls -->
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

    <!-- Right Panel: Prediction Area -->
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
    <!-- Right Sidebar: Knowledge/Thinking Cards -->
    <div class="sidebar">
      <img :src="ASSETS.sidebarKnowledge" class="sidebar-card" />
      <img :src="ASSETS.sidebarThinking" class="sidebar-card" />
    </div>

  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

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
// 数据定义
// ====================
const baseData = [
  { rain: true, sun: false, zero: true, img: ASSETS.iconSnow, type: 'snow' },
  { rain: true, sun: false, zero: false, img: ASSETS.iconRain, type: 'rain' },
  { rain: false, sun: true, zero: false, img: ASSETS.iconSun, type: 'sun' },
  { rain: false, sun: false, zero: false, img: ASSETS.iconCloud, type: 'cloud' }
];

const items = ref([]);

// 过滤选项及图片映射
const FILTER_TYPES = ['rain', 'sun', 'zero'];
const FILTER_IMAGES = {
  rain: ASSETS.filterRain,
  sun: ASSETS.filterSun,
  zero: ASSETS.filterZero
};

const FILTER_TEXT = {
  rain: '是否降水',
  sun: '是否有阳光',
  zero: '是否零下'
};

// ====================
// 坐标与布局定义 (相对 .tree-area)
// ====================
// 为了贴合设计图，定义一个非常细致的坐标网格
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
// 树结构状态 (Vue State)
// ====================
// 记录每个节点的状态: id, level, selectedFilter, availableFilters, status ('pending', 'completed'), children
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

const currentStage = ref(0); // 0: 在 root, 1: 在 level 2, 2: 在 level 3, 3: completed
const isAnimating = ref(false);

const isTreeCompleted = computed(() => currentStage.value === 3);
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

// 预测区状态
const predictOptions = reactive({
  rain: true,
  sun: false,
  zero: false
});
const predictResultText = ref('');
const predictItem = reactive({
  x: POS.root.x, y: POS.root.y, scale: 1, visible: false, img: ASSETS.iconQuestion
});

const openMenuNodeId = ref('');

/**
 * 获取筛选条件对应的中文文案
 * @param {string} filter 筛选条件 key（rain/sun/zero）
 * @returns {string} 中文文案
 */
const getFilterText = (filter) => FILTER_TEXT[filter] || '';

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

const getNodeMenuStyle = (nodeId) => {
  const pos = POS[nodeId];
  if (!pos) return {};
  return {
    left: `${pos.x}px`,
    top: `${pos.y + 40}px`
  };
};

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
// 核心逻辑函数

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

// 在指定的框内网格布局排布图标
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

const getFilterImage = (filter) => {
  return FILTER_IMAGES[filter] || ASSETS.waitingLabel;
};

// 判定数据是否符合节点的条件
const checkCondition = (data, filterType) => {
  return data[filterType]; // data.rain, data.sun, data.zero
};

const resolveWeatherType = (attrs) => {
  const { rain, sun, zero } = attrs;
  if (rain && sun) return 'none';
  if (rain && !sun && zero) return 'snow';
  if (rain && !sun && !zero) return 'rain';
  if (!rain && sun) return 'sun';
  return 'cloud';
};

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

// 动态计算叶子节点的结果图片
const getLeafResultImage = (leafId) => {
  const attrs = getLeafAttrs(leafId);
  return resolveWeatherImage(resolveWeatherType(attrs));
};

// ====================
// 分类与动画执行
// ====================

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

const goBack = () => {
  // 为了简化，这里不支持退回上一步，仅支持完全重置
  resetAll();
};


// ====================
// 右侧预测功能
// ====================

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

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
