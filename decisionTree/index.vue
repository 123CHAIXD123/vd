<template>
  <div class="decision-tree-container">
    <!-- Left Panel: Main Tree Area -->
    <div class="left-panel">
      <!-- Title -->
      <div class="top-header">
        <img src="./image/label_waiting.png" alt="等待分类" class="header-img" />
      </div>

      <!-- Decision Tree Canvas -->
      <div class="tree-area">
        
        <!-- Connection Paths (Backgrounds) -->
        <img src="./image/Group 2174.png" class="path-bg path-level-1" />
        <!-- Re-using smaller paths or scaled paths for lower levels -->
        <img src="./image/path_blue.png" class="path-bg path-level-2 path-left" v-if="treeNodes['node_2_yes']" />
        <img src="./image/path_blue.png" class="path-bg path-level-2 path-right" v-if="treeNodes['node_2_no']" />

        <!-- Nodes Configuration (Buttons) -->
        <!-- Level 1 (Root) -->
        <div class="filter-node node-l1" @click="cycleFilter('root')">
          <img :src="getFilterImage(treeNodes['root'].selectedFilter)" class="filter-img" />
        </div>

        <!-- Level 2 Nodes (Left and Right) -->
        <div class="filter-node node-l2 node-l2-left" v-if="treeNodes['node_2_yes'] && treeNodes['root'].status === 'completed'" @click="cycleFilter('node_2_yes')">
          <img :src="getFilterImage(treeNodes['node_2_yes'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l2 node-l2-right" v-if="treeNodes['node_2_no'] && treeNodes['root'].status === 'completed'" @click="cycleFilter('node_2_no')">
          <img :src="getFilterImage(treeNodes['node_2_no'].selectedFilter)" class="filter-img" />
        </div>

        <!-- Level 3 Nodes (4 Nodes) -->
        <div class="filter-node node-l3 node-l3-ll" v-if="treeNodes['node_3_ll'] && treeNodes['node_2_yes'].status === 'completed'">
          <img :src="getFilterImage(treeNodes['node_3_ll'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-lr" v-if="treeNodes['node_3_lr'] && treeNodes['node_2_yes'].status === 'completed'">
          <img :src="getFilterImage(treeNodes['node_3_lr'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-rl" v-if="treeNodes['node_3_rl'] && treeNodes['node_2_no'].status === 'completed'">
          <img :src="getFilterImage(treeNodes['node_3_rl'].selectedFilter)" class="filter-img" />
        </div>
        <div class="filter-node node-l3 node-l3-rr" v-if="treeNodes['node_3_rr'] && treeNodes['node_2_no'].status === 'completed'">
          <img :src="getFilterImage(treeNodes['node_3_rr'].selectedFilter)" class="filter-img" />
        </div>


        <!-- Container Boxes -->
        <!-- Level 2 Boxes -->
        <div class="box-container box-l2-left">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge" /> <!-- '是' badge -->
        </div>
        <div class="box-container box-l2-right">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge" /> <!-- '否' badge -->
        </div>

        <!-- Level 3 Boxes (Medium) -->
        <div class="box-container box-l3-ll box-medium">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge" />
        </div>
        <div class="box-container box-l3-lr box-medium">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge" />
        </div>
        <div class="box-container box-l3-rl box-medium">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge" />
        </div>
        <div class="box-container box-l3-rr box-medium">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge" />
        </div>

        <!-- Level 4 Boxes (Leaves) -->
        <div class="box-container box-leaf box-leaf-1">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_1')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-2">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_2')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-3">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_3')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-4">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_4')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-5">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_5')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-6">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_6')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-7">
          <img src="./image/box_yes.png" class="box-bg" />
          <img src="./image/Group 2255.png" class="box-badge-small" />
          <img :src="getLeafResultImage('leaf_7')" class="box-result-label" v-if="isTreeCompleted" />
        </div>
        <div class="box-container box-leaf box-leaf-8">
          <img src="./image/box_no.png" class="box-bg" />
          <img src="./image/Group 2256.png" class="box-badge-small" />
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
          <img :src="`./image/${item.img}`" class="item-icon" />
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
          <img :src="`./image/${predictItem.img}`" class="item-icon" />
        </div>
      </div>

      <!-- Bottom Controls -->
      <div class="bottom-controls">
        <img src="./image/btn_restart.png" class="btn btn-restart" @click="resetAll" />
        <div class="center-buttons">
          <img src="./image/btn_prev.png" class="btn btn-prev" @click="goBack" />
          <img
            :src="isTreeCompleted ? './image/btn_start_filled.png' : './image/btn_start_outline.png'"
            class="btn btn-start"
            @click="startClassification"
          />
        </div>
      </div>
    </div>

    <!-- Right Panel: Prediction Area -->
    <div class="right-panel">
      <img src="./image/Rectangle 881.png" class="right-panel-bg" />
      <div class="right-panel-content">
        <div class="panel-header">
          <img src="./image/Group 2243.png" alt="天气预测区" class="panel-title-img" />
        </div>
        
        <div class="panel-body">
          <div class="option-row">
            <span class="option-label">是否降水:</span>
            <div class="toggle-group">
              <img :src="predictOptions.rain === true ? './image/Group 2246.png' : './image/Group 2247.png'" class="toggle-img" @click="predictOptions.rain = true" />
              <img :src="predictOptions.rain === false ? './image/Group 2248.png' : './image/Group 2249.png'" class="toggle-img" @click="predictOptions.rain = false" />
            </div>
          </div>
          <div class="option-row">
            <span class="option-label">是否有阳光:</span>
            <div class="toggle-group">
              <img :src="predictOptions.sun === true ? './image/Group 2246.png' : './image/Group 2247.png'" class="toggle-img" @click="predictOptions.sun = true" />
              <img :src="predictOptions.sun === false ? './image/Group 2248.png' : './image/Group 2249.png'" class="toggle-img" @click="predictOptions.sun = false" />
            </div>
          </div>
          <div class="option-row">
            <span class="option-label">是否零下:</span>
            <div class="toggle-group">
              <img :src="predictOptions.zero === true ? './image/Group 2246.png' : './image/Group 2247.png'" class="toggle-img" @click="predictOptions.zero = true" />
              <img :src="predictOptions.zero === false ? './image/Group 2248.png' : './image/Group 2249.png'" class="toggle-img" @click="predictOptions.zero = false" />
            </div>
          </div>
          <div class="divider"></div>
          <div class="option-row result-row">
            <span class="option-label">预测结果:</span>
            <span class="result-text">{{ predictResultText }}</span>
          </div>
        </div>
        <div class="panel-footer">
          <img src="./image/btn_restart_predict.png" class="btn btn-predict" @click="startPrediction" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

// ====================
// 数据定义
// ====================
const baseData = [
  { rain: true, sun: false, zero: true, img: 'icon_snow.png', type: 'snow' },
  { rain: true, sun: false, zero: false, img: 'icon_rain.png', type: 'rain' },
  { rain: false, sun: true, zero: false, img: 'icon_sun.png', type: 'sun' },
  { rain: false, sun: false, zero: false, img: 'icon_cloud.png', type: 'cloud' }
];

const items = ref([]);

// 过滤选项及图片映射
const FILTER_TYPES = ['rain', 'sun', 'zero'];
const FILTER_IMAGES = {
  'rain': './image/label_precipitation.png',
  'sun': './image/label_sunlight.png',
  'zero': './image/label_below_zero.png',
  'unselected': './image/label_waiting.png' // Fallback
};

// ====================
// 坐标与布局定义 (相对 .tree-area)
// ====================
// 为了贴合设计图，定义一个非常细致的坐标网格
const POS = {
  root: { x: 500, y: 50 }, // 顶部初始区中心
  line1: { x: 500, y: 150 }, // 第一层连接线中点
  
  node_2_yes: { x: 260, y: 250 }, // 第二层左大框中心
  node_2_no: { x: 740, y: 250 }, // 第二层右大框中心
  
  line2_l: { x: 260, y: 350 }, // 左侧大框下连线中点
  line2_r: { x: 740, y: 350 }, // 右侧大框下连线中点

  node_3_ll: { x: 140, y: 430 }, // 第三层 左-左中框中心
  node_3_lr: { x: 380, y: 430 }, // 第三层 左-右中框中心
  node_3_rl: { x: 620, y: 430 }, // 第三层 右-左中框中心
  node_3_rr: { x: 860, y: 430 }, // 第三层 右-右中框中心

  line3_ll: { x: 140, y: 500 },
  line3_lr: { x: 380, y: 500 },
  line3_rl: { x: 620, y: 500 },
  line3_rr: { x: 860, y: 500 },

  // 最终的 8 个叶子框坐标
  leaf_1: { x: 80, y: 570 },
  leaf_2: { x: 200, y: 570 },
  leaf_3: { x: 320, y: 570 },
  leaf_4: { x: 440, y: 570 },
  leaf_5: { x: 560, y: 570 },
  leaf_6: { x: 680, y: 570 },
  leaf_7: { x: 800, y: 570 },
  leaf_8: { x: 920, y: 570 },
};

// ====================
// 树结构状态 (Vue State)
// ====================
// 记录每个节点的状态: id, level, selectedFilter, availableFilters, status ('pending', 'completed'), children
const treeNodes = reactive({
  root: {
    id: 'root', level: 1, selectedFilter: 'rain', availableFilters: ['sun', 'zero'], status: 'pending',
    yesChild: 'node_2_yes', noChild: 'node_2_no', linePos: POS.line1
  },
  // 第二层节点将在根节点完成时动态初始化
});

const currentStage = ref(0); // 0: 在 root, 1: 在 level 2, 2: 在 level 3, 3: completed
const isAnimating = ref(false);

const isTreeCompleted = computed(() => currentStage.value === 3);

// 预测区状态
const predictOptions = reactive({
  rain: true,
  sun: false,
  zero: false
});
const predictResultText = ref('');
const predictItem = reactive({
  x: POS.root.x, y: POS.root.y, scale: 1, visible: false, img: 'icon_question.png'
});


// ====================
// 核心逻辑函数
// ====================

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
  return FILTER_IMAGES[filter] || FILTER_IMAGES['unselected'];
};

// 切换某个节点的过滤条件
const cycleFilter = (nodeId) => {
  if (isAnimating.value || currentStage.value === 3) return;
  const node = treeNodes[nodeId];
  if (!node || node.status === 'completed') return;

  // 获取所有可用的选项 (包含当前已选的)
  const options = [node.selectedFilter, ...node.availableFilters];
  const idx = options.indexOf(node.selectedFilter);
  const nextFilter = options[(idx + 1) % options.length];
  
  // 更新 selectedFilter，并将原来的 filter 放回 available
  node.availableFilters = options.filter(f => f !== nextFilter);
  node.selectedFilter = nextFilter;
};

// 判定数据是否符合节点的条件
const checkCondition = (data, filterType) => {
  return data[filterType]; // data.rain, data.sun, data.zero
};

// 动态计算叶子节点的结果图片
const getLeafResultImage = (leafId) => {
  // 在当前逻辑中，需要追溯从根到该叶子的完整路径条件来推断结果
  // 但为了简化和稳定，这里使用“查看该叶子框内实际落入的数据类型”来决定图片。
  // 如果叶子框是空的，显示无。如果混杂（理应不会），显示第一个。
  const itemsInLeaf = items.value.filter(i => i.currentNode === leafId);
  if (itemsInLeaf.length === 0) return './image/result_none.png';
  
  const type = itemsInLeaf[0].type;
  if (type === 'snow') return './image/result_snow.png';
  if (type === 'rain') return './image/result_rain.png';
  if (type === 'sun') return './image/result_sun.png';
  if (type === 'cloud') return './image/result_cloud.png';
  return './image/result_none.png';
};

// ====================
// 分类与动画执行
// ====================

const startClassification = () => {
  if (isAnimating.value || isTreeCompleted.value) return;
  
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
    treeNodes['node_2_yes'] = {
      id: 'node_2_yes', level: 2, 
      selectedFilter: root.availableFilters[0], 
      availableFilters: root.availableFilters.slice(1),
      status: 'pending', yesChild: 'node_3_ll', noChild: 'node_3_lr', linePos: POS.line2_l
    };
    treeNodes['node_2_no'] = {
      id: 'node_2_no', level: 2, 
      selectedFilter: root.availableFilters[0], 
      availableFilters: root.availableFilters.slice(1),
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

    // 自动为第3层分配最后一个剩余的条件
    ['node_3_ll', 'node_3_lr'].forEach(id => {
      treeNodes[id] = {
        id, level: 3, selectedFilter: nodeYes.availableFilters[0], availableFilters: [], status: 'pending',
        yesChild: id === 'node_3_ll' ? 'leaf_1' : 'leaf_3',
        noChild: id === 'node_3_ll' ? 'leaf_2' : 'leaf_4',
        linePos: POS[`line3_${id.split('_')[2]}`]
      };
    });
    ['node_3_rl', 'node_3_rr'].forEach(id => {
      treeNodes[id] = {
        id, level: 3, selectedFilter: nodeNo.availableFilters[0], availableFilters: [], status: 'pending',
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
    id: 'root', level: 1, selectedFilter: 'rain', availableFilters: ['sun', 'zero'], status: 'pending',
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

const runPredictionStep = (nodeId, delayBase) => {
  const node = treeNodes[nodeId];
  if (!node) return;

  const isYes = checkCondition(predictOptions, node.selectedFilter);
  const nextNodeId = isYes ? node.yesChild : node.noChild;

  // 移动到线
  setTimeout(() => {
    predictItem.x = node.linePos.x;
    predictItem.y = node.linePos.y;
    predictItem.scale = 0.5;
  }, delayBase);

  // 移动到下一个框
  setTimeout(() => {
    const targetPos = POS[nextNodeId];
    predictItem.x = targetPos.x;
    predictItem.y = targetPos.y;
    predictItem.scale = 1;
    
    // 如果还没到底，递归下一步
    if (treeNodes[nextNodeId]) {
      runPredictionStep(nextNodeId, 0); // 递归调用时 delayBase 为 0，因为在自己的 setTimeout 里面
    } else {
      // 到底了，显示结果
      setTimeout(() => {
        // 根据真值表判定结果 (和左侧树解耦，直接算)
        let resultImg = 'icon_cloud.png';
        let resultText = '阴';

        if (predictOptions.rain && predictOptions.sun) {
           resultImg = 'result_none.png'; resultText = '无';
        } else if (predictOptions.rain && !predictOptions.sun && predictOptions.zero) {
           resultImg = 'icon_snow.png'; resultText = '雪';
        } else if (predictOptions.rain && !predictOptions.sun && !predictOptions.zero) {
           resultImg = 'icon_rain.png'; resultText = '雨';
        } else if (!predictOptions.rain && predictOptions.sun) {
           resultImg = 'icon_sun.png'; resultText = '晴';
        } else if (!predictOptions.rain && !predictOptions.sun) {
           resultImg = 'icon_cloud.png'; resultText = '阴';
        }

        predictItem.img = resultImg;
        predictResultText.value = resultText;
        isAnimating.value = false;
      }, 500);
    }
  }, delayBase + 1000);
};

const startPrediction = () => {
  if (!isTreeCompleted.value) {
    alert("请先完成左侧决策树的完整分类");
    return;
  }
  if (isAnimating.value) return;
  isAnimating.value = true;
  
  predictResultText.value = '预测中...';
  predictItem.img = 'icon_question.png';
  predictItem.visible = true;
  predictItem.x = POS.root.x;
  predictItem.y = POS.root.y - 40;
  predictItem.scale = 1;

  // 开始递归导航
  runPredictionStep('root', 500);
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
  background: url('./image/Slice 176.png') no-repeat center center;
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

.right-panel {
  position: relative;
  width: 440px;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
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
  width: 180px;
}

.tree-area {
  position: relative;
  width: 1000px;
  height: 700px;
  margin-top: 40px;
}

/* Background Paths */
.path-bg {
  position: absolute;
  pointer-events: none;
}
.path-level-1 {
  top: 100px; left: 500px; transform: translateX(-50%); width: 520px;
}
.path-level-2 {
  top: 300px; width: 260px;
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
.node-l1 { top: 80px; left: 500px; }
.node-l2-left { top: 280px; left: 260px; }
.node-l2-right { top: 280px; left: 740px; }

.node-l3 {
  top: 450px;
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
.box-l2-left { top: 190px; left: 260px; width: 440px; }
.box-l2-right { top: 190px; left: 740px; width: 440px; }

/* Level 3 Boxes (Medium) */
.box-medium { width: 220px; top: 370px; }
.box-l3-ll { left: 140px; }
.box-l3-lr { left: 380px; }
.box-l3-rl { left: 620px; }
.box-l3-rr { left: 860px; }

/* Level 4 Boxes (Leaves) */
.box-leaf { width: 100px; top: 520px; }
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
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  padding: 0 40px;
  margin-top: auto;
  margin-bottom: 20px;
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
  color: #2b579a;
  font-weight: bold;
  font-size: 18px;
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
</style>