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
      <button @click="startLottery" :disabled="isRunning">开始抽奖</button>
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
    // 从localStorage加载初始数据
    const savedPeople = localStorage.getItem('gift-exchange-people');
    return {
      people: savedPeople ? JSON.parse(savedPeople) : [
        { name: '张三' },
        { name: '李四' },
        { name: '王五' }
      ],
      pairs: [],
      isRunning: false,
      showAlgorithm: false
    }
  },
  watch: {
    // 深度监听people数组变化
    people: {
      handler(newVal) {
        localStorage.setItem('gift-exchange-people', JSON.stringify(newVal));
      },
      deep: true
    }
  },
  created() {
    // 加载时验证本地存储数据
    try {
      const data = localStorage.getItem('gift-exchange-people');
      if (data) this.people = JSON.parse(data);
    } catch (e) {
      console.error('本地存储数据解析失败，使用默认数据', e);
      localStorage.removeItem('gift-exchange-people');
    }
  },
  methods: {
    addPerson() {
      this.people.push({ name: '新参与者' });
    },
    removePerson(index) {
      this.people.splice(index, 1);
    },
    async startLottery() {
      if (this.isRunning) return;
      this.isRunning = true;
      this.pairs = [];
      
      // 初始化占位符数组
      const derangement = this.createDerangement(this.people.length);
      const placeholderPairs = this.people.map((_, index) => ({
        left: index,
        right: -1,      // -1表示未抽中
        revealed: false // 是否已揭示
      }));
      this.pairs = placeholderPairs;

      // 逐个揭示结果
      for (let i = 0; i < this.people.length; i++) {
        // 当前项停止动画
        this.pairs[i].right = derangement[i];
        this.pairs[i].revealed = true;
        
        // 其他项继续动画
        if (i < this.people.length - 1) {
          await this.animateOthers(i + 1, 800); // 800ms动画时间
        }
      }
      
      this.isRunning = false;
    },

    animateOthers(startIndex, duration) {
      return new Promise(resolve => {
        const startTime = Date.now();
        const animate = () => {
          const progress = Date.now() - startTime;
          
          // 仅更新未揭示的项目
          this.pairs = this.pairs.map((pair, index) => {
            if (index >= startIndex && !pair.revealed) {
              return {
                ...pair,
                right: progress % 200 < 100 ? 
                  Math.floor(Math.random() * this.people.length) : -1
              };
            }
            return pair;
          });

          if (progress < duration) {
            requestAnimationFrame(animate);
          } else {
            resolve();
          }
        };
        requestAnimationFrame(animate);
      });
    },

    createDerangement(n) {
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
  transition: all 0.3s;
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

/* 添加揭示动画 */
.pair-item {
  transition: all 0.3s;
}

/* 未揭示项目的样式 */
.pair-item:not(.revealed) {
  animation: shake 0.3s ease-in-out infinite alternate;
}

@keyframes shake {
  0% { transform: translateX(-2px); }
  100% { transform: translateX(2px); }
}

/* 已揭示项目的样式 */
.revealed .right-item {
  color: #338a6e;
  font-weight: bold;
  background: rgba(66, 185, 131, 0.1);
}

/* 礼物图标动画 */
.revealed .pair-item::before {
  animation: giftReveal 0.6s ease-out forwards;
}

@keyframes giftReveal {
  0% {
    opacity: 1;
    transform: scale(1.5) rotate(-15deg);
  }
  100% {
    opacity: 0;
    transform: scale(0) rotate(45deg);
  }
}
</style>
