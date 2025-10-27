<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>森 - iOS手机 (v2.1)</title>
    <style>
        :root {
            --phone-width: 375px;
            --phone-height: 750px;
            --primary-text: #000; --secondary-text: #8e8e93;
            --background-color: #f0f2f5; --qq-blue: #12B7F5;
            --battery-green: #34c759; --battery-red: #ff3b30;
            --message-green: #34C759; --forum-orange: #FF9500;
            --settings-gray: #8E8E93; --photos-blue: #415ED0;
            --books-orange: #FF9500; --music-pink: #FF2D55;
            --theme-purple: #BF5AF2; --jd-red: #E31436;
            --meituan-yellow: #FFC300; --eleme-blue: #0088FF;
        }

        body { display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #333; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; margin: 0; }
        .phone-container { width: var(--phone-width); height: var(--phone-height); background: #fff; border-radius: 40px; box-shadow: 0 10px 40px rgba(0,0,0,0.3); display: flex; flex-direction: column; overflow: hidden; position: relative; border: 8px solid #111; }
        .screen { width: 100%; height: 100%; display: flex; flex-direction: column; position: relative; background-color: var(--background-color); }

        /* 状态栏样式 */
        .status-bar { height: 40px; display: flex; justify-content: space-between; align-items: center; padding: 0 12px; color: var(--primary-text); font-size: 16px; font-weight: 600; position: absolute; top: 0; left: 0; right: 0; z-index: 1100; background-color: transparent; }
        .left-items, .right-items { display: flex; align-items: center; }
        .right-items { gap: 8px; }
        .time { font-weight: 700; }
        .dnd-icon { margin-left: 12px; font-size: 20px; display: none; }
        .network-container { display: flex; align-items: center; margin-right: 8px; }
        .network-type { margin-right: 8px; font-size: 14px; }
        .signal-bars { display: flex; align-items: flex-end; height: 14px; }
        .signal-bar { width: 4px; background-color: currentColor; margin-right: 1px; border-radius: 1px; }
        .signal-bar:nth-child(1) { height: 5px; } .signal-bar:nth-child(2) { height: 7px; } .signal-bar:nth-child(3) { height: 9px; } .signal-bar:nth-child(4) { height: 11px; } .signal-bar:nth-child(5) { height: 14px; margin-right: 0; }
        .battery-container { display: flex; align-items: center; }
        .battery-icon { width: 32px; height: 16px; border: 2px solid currentColor; border-radius: 5px; position: relative; overflow: hidden; }
        .battery-icon::after { content: ''; position: absolute; right: -4px; top: 4px; width: 3px; height: 8px; background-color: currentColor; border-radius: 0 3px 3px 0; }
        .battery-level { position: absolute; top: 1px; left: 1px; bottom: 1px; background-color: currentColor; border-radius: 3px; width: 80%; transition: width 0.5s ease, background-color 0.5s ease; }
        .battery-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 11px; font-weight: bold; color: #111; z-index: 2; }
        .charging .battery-level { background-color: var(--battery-green); }
        .low-battery .battery-level { background-color: var(--battery-red); }

        /* 灵动岛样式 */
        .dynamic-island { position: absolute; top: 10px; left: 50%; transform: translateX(-50%); width: 140px; height: 40px; background-color: rgba(0, 0, 0, 0.8); border-radius: 25px; display: flex; justify-content: center; align-items: center; color: white; font-size: 16px; cursor: pointer; transition: all 0.3s ease; z-index: 1099; }
        .dynamic-island.expanded { width: 320px; height: 90px; border-radius: 45px; flex-direction: column; }
        .island-content { display: flex; align-items: center; }
        .island-icon { margin-right: 10px; font-size: 18px; }
        .expanded-content { display: none; text-align: center; margin-top: 8px; font-size: 14px; }
        .expanded .expanded-content { display: block; }
        .music-controls { display: none; flex-direction: column; align-items: center; width: 100%; padding: 10px; box-sizing: border-box; }
        .music-info { font-size: 14px; margin-bottom: 8px; }
        .music-progress { width: 90%; height: 4px; background-color: rgba(255,255,255,0.3); border-radius: 3px; margin-bottom: 8px; }
        .music-progress-bar { height: 100%; width: 50%; background-color: white; border-radius: 3px; }
        .music-buttons { display: flex; justify-content: center; align-items: center; }
        .music-btn { margin: 0 8px; font-size: 18px; cursor: pointer; }

        /* 主屏幕 & Dock (已更新) */
        .home-screen { flex: 1; padding-top: 40px; padding-bottom: 100px; overflow-y: auto; display: grid; grid-template-columns: repeat(4, 1fr); grid-auto-rows: min-content; gap: 20px; padding: 60px 20px 100px; }
        .app-icon { display: flex; flex-direction: column; align-items: center; justify-content: center; cursor: pointer; }
        .app-icon-image { width: 60px; height: 60px; border-radius: 15px; display: flex; align-items: center; justify-content: center; margin-bottom: 8px; overflow: hidden; background-color: white; padding: 4px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .app-icon-image img { width: 100%; height: 100%; object-fit: cover; border-radius: 11px; }
        .app-icon-name { font-size: 12px; color: var(--primary-text); text-align: center; }
        .dock { position: absolute; bottom: 0; left: 0; right: 0; height: 100px; background-color: rgba(240, 242, 245, 0.8); backdrop-filter: blur(12px); display: flex; justify-content: center; align-items: center; padding: 0 25px; }
        .dock-apps { display: flex; justify-content: space-around; width: 100%; }
        .dock-app { display: flex; flex-direction: column; align-items: center; cursor: pointer; }
        .dock-app-icon { width: 60px; height: 60px; border-radius: 12px; display: flex; align-items: center; justify-content: center; overflow: hidden; background-color: white; padding: 4px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .dock-app-icon img { width: 100%; height: 100%; object-fit: cover; border-radius: 8px; }

        /* 底部导航栏 */
        .home-indicator { position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%); width: 120px; height: 5px; background-color: #000; border-radius: 3px; cursor: pointer; z-index: 1100; }
        
        /* 应用容器与页面 */
        .app-container, .app-page { position: absolute; inset: 0; background-color: #f5f5f5; display: none; flex-direction: column; z-index: 1000; transform: translateX(100%); transition: transform 0.3s ease-in-out; }
        .app-container.active, .app-page.active { display: flex; transform: translateX(0); }
        .app-header { height: 50px; display: flex; align-items: center; padding: 0 15px; color: var(--primary-text); position: relative; background-color: #f7f7f7; border-bottom: 1px solid #ddd; flex-shrink: 0; }
        .app-back-btn { font-size: 22px; cursor: pointer; padding: 5px 10px; margin-left: -10px; }
        .app-title { flex: 1; text-align: center; font-weight: bold; font-size: 18px; position: absolute; left: 50%; transform: translateX(-50%); }
        .app-content { flex: 1; padding: 20px; overflow-y: auto; }

        /* 设置菜单样式 (新增) */
        .settings-menu { display: flex; flex-direction: column; gap: 10px; }
        .setting-item { background-color: white; padding: 15px; border-radius: 10px; display: flex; justify-content: space-between; align-items: center; cursor: pointer; font-size: 16px; }
        .setting-item:hover { background-color: #f0f0f0; }
        .setting-item::after { content: '>'; color: #ccc; }

        /* API 设置样式 */
        .api-settings { display: flex; flex-direction: column; gap: 15px; }
        .api-settings h2 { margin: 0 0 10px 0; font-size: 22px; }
        .api-form-group { display: flex; flex-direction: column; }
        .api-form-group label { margin-bottom: 5px; font-size: 14px; color: #555; }
        .api-form-group select, .api-form-group input { padding: 10px; border: 1px solid #ccc; border-radius: 8px; font-size: 16px; }
        .api-actions { display: flex; gap: 10px; margin-top: 10px; }
        .api-btn { flex: 1; padding: 12px; border: none; border-radius: 8px; font-size: 16px; cursor: pointer; transition: background-color 0.2s; }
        .api-btn-primary { background-color: var(--qq-blue); color: white; }
        .api-btn-secondary { background-color: #e0e0e0; color: #333; }
        .api-status { margin-top: 15px; padding: 10px; border-radius: 8px; text-align: center; }
        .api-status.success { background-color: #d4edda; color: #155724; }
        .api-status.error { background-color: #f8d7da; color: #721c24; }
        .api-models-list { margin-top: 20px; border: 1px solid #ddd; border-radius: 8px; padding: 10px; max-height: 200px; overflow-y: auto; }
        .api-models-list h3 { margin: 0 0 10px 0; }
        .api-model-item { padding: 8px; border-bottom: 1px solid #eee; cursor: pointer; transition: background-color 0.2s; }
        .api-model-item:hover { background-color: #f5f5f5; }
        .api-model-item.selected { background-color: #e6f7ff; border-left: 3px solid var(--qq-blue); }
        .api-model-item:last-child { border-bottom: none; }
        .api-model-name { font-weight: bold; }
        .api-model-id { font-size: 12px; color: #777; }
        .current-model { margin-top: 15px; padding: 10px; background-color: #f0f8ff; border-radius: 8px; border: 1px solid #d0e8ff; }
        .current-model h4 { margin: 0 0 5px 0; font-size: 16px; }
        .current-model p { margin: 0; font-size: 14px; color: #555; }

        /* 世界书菜单样式 */
        .worldbook-menu {
            display: flex;
            flex-direction: column;
            gap: 20px;
            margin-bottom: 20px;
        }
        
        .menu-section {
            background-color: white;
            border-radius: 12px;
            padding: 15px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        .menu-section h2 {
            margin: 0 0 15px 0;
            font-size: 18px;
            color: #333;
        }
        
        .menu-item {
            display: flex;
            align-items: center;
            padding: 12px 10px;
            border-radius: 8px;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        
        .menu-item:hover {
            background-color: #f5f5f5;
        }
        
        .menu-icon {
            font-size: 24px;
            margin-right: 12px;
            width: 30px;
            text-align: center;
        }
        
        .menu-text {
            font-size: 16px;
            color: #333;
        }
        
        /* 世界书内容区域 */
        .worldbook-content {
            display: none;
            background-color: white;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        /* 数据管理样式 */
        .data-management {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        
        .data-actions {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        
        .data-btn {
            padding: 12px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        
        .data-btn-primary {
            background-color: var(--qq-blue);
            color: white;
        }
        
        .data-btn-secondary {
            background-color: #e0e0e0;
            color: #333;
        }
        
        .file-input {
            display: none;
        }
    </style>
</head>
<body>
    <div class="phone-container">
        <div class="screen">
            <!-- 状态栏 -->
            <div class="status-bar">
                <div class="left-items"><div class="time"></div><div class="dnd-icon">🌙</div></div>
                <div class="right-items">
                    <div class="network-container"><div class="network-type">5G</div><div class="signal-bars"><div class="signal-bar"></div><div class="signal-bar"></div><div class="signal-bar"></div><div class="signal-bar"></div><div class="signal-bar"></div></div></div>
                    <div class="battery-container"><div class="battery-icon"><div class="battery-level"></div><div class="battery-text"></div></div></div>
                </div>
            </div>
            
            <!-- 灵动岛 -->
            <div class="dynamic-island"><div class="island-content"><div class="island-icon">ㅇㅅㅇ</div><div>森</div></div></div>

            <!-- 主屏幕 -->
            <div class="home-screen"></div>

            <!-- 底部导航栏 -->
            <div class="dock"><div class="dock-apps"></div></div>
            
            <!-- Home指示器 -->
            <div class="home-indicator" onclick="returnToHome()"></div>
        
            <!-- 应用容器 -->
            <div class="app-container" id="messageContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('messageContainer')">←</div><div class="app-title">信息</div></div><div class="app-content"></div></div>
            <div class="app-container" id="forumContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('forumContainer')">←</div><div class="app-title">论坛八卦</div></div><div class="app-content"></div></div>
            
            <!-- 设置应用 (重构) -->
            <div class="app-container" id="settingsContainer">
                <div class="app-header"><div class="app-back-btn" onclick="closeApp('settingsContainer')">←</div><div class="app-title">设置</div></div>
                <div class="app-content">
                    <div class="settings-menu">
                        <div class="setting-item" onclick="openAppPage('apiSettingsPage')">API与模型设置</div>
                        <div class="setting-item" onclick="openAppPage('dataManagementPage')">数据管理</div>
                        <div class="setting-item">通用</div>
                        <div class="setting-item">显示与亮度</div>
                        <div class="setting-item">墙纸</div>
                    </div>
                </div>
            </div>
            
            <!-- API设置页面 (新增) -->
            <div class="app-page" id="apiSettingsPage">
                <div class="app-header"><div class="app-back-btn" onclick="closeAppPage('apiSettingsPage')">←</div><div class="app-title">API设置</div></div>
                <div class="app-content">
                    <div class="api-settings">
                        <!-- 当前模型显示 -->
                        <div class="current-model" id="currentModelDisplay" style="display: none;">
                            <h4>当前选定模型</h4>
                            <p id="currentModelName">无</p>
                        </div>
                        
                        <div class="api-form-group">
                            <label for="apiProvider">API接口模型供应商</label>
                            <select id="apiProvider">
                                <option value="openai">OpenAI</option>
                                <option value="anthropic">Anthropic</option>
                                <option value="google">Google Gemini</option>
                                <option value="custom">自定义</option>
                            </select>
                        </div>
                        <div class="api-form-group">
                            <label for="apiUrl">填写URL链接</label>
                            <input type="url" id="apiUrl" placeholder="https://api.openai.com">
                        </div>
                        <div class="api-form-group">
                            <label for="apiKey">输入API密钥</label>
                            <input type="password" id="apiKey" placeholder="输入您的API密钥">
                        </div>
                        <div class="api-actions">
                            <button class="api-btn api-btn-primary" onclick="fetchModels()">获取模型</button>
                            <button class="api-btn api-btn-secondary" onclick="testApiConnection()">测试模型</button>
                        </div>
                        <div class="api-status" id="apiStatus"></div>
                        <div class="api-models-list" id="apiModelsList" style="display: none;"></div>
                        <div class="api-actions" style="margin-top: 20px;">
                            <button class="api-btn api-btn-primary" onclick="saveApiSettings()">保存设置</button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 数据管理页面 (新增) -->
            <div class="app-page" id="dataManagementPage">
                <div class="app-header"><div class="app-back-btn" onclick="closeAppPage('dataManagementPage')">←</div><div class="app-title">数据管理</div></div>
                <div class="app-content">
                    <div class="data-management">
                        <h2>数据备份与恢复</h2>
                        <div class="data-actions">
                            <button class="data-btn data-btn-primary" onclick="exportAllData()">导出全部数据</button>
                            <button class="data-btn data-btn-secondary" onclick="document.getElementById('importFile').click()">导入数据</button>
                            <input type="file" id="importFile" class="file-input" accept=".json" onchange="importData(this)">
                        </div>
                    </div>
                </div>
            </div>

            <div class="app-container" id="phoneContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('phoneContainer')">←</div><div class="app-title">电话</div></div><div class="app-content"></div></div>
            <div class="app-container" id="photosContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('photosContainer')">←</div><div class="app-title">备忘录</div></div><div class="app-content"></div></div>
            
            <!-- 世界书应用 (更新) -->
            <div class="app-container" id="booksContainer">
                <div class="app-header">
                    <div class="app-back-btn" onclick="closeApp('booksContainer')">←</div>
                    <div class="app-title">世界书</div>
                </div>
                <div class="app-content">
                    <div class="worldbook-menu">
                        <!-- 世界书功能菜单 -->
                        <div class="menu-section">
                            <h2>世界书管理</h2>
                            <div class="menu-item" onclick="openWorldbookTab('create')">
                                <div class="menu-icon">📖</div>
                                <div class="menu-text">创建新世界书</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('library')">
                                <div class="menu-icon">📚</div>
                                <div class="menu-text">世界书库</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('import')">
                                <div class="menu-icon">⬇️</div>
                                <div class="menu-text">导入世界书</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('export')">
                                <div class="menu-icon">⬆️</div>
                                <div class="menu-text">导出世界书</div>
                            </div>
                        </div>

                        <!-- 世界书编辑功能 -->
                        <div class="menu-section">
                            <h2>编辑工具</h2>
                            <div class="menu-item" onclick="openWorldbookTab('characters')">
                                <div class="menu-icon">👥</div>
                                <div class="menu-text">角色管理</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('locations')">
                                <div class="menu-icon">🗺️</div>
                                <div class="menu-text">地点管理</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('timeline')">
                                <div class="menu-icon">⏳</div>
                                <div class="menu-text">时间线</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('relations')">
                                <div class="menu-icon">🔗</div>
                                <div class="menu-text">关系图</div>
                            </div>
                        </div>

                        <!-- 世界书设置 -->
                        <div class="menu-section">
                            <h2>设置</h2>
                            <div class="menu-item" onclick="openWorldbookTab('settings')">
                                <div class="menu-icon">⚙️</div>
                                <div class="menu-text">世界书设置</div>
                            </div>
                            <div class="menu-item" onclick="openWorldbookTab('templates')">
                                <div class="menu-icon">📝</div>
                                <div class="menu-text">模板管理</div>
                            </div>
                        </div>
                    </div>

                    <!-- 世界书内容区域 -->
                    <div class="worldbook-content" id="worldbookContent">
                        <!-- 动态加载的内容将显示在这里 -->
                    </div>
                </div>
            </div>

            <div class="app-container" id="qqContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('qqContainer')">←</div><div class="app-title">QQ</div></div><div class="app-content"></div></div>
            <div class="app-container" id="musicContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('musicContainer')">←</div><div class="app-title">音乐</div></div><div class="app-content"></div></div>
            <div class="app-container" id="themeContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('themeContainer')">←</div><div class="app-title">美化</div></div><div class="app-content"></div></div>
            <div class="app-container" id="jdContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('jdContainer')">←</div><div class="app-title">网购</div></div><div class="app-content"></div></div>
            <div class="app-container" id="meituanContainer"><div class="app-header"><div class="app-back-btn" onclick="closeApp('meituanContainer')">←</div><div class="app-title">外卖</div></div><div class="app-content"></div></div>
        </div>
    </div>
    
    <script>
        // --- SYSTEM DATA ---
        let systemData = {
            battery: { level: 80, isCharging: false },
            settings: { 
                apiProvider: 'openai', 
                apiKey: '', 
                apiUrl: '',
                selectedModel: null
            },
            worldbooks: {
                library: [],
                current: null
            }
        };

        // --- SYSTEM UI FUNCTIONS ---
        function updateTime() {
            const now = new Date();
            const hours = now.getHours();
            const minutes = now.getMinutes().toString().padStart(2, '0');
            const ampm = hours >= 12 ? '下午' : '上午';
            const displayHours = hours % 12 || 12;
            document.querySelector('.time').textContent = `${ampm}${displayHours}:${minutes}`;
        }

        function updateBattery() {
            const batteryIcon = document.querySelector('.battery-icon');
            const batteryLevelEl = document.querySelector('.battery-level');
            const batteryText = document.querySelector('.battery-text');
            if (!systemData.battery.isCharging && systemData.battery.level > 0) systemData.battery.level -= 0.05;
            batteryLevelEl.style.width = `${systemData.battery.level}%`;
            batteryText.textContent = Math.round(systemData.battery.level);
            batteryIcon.classList.toggle('low-battery', !systemData.battery.isCharging && systemData.battery.level <= 20);
        }

        // --- APP NAVIGATION ---
        function openApp(appId) {
            const appContainer = document.getElementById(appId);
            if(appContainer) appContainer.classList.add('active');
        }

        function closeApp(appId) {
            const appContainer = document.getElementById(appId);
            if(appContainer) appContainer.classList.remove('active');
        }

        function openAppPage(pageId) {
            const page = document.getElementById(pageId);
            if(page) page.classList.add('active');
        }

        function closeAppPage(pageId) {
            const page = document.getElementById(pageId);
            if(page) page.classList.remove('active');
        }

        function returnToHome() {
            document.querySelectorAll('.app-container.active, .app-page.active').forEach(el => el.classList.remove('active'));
        }

        // --- API FUNCTIONS (REAL IMPLEMENTATION) ---
        async function fetchModels() {
            const provider = document.getElementById('apiProvider').value;
            let apiUrl = document.getElementById('apiUrl').value.trim();
            const apiKey = document.getElementById('apiKey').value.trim();
            const statusEl = document.getElementById('apiStatus');
            const modelsListEl = document.getElementById('apiModelsList');
            
            modelsListEl.style.display = 'none';
            modelsListEl.innerHTML = '';
            if (!apiUrl || !apiKey) {
                statusEl.textContent = '错误：请填写完整的API URL和密钥。';
                statusEl.className = 'api-status error';
                return;
            }
            
            statusEl.textContent = '正在获取模型列表...';
            statusEl.className = 'api-status';

            // 智能处理URL
            if (apiUrl.endsWith('/')) apiUrl = apiUrl.slice(0, -1);
            if (provider === 'openai' && !apiUrl.endsWith('/v1')) apiUrl += '/v1';

            let requestUrl = `${apiUrl}/models`;
            let headers = { 'Content-Type': 'application/json' };

            // 根据供应商设置不同的认证头
            if (provider === 'openai' || provider === 'custom') {
                headers['Authorization'] = `Bearer ${apiKey}`;
            } else if (provider === 'anthropic') {
                headers['x-api-key'] = apiKey;
                headers['anthropic-version'] = '2023-06-01'; // Anthropic需要版本头
            } else if (provider === 'google') {
                requestUrl = `${apiUrl}/v1beta/models?key=${apiKey}`;
                headers = {}; // Google Gemini Key在URL中
            }

            try {
                const response = await fetch(requestUrl, { headers });
                if (!response.ok) {
                    const errorData = await response.text();
                    throw new Error(`请求失败 (${response.status}): ${errorData}`);
                }
                const data = await response.json();
                
                let models = [];
                if (provider === 'openai' || provider === 'custom') models = data.data || [];
                else if (provider === 'anthropic') models = data.data || []; // Anthropic新版API也是data
                else if (provider === 'google') models = data.models || [];

                if (models.length > 0) {
                    modelsListEl.innerHTML = '<h3>可用模型列表 (点击选择)</h3>';
                    models.forEach(model => {
                        const modelId = model.id || model.name;
                        const modelName = model.id || model.name || model.displayName;
                        const modelEl = document.createElement('div');
                        modelEl.className = 'api-model-item';
                        modelEl.dataset.modelId = modelId;
                        modelEl.dataset.modelName = modelName;
                        modelEl.innerHTML = `<div class="api-model-name">${modelName.split('/').pop()}</div><div class="api-model-id">ID: ${modelId}</div>`;
                        
                        // 添加点击事件以选择模型
                        modelEl.addEventListener('click', () => {
                            // 移除之前的选择
                            document.querySelectorAll('.api-model-item.selected').forEach(item => {
                                item.classList.remove('selected');
                            });
                            
                            // 标记当前选择
                            modelEl.classList.add('selected');
                            
                            // 保存到系统设置
                            systemData.settings.selectedModel = {
                                id: modelId,
                                name: modelName
                            };
                            
                            // 更新当前模型显示
                            updateCurrentModelDisplay();
                        });
                        
                        modelsListEl.appendChild(modelEl);
                    });
                    modelsListEl.style.display = 'block';
                    statusEl.textContent = `成功获取 ${models.length} 个模型，请点击选择一个模型`;
                    statusEl.className = 'api-status success';
                } else {
                    statusEl.textContent = '警告：API成功响应，但未找到可用模型。';
                    statusEl.className = 'api-status error';
                }
            } catch (error) {
                statusEl.textContent = `获取模型失败: ${error.message}`;
                statusEl.className = 'api-status error';
                console.error('获取模型失败详情:', error);
            }
        }

        // 更新当前模型显示
        function updateCurrentModelDisplay() {
            const currentModelDisplay = document.getElementById('currentModelDisplay');
            const currentModelName = document.getElementById('currentModelName');
            
            if (systemData.settings.selectedModel) {
                currentModelName.textContent = systemData.settings.selectedModel.name;
                currentModelDisplay.style.display = 'block';
            } else {
                currentModelDisplay.style.display = 'none';
            }
        }

        async function testApiConnection() {
            const statusEl = document.getElementById('apiStatus');
            statusEl.textContent = '正在测试连接...';
            statusEl.className = 'api-status';
            await fetchModels(); // 复用获取模型的逻辑来测试
            if (statusEl.classList.contains('success')) {
                statusEl.textContent = 'API连接测试成功！';
            }
        }

        function saveApiSettings() {
            systemData.settings.apiProvider = document.getElementById('apiProvider').value;
            systemData.settings.apiUrl = document.getElementById('apiUrl').value.trim();
            systemData.settings.apiKey = document.getElementById('apiKey').value.trim();
            
            const statusEl = document.getElementById('apiStatus');
            
            // 检查是否已选择模型
            if (systemData.settings.selectedModel) {
                statusEl.textContent = `API设置已保存，已选择模型: ${systemData.settings.selectedModel.name}`;
            } else {
                statusEl.textContent = 'API设置已保存，但未选择模型。';
            }
            
            statusEl.className = 'api-status success';
            
            // 更新当前模型显示
            updateCurrentModelDisplay();
            
            // 保存到本地存储
            saveSystemData();
        }

        // --- 世界书功能 ---
        function openWorldbookTab(tabName) {
            // 隐藏所有内容区域
            document.querySelectorAll('.worldbook-content').forEach(el => {
                el.style.display = 'none';
            });
            
            // 显示选中的内容区域
            const contentEl = document.getElementById('worldbookContent');
            contentEl.style.display = 'block';
            
            // 根据tabName加载不同内容
            switch(tabName) {
                case 'create':
                    contentEl.innerHTML = `
                        <h3>创建新世界书</h3>
                        <div class="form-group">
                            <label>世界书名称</label>
                            <input type="text" id="worldbookName" placeholder="输入世界书名称">
                        </div>
                        <div class="form-group">
                            <label>描述</label>
                            <textarea id="worldbookDesc" placeholder="输入世界书描述"></textarea>
                        </div>
                        <button class="btn-primary" onclick="createNewWorldbook()">创建</button>
                    `;
                    break;
                    
                case 'library':
                    contentEl.innerHTML = `
                        <h3>世界书库</h3>
                        <div class="worldbook-list" id="worldbookList">
                            <!-- 世界书列表将通过JS动态加载 -->
                        </div>
                    `;
                    loadWorldbookLibrary();
                    break;
                    
                case 'import':
                    contentEl.innerHTML = `
                        <h3>导入世界书</h3>
                        <p>从文件导入世界书数据</p>
                        <button class="btn-primary" onclick="document.getElementById('worldbookImportFile').click()">选择文件</button>
                        <input type="file" id="worldbookImportFile" class="file-input" accept=".json" onchange="importWorldbook(this)">
                    `;
                    break;
                    
                case 'export':
                    contentEl.innerHTML = `
                        <h3>导出世界书</h3>
                        <p>将当前世界书导出为文件</p>
                        <button class="btn-primary" onclick="exportCurrentWorldbook()">导出</button>
                    `;
                    break;
                    
                // 其他tab的实现类似...
            }
        }
        
        function createNewWorldbook() {
            const name = document.getElementById('worldbookName').value;
            const desc = document.getElementById('worldbookDesc').value;
            
            if (!name) {
                alert('请输入世界书名称');
                return;
            }
            
            const newWorldbook = {
                id: Date.now().toString(),
                name: name,
                description: desc,
                createdAt: new Date().toISOString(),
                characters: [],
                locations: [],
                timeline: []
            };
            
            systemData.worldbooks.library.push(newWorldbook);
            systemData.worldbooks.current = newWorldbook.id;
            
            // 保存数据
            saveSystemData();
            
            alert(`世界书 "${name}" 创建成功！`);
        }
        
        function loadWorldbookLibrary() {
            const listEl = document.getElementById('worldbookList');
            
            if (systemData.worldbooks.library.length === 0) {
                listEl.innerHTML = '<p>暂无世界书，请先创建一个</p>';
                return;
            }
            
            listEl.innerHTML = '';
            
            systemData.worldbooks.library.forEach(worldbook => {
                const itemEl = document.createElement('div');
                itemEl.className = 'worldbook-item';
                itemEl.innerHTML = `
                    <div class="worldbook-title">${worldbook.name}</div>
                    <div class="worldbook-desc">${worldbook.description || '无描述'}</div>
                    <div class="worldbook-actions">
                        <button onclick="openWorldbook('${worldbook.id}')">打开</button>
                        <button onclick="editWorldbook('${worldbook.id}')">编辑</button>
                    </div>
                `;
                listEl.appendChild(itemEl);
            });
        }
        
        function openWorldbook(id) {
            const worldbook = systemData.worldbooks.library.find(wb => wb.id === id);
            if (!worldbook) return;
            
            systemData.worldbooks.current = id;
            
            // 更新UI显示当前世界书
            document.getElementById('worldbookContent').innerHTML = `
                <h3>${worldbook.name}</h3>
                <p>${worldbook.description || '无描述'}</p>
                <div class="worldbook-stats">
                    <div>角色: ${worldbook.characters.length}</div>
                    <div>地点: ${worldbook.locations.length}</div>
                    <div>事件: ${worldbook.timeline.length}</div>
                </div>
            `;
        }
        
        function editWorldbook(id) {
            const worldbook = systemData.worldbooks.library.find(wb => wb.id === id);
            if (!worldbook) return;
            
            document.getElementById('worldbookContent').innerHTML = `
                <h3>编辑世界书: ${worldbook.name}</h3>
                <div class="form-group">
                    <label>名称</label>
                    <input type="text" id="editWorldbookName" value="${worldbook.name}">
                </div>
                <div class="form-group">
                    <label>描述</label>
                    <textarea id="editWorldbookDesc">${worldbook.description || ''}</textarea>
                </div>
                <button class="btn-primary" onclick="saveWorldbookChanges('${id}')">保存</button>
            `;
        }
        
        function saveWorldbookChanges(id) {
            const worldbook = systemData.worldbooks.library.find(wb => wb.id === id);
            if (!worldbook) return;
            
            const newName = document.getElementById('editWorldbookName').value;
            const newDesc = document.getElementById('editWorldbookDesc').value;
            
            if (!newName) {
                alert('世界书名称不能为空');
                return;
            }
            
            worldbook.name = newName;
            worldbook.description = newDesc;
            
            saveSystemData();
            
            alert('世界书修改已保存');
            loadWorldbookLibrary();
        }
        
        function importWorldbook(input) {
            const file = input.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const importedData = JSON.parse(e.target.result);
                    
                    // 验证数据格式
                    if (!importedData.id || !importedData.name) {
                        throw new Error('导入的文件格式不正确');
                    }
                    
                    // 检查是否已存在相同ID的世界书
                    const existingIndex = systemData.worldbooks.library.findIndex(wb => wb.id === importedData.id);
                    
    if(现有索引> = 0) {
                        if (confirm('已存在相同ID的世界书，是否覆盖？')) {
                            systemData.worldbooks.library[existingIndex] = importedData;
                        }
                    } else {
                        systemData.worldbooks.library.push(importedData);
                    }
                    
                    saveSystemData();
                    loadWorldbookLibrary();
                    
                    alert('世界书导入成功！');
                } catch (error) {
                    alert(`导入失败: ${error.message}`);
                    console.error('导入世界书错误:', error);
                }
                
                // 重置文件输入，允许重复导入同一文件
                input.value = '';
            };
            reader.onerror = function() {
                alert('读取文件失败');
            };
            reader.readAsText(file);
        }
        
        function exportCurrentWorldbook() {
            if (!systemData.worldbooks.current) {
                alert('请先打开一个世界书');
                return;
            }
            
            const worldbook = systemData.worldbooks.library.find(wb => wb.id === systemData.worldbooks.current);
            if (!worldbook) return;
            
            // 创建下载链接
            const dataStr = JSON.stringify(worldbook, null, 2);
            const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
            const exportFileName = `世界书_${worldbook.name}_${new Date().toISOString().slice(0,10)}.json`;
            
            // 触发下载
            const linkElement = document.createElement('a');
            linkElement.setAttribute('href', dataUri);
            linkElement.setAttribute('download', exportFileName);
            linkElement.click();
        }

        // --- 数据管理功能 ---
        function exportAllData() {
            // 创建包含所有系统数据的对象
            const exportData = {
                systemData: systemData,
                timestamp: new Date().toISOString()
            };
            
            // 转换为JSON字符串
            const dataStr = JSON.stringify(exportData, null, 2);
            
            // 创建下载链接
            const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
            const exportFileName = `森手机数据备份_${new Date().toISOString().slice(0, 10)}.json`;
            
            // 触发下载
            const linkElement = document.createElement('a');
            linkElement.setAttribute('href', dataUri);
            linkElement.setAttribute('download', exportFileName);
            linkElement.click();
            
            // 显示成功消息
            showAlert('数据导出成功！');
        }

        function importData(input) {
            const file = input.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const importedData = JSON.parse(e.target.result);
                    
                    // 验证数据格式
                    if (!importedData.systemData || !importedData.systemData.settings) {
                        throw new Error('导入的文件格式不正确');
                    }
                    
                    // 显示确认对话框
                    if (confirm('确定要导入数据吗？这将覆盖当前所有设置。')) {
                        // 导入数据
                        systemData = importedData.systemData;
                        
                        // 更新UI
                        updateUIFromSystemData();
                        
                        // 显示成功消息
                        showAlert('数据导入成功！');
                    }
                    
                    // 重置文件输入，允许重复导入同一文件
                    input.value = '';
                } catch (error) {
                    showAlert(`导入失败: ${error.message}`);
                    console.error('导入数据错误:', error);
                }
            };
            reader.onerror = function() {
                showAlert('读取文件失败');
            };
            reader.readAsText(file);
        }

        function updateUIFromSystemData() {
            // 更新API设置表单
            document.getElementById('apiProvider').value = systemData.settings.apiProvider;
            document.getElementById('apiUrl').value = systemData.settings.apiUrl;
            document.getElementById('apiKey').value = systemData.settings.apiKey;
            
            // 更新当前模型显示
            updateCurrentModelDisplay();
        }

        function showAlert(message) {
            const statusEl = document.getElementById('apiStatus');
            statusEl.textContent = message;
            statusEl.className = 'api-status success';
            setTimeout(() => {
                if (statusEl.textContent === message) {
                    statusEl.textContent = '';
                    statusEl.className = 'api-status';
                }
            }, 3000);
        }

        // 保存系统数据到本地存储
        function saveSystemData() {
            try {
                localStorage.setItem('systemData', JSON.stringify(systemData));
            } catch (e) {
                console.log('无法保存到本地存储:', e);
            }
        }

        // --- INITIALIZATION ---
        document.addEventListener('DOMContentLoaded', () => {
            const appData = [
                { id: 'messageContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/YOpv/1186X1080/Image_1761364260618.png', name: '信息' },
                { id: 'forumContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251021/KpZf/1079X1083/Image_1760785735649.png', name: '论坛八卦' },
                { id: 'settingsContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/DWyy/1015X861/Image_1761364260122.png', name: '设置' },
                { id: 'phoneContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/7NYe/1208X1072/Image_1761364261254.png', name: '电话' },
                { id: 'photosContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/OJ0P/920X1024/Image_1760785734981.png', name: '备忘录' },
                { id: 'booksContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251021/yD6d/742X1024/Image_1760785734886.png', name: '世界书' },
                { id: 'qqContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/qu4r/527X693/Image_1761364259317.png', name: 'QQ' },
                { id: 'musicContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/C1S1/863X793/Image_1761364257242.png', name: '音乐' },
                { id: 'themeContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/wdgd/643X630/Image_1761364259392.png', name: '美化' },
                { id: 'jdContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/h86b/640X594/Image_1761364260149.png', name: '网购' },
                { id: 'meituanContainer', icon: 'https://tc.z.wiki/autoupload/f/_dUEGnKEJQzmc-cKlsXN5NiO_OyvX7mIgxFBfDMDErs/20251025/L6Oz/985X1112/Image_1761364257742.png', name: '外卖' },
            ];
            
    const home screen = document . query selector('。主屏幕’)；
    const dock应用程序=文档。查询选择器(.dock-apps’)；
    const dockAppIds =['phone container','messageContainer','photosContainer','containers settings'];

    appData.forEach(app => {
    const isDockApp = dockAppIds.includes(app.id);
    const container = isDockApp?DockApps:主屏幕；
    const El = 文件。createelement('div');
    el.className = isDockApp?“dock-app”: “app-icon”;
    艾尔。onclick =()= > 打开应用程序(app)。id);
    el.innerHTML =
    < div class="${isDockApp？dock-app-icon ':' app-icon-image ' } " >
    < img src = " $ { app . icon } " alt = " $ { app . name } " >
    </div >
    ${!isDockApp？`< div class = " app-icon-name " >＄{ app . name } </div > `: ' ' }
                `;
    集装箱。appendchild(El)；
            });

            // 尝试从本地存储加载数据
    尝试{
    const保存的数据=本地存储。getitem(“系统数据”)；
    if(savedData)
    系统数据= JSON。解析(保存的数据)；
                }
    捕捉(五)
    console.log('无法从本地存储加载数据:’，e)；
            }

    // Laden der gespeicherten API-Einstellungen in das Formular
    updateUIFromSystemData()；

    更新时间()；
    update battery()；
    setInterval(updateTime，1000)；
    setInterval(updateBattery，5000)；
    const现有索引=系统数据。世界图书。图书馆。查找索引(WB = > WB。id = = =导入的数据。id)；
    </script >
</身体>
</超文本标记语言>
