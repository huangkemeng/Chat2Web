<template>
  <div class="h-screen flex flex-col text-gray-200 bg-dark">
    <!-- 顶部导航栏 -->
    <header class="bg-dark border-b border-gray-700 py-3 px-6 flex items-center justify-between">
      <div class="flex items-center space-x-3">
        <div class="bg-primary p-2 rounded-lg">
          <i class="fas fa-code text-white text-xl"></i>
        </div>
        <h1 class="text-xl font-bold">AI HTML生成系统</h1>
      </div>
      <div class="flex items-center space-x-4">
        <button class="tooltip bg-gray-700 hover:bg-gray-600 rounded-lg p-2 transition-colors">
          <i class="fas fa-save"></i>
          <span class="tooltip-text">保存代码</span>
        </button>
        <button class="tooltip bg-gray-700 hover:bg-gray-600 rounded-lg p-2 transition-colors">
          <i class="fas fa-download"></i>
          <span class="tooltip-text">导出HTML</span>
        </button>
        <button class="tooltip bg-gray-700 hover:bg-gray-600 rounded-lg p-2 transition-colors">
          <i class="fas fa-cog"></i>
          <span class="tooltip-text">设置</span>
        </button>
      </div>
    </header>

    <!-- 主内容区 -->
    <main class="flex-1 flex overflow-hidden">
      <!-- 左侧代码编辑器 -->
      <div ref="leftPanel" class="flex flex-col border-r border-gray-700"
           :style="{ width: `${panelWidth}%` }">
        <div class="bg-gray-800 px-4 py-2 flex items-center justify-between">
          <div class="flex space-x-2">
            <div class="w-3 h-3 bg-red-500 rounded-full"></div>
            <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
            <div class="w-3 h-3 bg-green-500 rounded-full"></div>
          </div>
          <h2 class="text-sm font-medium">代码编辑器</h2>
          <div class="flex space-x-2">
            <button class="text-gray-400 hover:text-white">
              <i class="fas fa-expand"></i>
            </button>
          </div>
        </div>
        <textarea
          v-model="htmlCode"
          class="code-editor flex-1 bg-gray-900 text-gray-200 p-4 resize-none focus:outline-none leading-relaxed"
        ></textarea>
      </div>

      <!-- 可拖拽分隔条 -->
      <div
        ref="resizeHandle"
        class="resize-handle w-1.5 h-full cursor-col-resize"
        @mousedown="startResize"
      ></div>

      <!-- 右侧预览区 -->
      <div ref="rightPanel" class="flex flex-col" :style="{ width: `${100 - panelWidth}%` }">
        <div class="bg-gray-800 px-4 py-2 flex items-center justify-between">
          <h2 class="text-sm font-medium">实时预览</h2>
          <div class="flex space-x-2">
            <button @click="updatePreview" class="text-gray-400 hover:text-white">
              <i class="fas fa-sync-alt"></i>
            </button>
            <button class="text-gray-400 hover:text-white">
              <i class="fas fa-expand"></i>
            </button>
          </div>
        </div>
        <div class="preview-container flex-1 bg-gray-800 overflow-auto relative">
          <iframe ref="previewFrame" class="w-full h-full"
                  sandbox="allow-same-origin allow-scripts"></iframe>
        </div>
      </div>
    </main>

    <!-- 悬浮需求输入框 -->
    <div
      v-if="showFloatingInput"
      class="floating-input fixed bottom-6 right-6 w-96 bg-dark border border-gray-700 rounded-xl overflow-hidden z-50"
    >
      <div class="bg-gray-800 px-4 py-3 flex items-center justify-between">
        <h3 class="font-medium">AI HTML生成器</h3>
        <button @click="showFloatingInput = false" class="text-gray-400 hover:text-white">
          <i class="fas fa-times"></i>
        </button>
      </div>
      <div class="p-4">
        <div class="mb-4">
          <label class="block text-sm font-medium mb-2">输入你的需求</label>
          <textarea
            v-model="aiInput"
            class="w-full bg-gray-800 border border-gray-700 rounded-lg p-3 text-sm focus:ring-2 focus:ring-primary focus:border-transparent"
            rows="3"
            placeholder="例如：创建一个带有标题和蓝色按钮的响应式登录表单"
          ></textarea>
        </div>
        <div v-if="aiResponse"
             class="ai-response mb-4 p-3 bg-gray-800 rounded-lg border border-gray-700 text-sm">
          <p v-html="aiResponse"></p>
        </div>
        <button
          @click="generateHtml"
          class="w-full bg-primary hover:bg-indigo-600 text-white py-2.5 rounded-lg font-medium transition-colors"
        >
          生成HTML代码
        </button>
      </div>
    </div>

    <!-- 状态栏 -->
    <footer
      class="bg-dark border-t border-gray-700 px-4 py-2 flex items-center justify-between text-xs text-gray-400">
      <div class="flex items-center space-x-4">
        <div class="flex items-center space-x-1">
          <i class="fas fa-code-branch"></i>
          <span>main</span>
        </div>
        <div class="flex items-center space-x-1">
          <i class="fas fa-circle text-green-500"></i>
          <span>AI在线</span>
        </div>
      </div>
      <div>
        <span>HTML5 | Tailwind CSS | 实时预览</span>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted, watch} from 'vue';

// 面板状态
const panelWidth = ref(50);
const leftPanel = ref<HTMLElement | null>(null);
const rightPanel = ref<HTMLElement | null>(null);
const resizeHandle = ref<HTMLElement | null>(null);

// 编辑器状态
const htmlCode = ref<string>('');
const previewFrame = ref<HTMLIFrameElement | null>(null);

// AI生成状态
const showFloatingInput = ref(true);
const aiInput = ref<string>('');
const aiResponse = ref<string>('');
const isGenerating = ref<boolean>(false);

// 初始化代码
const initCode = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI生成的页面</title>
    <script src="https://cdn.tailwindcss.com"><\\/script>
</head>
<body class="min-h-screen bg-gradient-to-br from-indigo-50 to-purple-100 flex items-center justify-center">
    <div class="max-w-md w-full p-6 bg-white rounded-xl shadow-lg">
        <div class="text-center mb-8">
            <h1 class="text-3xl font-bold text-indigo-700 mb-2">欢迎使用AI生成系统</h1>
            <p class="text-gray-600">输入你的需求，系统将自动生成HTML代码</p>
        </div>

        <div class="bg-indigo-50 p-4 rounded-lg mb-6">
            <p class="text-indigo-800">这是一个由AI生成的示例页面。在右侧的悬浮窗口中输入你的需求，系统将为你生成完整的HTML代码。</p>
        </div>

        <div class="flex justify-center">
            <button class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2.5 px-6 rounded-lg transition-colors">
                开始创作
            </button>
        </div>
    </div>
</body>
</html>`;

// 更新预览
const updatePreview = () => {
  if (!previewFrame.value) return;

  const blob = new Blob([htmlCode.value], {type: 'text/html'});
  previewFrame.value.src = URL.createObjectURL(blob);
};

// 生成HTML代码
const generateHtml = () => {
  const requirement = aiInput.value.trim();
  if (!requirement) {
    aiResponse.value = '<p class="text-red-400">请输入你的需求</p>';
    return;
  }

  isGenerating.value = true;
  aiResponse.value = '<p>AI正在思考... <i class="fas fa-spinner fa-spin ml-2"></i></p>';

  // 模拟AI处理延迟
  setTimeout(() => {
    const generatedCode = generateHTMLFromRequirement(requirement);
    htmlCode.value = generatedCode;
    updatePreview();

    aiResponse.value = '<p class="text-green-400"><i class="fas fa-check-circle mr-2"></i>HTML代码已生成！</p>';
    isGenerating.value = false;

    // 3秒后清空消息
    setTimeout(() => {
      aiResponse.value = '';
    }, 3000);
  }, 2000);
};

// 根据需求生成HTML代码
const generateHTMLFromRequirement = (requirement: string): string => {
  let htmlCode = `<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI生成的页面</title>
    <script src="https://cdn.tailwindcss.com"><\\/script>
</head>
<body class="min-h-screen bg-gradient-to-br from-indigo-50 to-purple-100">`;

  if (requirement.includes('登录') || requirement.includes('表单')) {
    htmlCode += `
    <div class="flex items-center justify-center min-h-screen">
        <div class="max-w-md w-full p-8 bg-white rounded-xl shadow-xl">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-bold text-indigo-700 mb-3">用户登录</h1>
                <p class="text-gray-600">请输入您的账号信息</p>
            </div>

            <form>
                <div class="mb-5">
                    <label class="block text-gray-700 mb-2">电子邮箱</label>
                    <input type="email" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent" placeholder="your@email.com">
                </div>

                <div class="mb-6">
                    <label class="block text-gray-700 mb-2">密码</label>
                    <input type="password" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent" placeholder="••••••••">
                </div>

                <div class="flex items-center justify-between mb-6">
                    <div class="flex items-center">
                        <input type="checkbox" class="h-4 w-4 text-indigo-600 border-gray-300 rounded">
                        <label class="ml-2 text-gray-700">记住我</label>
                    </div>
                    <a href="#" class="text-indigo-600 hover:text-indigo-800">忘记密码?</a>
                </div>

                <button class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-3 rounded-lg transition-colors">登录</button>

                <div class="mt-5 text-center">
                    <p class="text-gray-600">还没有账号? <a href="#" class="text-indigo-600 font-medium">立即注册</a></p>
                </div>
            </form>
        </div>
    </div>`;
  } else if (requirement.includes('卡片') || requirement.includes('产品')) {
    htmlCode += `
    <div class="container mx-auto py-12 px-4">
        <h1 class="text-3xl font-bold text-center mb-12 text-indigo-800">产品展示</h1>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
            <!-- 卡片 1 -->
            <div class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl transition-shadow">
                <div class="h-48 bg-gradient-to-r from-indigo-500 to-purple-500 flex items-center justify-center">
                    <span class="text-white text-4xl font-bold">AI</span>
                </div>
                <div class="p-6">
                    <h2 class="text-xl font-bold mb-2 text-gray-800">AI生成系统</h2>
                    <p class="text-gray-600 mb-4">基于人工智能的HTML代码生成工具，让网页开发变得简单高效。</p>
                    <div class="flex justify-between items-center">
                        <span class="text-2xl font-bold text-indigo-600">¥199</span>
                        <button class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg transition-colors">购买</button>
                    </div>
                </div>
            </div>

            <!-- 卡片 2 -->
            <div class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl transition-shadow">
                <div class="h-48 bg-gradient-to-r from-green-500 to-teal-500 flex items-center justify-center">
                    <span class="text-white text-4xl font-bold">UI</span>
                </div>
                <div class="p-6">
                    <h2 class="text-xl font-bold mb-2 text-gray-800">UI组件库</h2>
                    <p class="text-gray-600 mb-4">丰富的UI组件和模板，加速你的前端开发流程。</p>
                    <div class="flex justify-between items-center">
                        <span class="text-2xl font-bold text-indigo-600">¥149</span>
                        <button class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg transition-colors">购买</button>
                    </div>
                </div>
            </div>

            <!-- 卡片 3 -->
            <div class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-xl transition-shadow">
                <div class="h-48 bg-gradient-to-r from-amber-500 to-orange-500 flex items-center justify-center">
                    <span class="text-white text-4xl font-bold">API</span>
                </div>
                <div class="p-6">
                    <h2 class="text-xl font-bold mb-2 text-gray-800">API服务</h2>
                    <p class="text-gray-600 mb-4">强大的API接口，为你的应用提供数据支持和功能扩展。</p>
                    <div class="flex justify-between items-center">
                        <span class="text-2xl font-bold text-indigo-600">¥299</span>
                        <button class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg transition-colors">购买</button>
                    </div>
                </div>
            </div>
        </div>
    </div>`;
  } else {
    htmlCode += `
    <div class="container mx-auto py-16 px-4">
        <div class="text-center max-w-2xl mx-auto">
            <h1 class="text-4xl md:text-5xl font-bold mb-6 text-indigo-800">AI HTML生成器</h1>
            <p class="text-lg text-gray-700 mb-10 leading-relaxed">
                这是一个由AI生成的页面。根据您的需求"${requirement}"，我们创建了这个响应式的布局。您可以使用右侧的悬浮窗口输入新的需求，系统将实时生成相应的HTML代码。
            </p>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-12">
                <div class="bg-white p-6 rounded-xl shadow-md border border-indigo-100">
                    <div class="text-indigo-600 mb-4">
                        <i class="fas fa-bolt text-3xl"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-2">快速生成</h3>
                    <p class="text-gray-600">输入简单描述，立即获得完整的HTML代码</p>
                </div>

                <div class="bg-white p-6 rounded-xl shadow-md border border-indigo-100">
                    <div class="text-indigo-600 mb-4">
                        <i class="fas fa-sync-alt text-3xl"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-2">实时预览</h3>
                    <p class="text-gray-600">代码变化即时反映在预览窗口中</p>
                </div>

                <div class="bg-white p-6 rounded-xl shadow-md border border-indigo-100">
                    <div class="text-indigo-600 mb-4">
                        <i class="fas fa-mobile-alt text-3xl"></i>
                    </div>
                    <h3 class="text-xl font-bold mb-2">响应式设计</h3>
                    <p class="text-gray-600">生成的代码适配各种屏幕尺寸</p>
                </div>
            </div>

            <div class="flex justify-center">
                <button class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-3 px-8 rounded-lg mr-4 transition-colors">
                    开始使用
                </button>
                <button class="border border-indigo-600 text-indigo-600 hover:bg-indigo-50 font-medium py-3 px-8 rounded-lg transition-colors">
                    了解更多
                </button>
            </div>
        </div>
    </div>`;
  }

  htmlCode += `
</body>
</html>`;

  return htmlCode;
};

// 面板拖拽功能
const startResize = (e: MouseEvent) => {
  e.preventDefault();
  window.addEventListener('mousemove', resize);
  window.addEventListener('mouseup', stopResize);
};

const resize = (e: MouseEvent) => {
  if (!leftPanel.value || !rightPanel.value) return;

  const containerWidth = leftPanel.value.parentElement?.offsetWidth || 0;
  const mouseX = e.clientX;
  const leftWidth = (mouseX / containerWidth) * 100;

  if (leftWidth > 20 && leftWidth < 80) {
    panelWidth.value = leftWidth;
  }
};

const stopResize = () => {
  window.removeEventListener('mousemove', resize);
  window.removeEventListener('mouseup', stopResize);
};

// 初始化
onMounted(() => {
  htmlCode.value = initCode;
  updatePreview();
});

// 监听代码变化自动更新预览
watch(htmlCode, () => {
  updatePreview();
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css');

.resize-handle {
  background-color: #334155;
  cursor: col-resize;
  transition: background-color 0.2s;
}

.resize-handle:hover {
  background-color: #4F46E5;
}

.code-editor {
  font-family: 'Fira Code', 'Consolas', monospace;
  tab-size: 2;
}

.floating-input {
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.5);
  animation: float 6s ease-in-out infinite;
}

.preview-container {
  background-image: linear-gradient(45deg, #1E293B 25%, transparent 25%),
  linear-gradient(-45deg, #1E293B 25%, transparent 25%),
  linear-gradient(45deg, transparent 75%, #1E293B 75%),
  linear-gradient(-45deg, transparent 75%, #1E293B 75%);
  background-size: 20px 20px;
  background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
}

.ai-response {
  animation: fadeIn 0.5s ease-out;
}

.tooltip {
  position: relative;
}

.tooltip-text {
  visibility: hidden;
  width: 120px;
  background-color: #1E293B;
  color: #fff;
  text-align: center;
  border-radius: 6px;
  padding: 5px;
  position: absolute;
  z-index: 1;
  bottom: 125%;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  transition: opacity 0.3s;
}

.tooltip:hover .tooltip-text {
  visibility: visible;
  opacity: 1;
}

@keyframes float {
  0% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
  100% {
    transform: translateY(0px);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

body {
  font-family: 'Inter', sans-serif;
  background-color: #0F172A;
  overflow: hidden;
}
</style>
