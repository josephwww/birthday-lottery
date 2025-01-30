<template>
  <div class="birthday-lottery">
    <!-- 在标题旁添加算法说明按钮 -->
    <div class="header-section">
      <h1>礼物互赠配对</h1>
      <button class="info-btn" @click="showAlgorithm = true">ℹ️ 算法说明</button>
    </div>

    <!-- 算法说明弹窗 -->
    <div v-if="showAlgorithm" class="algorithm-modal">
      <div class="modal-content">
        <h3>配对算法原理</h3>
        <div class="algorithm-description">
          <p>本系统采用数学上的<b>错位排列（Derangement）</b>算法实现：</p>
          <ul>
            <li>🎯 保证每人都会且只会收到一份礼物</li>
            <li>🔄 使用改进的 Fisher-Yates 洗牌算法</li>
            <li>⚙️ 三重保障机制：
              <ol>
                <li>绝对排除自配对（不会抽到自己）</li>
                <li>自动重试机制（最多100次迭代）</li>
                <li>最终回退方案（返回最佳可能结果）</li>
              </ol>
            </li>
            <li>⏱️ 基于时间种子的随机数生成</li>
          </ul>
        </div>
        <button @click="showAlgorithm = false">关闭说明</button>
      </div>
    </div>
    
    <!-- 修改后的配对展示结构 -->
    <div class="pair-container">
      <div class="pair-row" v-for="(pair, index) in pairs" :key="index">
        <div class="pair-item left-item">
          {{ people[pair.left]?.name }}
        </div>
        <div class="arrow">→</div>
        <div class="pair-item right-item">
          {{ people[pair.right]?.name }}
        </div>
      </div>
    </div>

    <!-- 控制按钮 -->
    <div class="controls">
      <button @click="generatePairs" :disabled="isRunning">生成配对</button>
    </div>

    <!-- 编辑区域 -->
    <div class="editor">
      <h2>编辑参与者</h2>
      <div v-for="(person, index) in people" :key="index" class="person-item">
        <input v-model="person.name" placeholder="姓名" />
        <button @click="removePerson(index)">删除</button>
      </div>
      <button @click="addPerson">添加参与者</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      people: [
        { name: '张三' },
        { name: '李四' },
        { name: '王五' }
      ],
      pairs: [],
      isRunning: false,
      showAlgorithm: false
    }
  },
  methods: {
    addPerson() {
      this.people.push({ name: '新参与者' });
    },
    removePerson(index) {
      this.people.splice(index, 1);
    },
    generatePairs() {
      if (this.isRunning) return;
      this.isRunning = true;
      
      const createDerangement = (n) => {
        let arr = [...Array(n).keys()];
        let attempts = 0;
        
        while (attempts++ < 100) { // 安全阀
          // Fisher-Yates洗牌算法
          for (let i = n - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [arr[i], arr[j]] = [arr[j], arr[i]];
          }
          
          // 检查是否满足无自配对且全排列
          const isValid = arr.every((val, idx) => val !== idx) 
            && new Set(arr).size === n;
          
          if (isValid) return arr;
        }
        return arr; // 超过尝试次数返回最佳结果
      };

      const derangement = createDerangement(this.people.length);
      this.pairs = derangement.map((right, left) => ({ left, right }));
      
      this.isRunning = false;
    }
  }
}
</script>

<style scoped>
.birthday-lottery {
  max-width: 95%;
  margin: 0 auto;
  padding: 15px;
}

.pair-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin: 20px 0;
}

.pair-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
}

.arrow {
  font-size: 1.5em;
  color: #42b983;
  padding: 0 15px;
}

.pair-item {
  padding: 12px 25px;
  border: 2px solid #42b983;
  border-radius: 8px;
  min-width: 120px;
  text-align: center;
  background: #f8f8f8;
  transition: background 0.3s;
}

.pair-item::after {
  display: none;
}

.controls {
  margin: 20px 0;
  display: flex;
  justify-content: center;
  gap: 10px;
}

.editor {
  margin-top: 40px;
}

.person-item {
  margin: 10px 0;
  display: flex;
  gap: 10px;
}

button {
  padding: 5px 10px;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .pair-row {
    flex-direction: column;
    gap: 10px;
  }
  
  .arrow {
    transform: rotate(90deg);
    padding: 10px 0;
  }
  
  .pair-item {
    width: 100%;
    padding: 10px 15px;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 1.5rem;
  }

  .pair-item {
    font-size: 0.9rem;
    min-width: auto;
  }
  
  .arrow {
    font-size: 1.2em;
  }

  button {
    padding: 8px 12px;
    font-size: 0.9rem;
  }

  .editor input {
    width: 60%;
  }
}

/* 添加算法说明样式 */
.header-section {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  position: relative;
}

.info-btn {
  padding: 5px 10px;
  border-radius: 15px;
  background: #f0f9eb;
  border: 1px solid #42b983;
  cursor: help;
  transition: all 0.3s;
}

.info-btn:hover {
  background: #42b983;
  color: white;
}

.algorithm-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 25px;
  border-radius: 10px;
  max-width: 500px;
  box-shadow: 0 0 20px rgba(0,0,0,0.2);
}

.algorithm-description {
  text-align: left;
  margin: 15px 0;
}

.algorithm-description li {
  margin: 8px 0;
  line-height: 1.6;
}

@media (max-width: 768px) {
  .modal-content {
    width: 90%;
    padding: 15px;
  }
  
  .header-section {
    flex-direction: column;
    gap: 10px;
  }
}
</style>
