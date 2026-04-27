<template>
  <div class="decision-tree-container">
    <div class="left-panel">
      <div class="top-header">
        <img src="./image/label_waiting.png" alt="等待分类" class="header-img" />
      </div>

      <!-- Decision Tree Area -->
      <div class="tree-area">
        <!-- Connecting paths -->
        <img src="./image/path_blue.png" class="path-bg" />

        <!-- Current Filter Option -->
        <div class="filter-node" @click="cycleFilter">
          <img :src="getFilterImage()" class="filter-img" />
        </div>

        <!-- Result Boxes -->
        <div class="result-boxes">
          <div class="box yes-box">
            <img src="./image/box_yes.png" class="box-bg" />
            <img src="./image/result_none.png" class="box-label" v-if="!isClassified" />
          </div>
          <div class="box no-box">
            <img src="./image/box_no.png" class="box-bg" />
            <img src="./image/result_none.png" class="box-label" v-if="!isClassified" />
          </div>
        </div>

        <!-- Data Items -->
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
          <img src="./image/btn_prev.png" class="btn btn-prev" @click="resetClassification" />
          <img
            :src="isClassified ? './image/btn_start_filled.png' : './image/btn_start_outline.png'"
            class="btn btn-start"
            @click="startClassification"
          />
        </div>
      </div>
    </div>

    <!-- Right Panel: Prediction Area -->
    <div class="right-panel">
      <div class="panel-header">
        <h2>天气预测区</h2>
      </div>
      <div class="panel-content">
        <div class="option-row">
          <span class="option-label">是否降水:</span>
          <div class="toggle-group">
            <span class="toggle-btn" :class="{ active: predictOptions.isRain === true }" @click="predictOptions.isRain = true">是</span>
            <span class="toggle-btn" :class="{ active: predictOptions.isRain === false }" @click="predictOptions.isRain = false">否</span>
          </div>
        </div>
        <div class="option-row">
          <span class="option-label">是否有阳光:</span>
          <div class="toggle-group">
            <span class="toggle-btn" :class="{ active: predictOptions.isSun === true }" @click="predictOptions.isSun = true">是</span>
            <span class="toggle-btn" :class="{ active: predictOptions.isSun === false }" @click="predictOptions.isSun = false">否</span>
          </div>
        </div>
        <div class="option-row">
          <span class="option-label">是否零下:</span>
          <div class="toggle-group">
            <span class="toggle-btn" :class="{ active: predictOptions.isZero === true }" @click="predictOptions.isZero = true">是</span>
            <span class="toggle-btn" :class="{ active: predictOptions.isZero === false }" @click="predictOptions.isZero = false">否</span>
          </div>
        </div>
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
</template>

<script setup>
import { ref, onMounted } from 'vue';

/**
 * 数据生成与初始化
 * 根据需求，共有12个数据，每个数据包含：是否降水、是否有阳光、是否零下。不同数据使用不同图片。
 * 这里构造12个数据点，分为4种天气类型，各3个。
 */
const baseData = [
  // 雪: 降水=是, 阳光=否, 零下=是
  { isRain: true, isSun: false, isZero: true, img: 'icon_snow.png', type: 'snow' },
  // 雨: 降水=是, 阳光=否, 零下=否
  { isRain: true, isSun: false, isZero: false, img: 'icon_rain.png', type: 'rain' },
  // 晴: 降水=否, 阳光=是, 零下=否
  { isRain: false, isSun: true, isZero: false, img: 'icon_sun.png', type: 'sun' },
  // 阴: 降水=否, 阳光=否, 零下=否
  { isRain: false, isSun: false, isZero: false, img: 'icon_cloud.png', type: 'cloud' }
];

const items = ref([]);

// 状态控制
const isClassified = ref(false);
// 过滤选项：'rain', 'sun', 'zero'
const filterOptions = ['rain', 'sun', 'zero'];
const currentFilterIndex = ref(0);

// 预测区状态
const predictOptions = ref({
  isRain: true,
  isSun: false,
  isZero: false
});
const predictResultText = ref('');
const isPredicting = ref(false);

const predictItem = ref({
  x: 0,
  y: 0,
  scale: 1,
  visible: false,
  img: 'icon_question.png'
});

// 坐标配置 (相对于 .tree-area)
const POS = {
  TOP_BOX: { x: 440, y: 80, width: 460, cols: 6, dx: 60, dy: 60 },
  LINE_CENTER: { x: 440, y: 300 },
  YES_BOX: { x: 200, y: 480, width: 300, cols: 4, dx: 60, dy: 60 },
  NO_BOX: { x: 680, y: 480, width: 300, cols: 4, dx: 60, dy: 60 }
};

/**
 * 获取当前选中的过滤选项对应的图片路径
 * @returns {string} 图片相对路径
 */
const getFilterImage = () => {
  const f = filterOptions[currentFilterIndex.value];
  if (f === 'rain') return './image/label_precipitation.png';
  if (f === 'sun') return './image/label_sunlight.png';
  return './image/label_below_zero.png';
};

/**
 * 切换当前的过滤条件
 */
const cycleFilter = () => {
  if (isClassified.value) return; // 已分类时不可切换
  currentFilterIndex.value = (currentFilterIndex.value + 1) % filterOptions.length;
};

/**
 * 初始化所有的待分类数据项，并打乱顺序
 */
const initItems = () => {
  const newItems = [];
  let idCounter = 1;
  baseData.forEach(d => {
    for (let i = 0; i < 3; i++) {
      newItems.push({ 
        ...d, 
        id: idCounter++, 
        isClassified: false, // 是否分类
        x: 0, 
        y: 0, 
        scale: 1, 
        visible: true 
      });
    }
  });
  // 随机打乱
  newItems.sort(() => Math.random() - 0.5);
  items.value = newItems;
  layoutTop();
};

/**
 * 将数据项布局在顶部的“等待分类”区域
 */
const layoutTop = () => {
  items.value.forEach((item, index) => {
    const row = Math.floor(index / POS.TOP_BOX.cols);
    const col = index % POS.TOP_BOX.cols;
    const startX = POS.TOP_BOX.x - (POS.TOP_BOX.cols - 1) * POS.TOP_BOX.dx / 2;
    item.x = startX + col * POS.TOP_BOX.dx;
    item.y = POS.TOP_BOX.y + row * POS.TOP_BOX.dy;
    item.scale = 1;
  });
};

/**
 * 检查数据是否符合当前的过滤条件
 * @param {Object} data 待检查的数据项
 * @param {string} filterType 过滤类型
 * @returns {boolean} 是否符合条件
 */
const checkCondition = (data, filterType) => {
  if (filterType === 'rain') return data.isRain;
  if (filterType === 'sun') return data.isSun;
  if (filterType === 'zero') return data.isZero;
  return false;
};

/**
 * 开始执行分类操作，包含向下的滚动和缩放动画
 */
const startClassification = () => {
  if (isClassified.value) return;
  isClassified.value = true;
  
  // 第一步：移动到连接线中间，缩小
  items.value.forEach(item => {
    item.x = POS.LINE_CENTER.x;
    item.y = POS.LINE_CENTER.y;
    item.scale = 0.5;
  });

  // 第二步：移动到下方框中，放大
  setTimeout(() => {
    const filterType = filterOptions[currentFilterIndex.value];
    let yesCount = 0;
    let noCount = 0;

    items.value.forEach(item => {
      item.isClassified = true; // 标记数据项已被分类
      const isYes = checkCondition(item, filterType);
      if (isYes) {
        const row = Math.floor(yesCount / POS.YES_BOX.cols);
        const col = yesCount % POS.YES_BOX.cols;
        const startX = POS.YES_BOX.x - (POS.YES_BOX.cols - 1) * POS.YES_BOX.dx / 2;
        item.x = startX + col * POS.YES_BOX.dx;
        item.y = POS.YES_BOX.y + row * POS.YES_BOX.dy;
        yesCount++;
      } else {
        const row = Math.floor(noCount / POS.NO_BOX.cols);
        const col = noCount % POS.NO_BOX.cols;
        const startX = POS.NO_BOX.x - (POS.NO_BOX.cols - 1) * POS.NO_BOX.dx / 2;
        item.x = startX + col * POS.NO_BOX.dx;
        item.y = POS.NO_BOX.y + row * POS.NO_BOX.dy;
        noCount++;
      }
      item.scale = 1;
    });
  }, 1000); // 等待1秒
};

/**
 * 重置分类状态，将数据项移回顶部区域
 */
const resetClassification = () => {
  isClassified.value = false;
  layoutTop();
};

/**
 * 重置所有状态（包括分类和预测状态）
 */
const resetAll = () => {
  resetClassification();
  predictItem.value.visible = false;
  predictResultText.value = '';
  isPredicting.value = false;
  items.value.forEach(i => i.visible = true);
};

/**
 * 开始进行天气预测，包含问号图标的滚动和最终结果替换动画
 */
const startPrediction = () => {
  if (isPredicting.value) return;
  
  // 隐藏原有数据，只展示预测图标
  items.value.forEach(i => i.visible = false);
  isPredicting.value = true;
  isClassified.value = true; // 确保树处于分类状态

  predictResultText.value = '预测中...';
  
  predictItem.value.img = 'icon_question.png';
  predictItem.value.visible = true;
  predictItem.value.x = POS.TOP_BOX.x;
  predictItem.value.y = POS.TOP_BOX.y + 30;
  predictItem.value.scale = 1;

  // 根据当前树的过滤条件进行预测路线
  const filterType = filterOptions[currentFilterIndex.value];
  const isYes = checkCondition(predictOptions.value, filterType);

  // 第一步：移动到中间，缩小
  setTimeout(() => {
    predictItem.value.x = POS.LINE_CENTER.x;
    predictItem.value.y = POS.LINE_CENTER.y;
    predictItem.value.scale = 0.5;
  }, 100);

  // 第二步：移动到目标框，放大，并显示结果
  setTimeout(() => {
    if (isYes) {
      predictItem.value.x = POS.YES_BOX.x;
      predictItem.value.y = POS.YES_BOX.y;
    } else {
      predictItem.value.x = POS.NO_BOX.x;
      predictItem.value.y = POS.NO_BOX.y;
    }
    predictItem.value.scale = 1;
  }, 1100);

  // 第三步：变为对应图片
  setTimeout(() => {
    // 逻辑判定：
    // 如果降水且零下 => 雪
    // 如果降水且不零下 => 雨
    // 如果不降水且阳光 => 晴
    // 都不满足（不降水且无阳光） => 阴
    let resultImg = 'icon_cloud.png';
    let resultText = '阴';

    if (predictOptions.value.isRain) {
      if (predictOptions.value.isZero) {
        resultImg = 'icon_snow.png';
        resultText = '雪';
      } else {
        resultImg = 'icon_rain.png';
        resultText = '雨';
      }
    } else {
      if (predictOptions.value.isSun) {
        resultImg = 'icon_sun.png';
        resultText = '晴';
      }
    }

    predictItem.value.img = resultImg;
    predictResultText.value = resultText;
    
    setTimeout(() => {
      isPredicting.value = false;
    }, 2000);
  }, 2100);
};

onMounted(() => {
  initItems();
});
</script>

<style scoped>
.decision-tree-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  background: linear-gradient(180deg, #b5d8ff 0%, #f4f9ff 100%);
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
  width: 320px;
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  margin: 20px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  border: 2px solid rgba(255, 255, 255, 0.8);
  box-shadow: 0 8px 32px rgba(0, 100, 255, 0.1);
}

.top-header {
  margin-top: 20px;
  z-index: 10;
}

.header-img {
  width: 220px;
}

.tree-area {
  position: relative;
  width: 880px;
  height: 640px;
  margin-top: 20px;
}

.path-bg {
  position: absolute;
  top: 140px;
  left: 50%;
  transform: translateX(-50%);
  width: 600px;
  pointer-events: none;
}

.filter-node {
  position: absolute;
  top: 120px;
  left: 50%;
  transform: translateX(-50%);
  cursor: pointer;
  z-index: 20;
  transition: transform 0.2s;
}
.filter-node:hover {
  transform: translateX(-50%) scale(1.05);
}
.filter-img {
  width: 220px;
}

.result-boxes {
  position: absolute;
  top: 400px;
  width: 100%;
  display: flex;
  justify-content: space-between;
}

.box {
  position: relative;
  width: 380px;
  display: flex;
  justify-content: center;
}

.box-bg {
  width: 100%;
}

.box-label {
  position: absolute;
  top: -20px;
  width: 80px;
}

.data-item {
  position: absolute;
  transition: all 1s ease-in-out;
  pointer-events: none;
}

.item-icon {
  width: 60px;
  height: 60px;
  object-fit: contain;
  filter: drop-shadow(0 4px 8px rgba(0, 50, 150, 0.2));
}

.bottom-controls {
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 0 40px;
  margin-top: auto;
  margin-bottom: 20px;
}

.center-buttons {
  display: flex;
  gap: 20px;
}

.btn {
  cursor: pointer;
  height: 60px;
  object-fit: contain;
  transition: transform 0.2s;
}
.btn:hover {
  transform: scale(1.05);
}

.btn-restart {
  width: 160px;
}

/* Right Panel Styles */
.panel-header h2 {
  color: #2b579a;
  margin: 0 0 20px 0;
  font-size: 20px;
}

.panel-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.option-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.option-label {
  color: #333;
  font-weight: bold;
  font-size: 16px;
}

.toggle-group {
  display: flex;
  gap: 10px;
}

.toggle-btn {
  padding: 6px 16px;
  border-radius: 20px;
  background: #fff;
  color: #666;
  cursor: pointer;
  border: 1px solid #ccc;
  transition: all 0.2s;
}

.toggle-btn.active {
  background: #4a90e2;
  color: #fff;
  border-color: #4a90e2;
  box-shadow: 0 2px 8px rgba(74, 144, 226, 0.4);
}

.result-row {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px dashed rgba(0,0,0,0.1);
}

.result-text {
  font-size: 24px;
  color: #e24a4a;
  font-weight: bold;
}

.panel-footer {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.btn-predict {
  width: 200px;
}
</style>
 
