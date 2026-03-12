<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Forex Trading Platform</title>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f5f7fa;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
        }
        .auth-container, .recharge-container {
            max-width: 500px;
            margin: 50px auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 15px rgba(0,0,0,0.05);
        }
        .trading-container, .admin-container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 15px rgba(0,0,0,0.05);
            display: none;
        }
        #kline-chart {
            width: 100%;
            height: 400px;
            margin: 20px 0;
        }
        .order-item, .user-item {
            border-bottom: 1px solid #eee;
            padding: 10px 0;
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 1px solid #eee;
        }
        .pnl-positive { color: #28a745; font-weight: bold; }
        .pnl-negative { color: #dc3545; font-weight: bold; }
        .pnl-neutral { color: #6c757d; }
        .recharge-card, .price-management-card {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
        }
        .address-item {
            word-break: break-all;
            padding: 8px 0;
            font-family: Consolas, monospace;
        }
        .recharge-amount-display {
            background: #e9ecef;
            padding: 10px;
            border-radius: 5px;
            margin: 15px 0;
            font-size: 18px;
            font-weight: bold;
        }
        .admin-card {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
        }
        .pending-recharge-item, .pending-settle-item, .pending-order-item {
            padding: 10px;
            border: 1px solid #dee2e6;
            border-radius: 5px;
            margin: 8px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .user-detail-item {
            margin: 8px 0;
        }
        .order-detail-item {
            padding: 8px;
            border: 1px solid #eee;
            border-radius: 4px;
            margin: 8px 0;
        }
        .loading-text {
            color: #0d6efd;
            font-style: italic;
        }
        .price-input-group {
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .price-input-group label {
            min-width: 100px;
            font-weight: 600;
        }
        .price-input-group input {
            flex: 1;
            max-width: 200px;
        }
        .loading-overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255,255,255,0.8);
            z-index: 9999;
            justify-content: center;
            align-items: center;
            font-size: 20px;
            color: #0d6efd;
        }
        .admin-user-data-editor {
            margin: 15px 0;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        .admin-order-editor {
            margin: 10px 0;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        .admin-settle-editor {
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            margin: 10px 0;
        }
        .disabled-btn {
            background-color: #6c757d !important;
            border-color: #6c757d !important;
            cursor: not-allowed;
            opacity: 0.7;
        }
        .pnl-info {
            padding: 8px;
            border-radius: 4px;
            margin: 8px 0;
            background-color: #f8f9fa;
            border-left: 4px solid #0d6efd;
        }
    </style>
</head>
<body>
    <!-- 加载遮罩层 -->
    <div class="loading-overlay" id="loadingOverlay">Please wait...</div>

    <!-- 登录/注册页面（移除重置按钮） -->
    <div class="auth-container" id="authPage">
        <h3 class="text-center mb-4">Forex Trading Platform</h3>
        <ul class="nav nav-tabs mb-4">
            <li class="nav-item">
                <button class="nav-link active" onclick="switchTab('login')">Login</button>
            </li>
            <li class="nav-item">
                <button class="nav-link" onclick="switchTab('register')">Register</button>
            </li>
        </ul>

        <div id="loginForm">
            <div class="mb-3">
                <label class="form-label">Username</label>
                <input type="text" class="form-control" id="loginUsername" placeholder="Enter username">
            </div>
            <div class="mb-3">
                <label class="form-label">Password</label>
                <input type="password" class="form-control" id="loginPassword" placeholder="Enter password">
            </div>
            <button class="btn btn-primary w-100" onclick="userLogin()">Login</button>
        </div>

        <div id="registerForm" style="display: none;">
            <div class="mb-3">
                <label class="form-label">Username</label>
                <input type="text" class="form-control" id="regUsername" placeholder="Create username">
            </div>
            <div class="mb-3">
                <label class="form-label">Email</label>
                <input type="email" class="form-control" id="regEmail" placeholder="Enter email">
            </div>
            <div class="mb-3">
                <label class="form-label">Password</label>
                <input type="password" class="form-control" id="regPassword" placeholder="Create password">
            </div>
            <button class="btn btn-success w-100" onclick="userRegister()">Register</button>
        </div>
    </div>

    <!-- 用户交易主页面 -->
    <div class="trading-container" id="tradingPage">
        <div class="header">
            <h4>Trading Dashboard</h4>
            <div>
                <span id="currentUserName"></span>
                <button class="btn btn-sm btn-outline-danger ms-2" onclick="logout()">Logout</button>
            </div>
        </div>

        <div class="row mb-4">
            <div class="col-md-6">
                <h5>Account Balance: $<span id="userBalance" class="text-success">0.00</span></h5>
                <h6>Total P/L: <span id="totalPnl" class="pnl-neutral">$0.00</span></h6>
                <!-- 余额不足提示 -->
                <p id="balanceWarning" class="text-danger" style="display: none;">Your balance is 0! Please recharge to place orders.</p>
            </div>
            <div class="col-md-6">
                <select class="form-select w-50 float-end" id="assetSelect" onchange="updateKline()">
                    <option value="gold">Gold (XAU/USD)</option>
                    <option value="silver">Silver (XAG/USD)</option>
                    <option value="eurusd">EUR/USD</option>
                </select>
            </div>
        </div>

        <!-- 充值模块 -->
        <div class="recharge-card">
            <h5>Account Recharge</h5>
            <div class="row align-items-center">
                <div class="col-md-8">
                    <div class="input-group">
                        <span class="input-group-text">$</span>
                        <input type="number" class="form-control" id="rechargeAmountInput" placeholder="Enter recharge amount (minimum $1)" min="1" step="0.01">
                    </div>
                    <small class="text-muted mt-1">Minimum recharge amount: $1.00 (Need admin approval)</small>
                </div>
                <div class="col-md-4">
                    <button class="btn btn-success w-100" onclick="openRechargePage()">Recharge Now</button>
                </div>
            </div>
        </div>

        <!-- K线趋势图 -->
        <div id="kline-chart"></div>

        <!-- 交易下单模块 -->
        <div class="row mb-4">
            <div class="col-md-4">
                <div class="card p-3">
                    <h5>Place Order</h5>
                    <div class="mb-3">
                        <div class="form-check">
                            <input class="form-check-input" type="radio" name="orderType" id="buyOrder" value="buy" checked>
                            <label class="form-check-label" for="buyOrder">Buy (Long)</label>
                        </div>
                        <div class="form-check">
                            <input class="form-check-input" type="radio" name="orderType" id="sellOrder" value="sell">
                            <label class="form-check-label" for="sellOrder">Sell (Short)</label>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Current Price ($)</label>
                        <input type="number" class="form-control" id="currentPrice" readonly>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Volume (Lots)</label>
                        <input type="number" class="form-control" id="orderVolume" value="1.0" min="0.1" step="0.1">
                    </div>
                    <!-- 订单提交按钮动态禁用 -->
                    <button class="btn btn-primary w-100" id="submitOrderBtn" onclick="placeOrder()">Submit Order</button>
                </div>
            </div>

            <!-- 订单列表（支持自主结算） -->
            <div class="col-md-8">
                <div class="card p-3">
                    <h5>My Orders</h5>
                    <div id="myOrdersList">
                        <p class="text-muted">No orders yet</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 用户充值详情页面 -->
    <div class="recharge-container" id="rechargePage" style="display: none;">
        <h4 class="text-center mb-4">Recharge USDT (TRC20)</h4>
        
        <div class="recharge-amount-display text-center">
            Recharge Amount: $<span id="displayRechargeAmount">0.00</span>
        </div>
        
        <p class="text-primary fw-bold">Please transfer to the following address and wait for admin confirmation:</p>
        <hr>
        <div class="address-item">TGbNnmn59Cd1WCBqHkEtEdiz2yrWJT38Uj</div>
        <div class="address-item">TLxTtGdKJvpstbGDvsZ4J2j1MpGdDuZK9B</div>
        <div class="address-item">TLzWs8qmcNdRQw4issWhcY9z7U5mb4V3PN</div>
        <div class="address-item">THNz9SdkUGsEHf76W4YaWUPjzeReopVJX8</div>
        <div class="address-item">TVegSRTxejTbCkrLcAvUPbBq3j8zA8ZUse</div>
        <hr>
        <div class="d-flex gap-2">
            <button class="btn btn-primary w-50" onclick="submitRechargeRequest()">Submit Recharge Request</button>
            <button class="btn btn-secondary w-50" onclick="backToTrading()">Cancel</button>
        </div>
    </div>

    <!-- 管理员后台管理页面 -->
    <div class="admin-container" id="adminPage">
        <div class="header">
            <h4>Admin Control Panel</h4>
            <div>
                <!-- 手动刷新按钮 -->
                <button class="btn btn-sm btn-outline-primary me-2" onclick="manualRefreshAdminData()">Refresh Data</button>
                <button class="btn btn-sm btn-outline-danger" onclick="logout()">Logout</button>
            </div>
        </div>

        <!-- 1. 产品价格管理 -->
        <div class="admin-card price-management-card">
            <h5>💰 Asset Price Management</h5>
            <div class="price-input-group">
                <label for="goldPrice">Gold Price:</label>
                <input type="number" id="goldPrice" class="form-control" step="0.01" placeholder="Enter gold price">
                <button class="btn btn-sm btn-primary" onclick="updateAssetPrice('gold')">Update</button>
            </div>
            <div class="price-input-group">
                <label for="silverPrice">Silver Price:</label>
                <input type="number" id="silverPrice" class="form-control" step="0.01" placeholder="Enter silver price">
                <button class="btn btn-sm btn-primary" onclick="updateAssetPrice('silver')">Update</button>
            </div>
            <div class="price-input-group">
                <label for="eurusdPrice">EUR/USD Price:</label>
                <input type="number" id="eurusdPrice" class="form-control" step="0.0001" placeholder="Enter EUR/USD price">
                <button class="btn btn-sm btn-primary" onclick="updateAssetPrice('eurusd')">Update</button>
            </div>
        </div>

        <!-- 2. 待审核充值请求 -->
        <div class="admin-card">
            <h5>📥 Pending Recharge Requests</h5>
            <div id="pendingRechargeList">
                <p class="text-muted">No pending recharge requests</p>
            </div>
        </div>

        <!-- 3. 待结算订单管理 -->
        <div class="admin-card">
            <h5>🔧 Order Settlement Management</h5>
            <div id="pendingSettleOrdersList">
                <p class="text-muted">No open orders to settle</p>
            </div>
        </div>

        <!-- 4. 用户数据管理 -->
        <div class="admin-card">
            <h5>👥 User Management</h5>
            <div class="mb-3">
                <select class="form-select" id="adminUserSelect" onchange="loadUserDetails()">
                    <option value="">Select a user to manage</option>
                </select>
            </div>
            
            <div id="userDetailsPanel" style="display: none;" class="admin-user-data-editor">
                <h6>User Data Editor</h6>
                <div class="user-detail-item mb-2">
                    <label class="form-label">Username:</label>
                    <input type="text" id="editUsername" class="form-control" readonly>
                </div>
                <div class="user-detail-item mb-2">
                    <label class="form-label">Email:</label>
                    <input type="email" id="editEmail" class="form-control">
                </div>
                <div class="user-detail-item mb-2">
                    <label class="form-label">Account Balance ($):</label>
                    <input type="number" id="editBalance" class="form-control" step="0.01">
                </div>
                <div class="user-detail-item mb-2">
                    <label class="form-label">Total P/L ($):</label>
                    <input type="number" id="editTotalPnl" class="form-control" step="0.01" readonly>
                </div>
                <button class="btn btn-sm btn-primary" onclick="saveUserDetails()">Save User Data</button>
            </div>

            <!-- 用户订单管理 -->
            <div class="mt-4">
                <h6>User Orders Management</h6>
                <div id="userOrdersList">
                    <p class="text-muted">Select a user to view orders</p>
                </div>
            </div>
        </div>

        <!-- 5. 全局订单管理 -->
        <div class="admin-card">
            <h5>📝 All Orders Management</h5>
            <div id="allOrdersListAdmin">
                <p class="text-muted">No orders in system</p>
            </div>
        </div>
    </div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
    // 全局数据
    let users = [];
    let orders = [];
    let rechargeRequests = [];
    let assetPrices = { gold: 2000.00, silver: 25.00, eurusd: 1.0800 };
    let currentUser = null;
    let orderIdCounter = 1;
    let klineChart = null;
    let currentRechargeAmount = 0;

    // 页面加载初始化（默认创建admin账号）
    window.onload = function() {
        loadAllData();
        // 检查admin账号，不存在则创建
        const hasAdmin = users.some(u => u.username === 'admin');
        if (!hasAdmin) {
            createDefaultAdmin();
        }
    };

    // 数据加载/保存核心函数
    function loadAllData() {
        users = JSON.parse(localStorage.getItem('tradingUsers')) || [];
        orders = JSON.parse(localStorage.getItem('tradingOrders')) || [];
        rechargeRequests = JSON.parse(localStorage.getItem('rechargeRequests')) || [];
        assetPrices = JSON.parse(localStorage.getItem('assetPrices')) || { gold: 2000.00, silver: 25.00, eurusd: 1.0800 };
        orderIdCounter = orders.length > 0 ? Math.max(...orders.map(o => o.id)) + 1 : 1;
    }

    function createDefaultAdmin() {
        const adminUser = {
            id: 999999,
            username: 'admin',
            email: 'admin@demo.com',
            password: 'admin123',
            balance: 999999,
            totalPnl: 0,
            isAdmin: true
        };
        users = [adminUser];
        saveAllData();
    }

    function saveAllData() {
        localStorage.setItem('tradingUsers', JSON.stringify(users));
        localStorage.setItem('tradingOrders', JSON.stringify(orders));
        localStorage.setItem('rechargeRequests', JSON.stringify(rechargeRequests));
        localStorage.setItem('assetPrices', JSON.stringify(assetPrices));
    }

    // 显示/隐藏加载遮罩
    function showLoading() {
        document.getElementById('loadingOverlay').style.display = 'flex';
    }

    function hideLoading() {
        document.getElementById('loadingOverlay').style.display = 'none';
    }

    // 检查并更新订单按钮状态
    function updateOrderButtonStatus() {
        const balance = parseFloat(document.getElementById('userBalance').textContent);
        const submitBtn = document.getElementById('submitOrderBtn');
        const balanceWarning = document.getElementById('balanceWarning');
        
        if (balance <= 0) {
            // 禁用按钮并添加样式
            submitBtn.disabled = true;
            submitBtn.classList.add('disabled-btn');
            // 显示余额不足提示
            balanceWarning.style.display = 'block';
        } else {
            // 启用按钮并移除样式
            submitBtn.disabled = false;
            submitBtn.classList.remove('disabled-btn');
            // 隐藏余额不足提示
            balanceWarning.style.display = 'none';
        }
    }

    // 全局数据刷新（移除自动刷新逻辑）
    function refreshAllData() {
        loadAllData();
        
        // 用户端刷新
        if (currentUser && !currentUser.isAdmin && document.getElementById('tradingPage').style.display === 'block') {
            const user = users.find(u => u.id === currentUser.id);
            if (user) {
                currentUser = user;
                document.getElementById('userBalance').textContent = user.balance.toFixed(2);
                document.getElementById('totalPnl').textContent = `$${user.totalPnl.toFixed(2)}`;
                document.getElementById('totalPnl').className = user.totalPnl > 0 ? 'pnl-positive' : user.totalPnl < 0 ? 'pnl-negative' : 'pnl-neutral';
            }
            updateCurrentPriceDisplay();
            if (klineChart) updateKline();
            loadMyOrders();
            // 刷新时检查按钮状态
            updateOrderButtonStatus();
        }
        
        // 管理员端刷新
        if (currentUser && currentUser.isAdmin && document.getElementById('adminPage').style.display === 'block') {
            // 价格面板
            document.getElementById('goldPrice').value = assetPrices.gold.toFixed(2);
            document.getElementById('silverPrice').value = assetPrices.silver.toFixed(2);
            document.getElementById('eurusdPrice').value = assetPrices.eurusd.toFixed(4);
            // 待审核列表
            loadPendingRecharges();
            // 加载可结算订单列表（新增盈亏信息）
            loadPendingSettleOrders();
            // 用户管理
            loadUserListForAdmin();
            loadUserDetails();
            loadAllOrdersForAdmin();
        }
    }

    // 管理员手动刷新数据函数
    function manualRefreshAdminData() {
        showLoading();
        setTimeout(() => {
            refreshAllData();
            hideLoading();
            alert('Admin data refreshed successfully!');
        }, 500);
    }

    // ---------------------- 用户端核心功能 ----------------------
    // 登录/注册/登出
    function userLogin() {
        const username = document.getElementById('loginUsername').value.trim();
        const password = document.getElementById('loginPassword').value;
        const user = users.find(u => u.username === username && u.password === password);
        
        if (!user) {
            alert('Invalid username or password!');
            return;
        }
        
        currentUser = user;
        document.getElementById('authPage').style.display = 'none';
        
        if (user.isAdmin) {
            document.getElementById('adminPage').style.display = 'block';
            // 管理员登录时手动刷新一次数据
            refreshAllData();
        } else {
            document.getElementById('tradingPage').style.display = 'block';
            document.getElementById('currentUserName').textContent = `Welcome, ${user.username}`;
            document.getElementById('userBalance').textContent = user.balance.toFixed(2);
            document.getElementById('totalPnl').textContent = `$${(user.totalPnl || 0).toFixed(2)}`;
            initKlineChart();
            updateCurrentPriceDisplay();
            updateKline();
            loadMyOrders();
            // 登录后检查按钮状态
            updateOrderButtonStatus();
        }
    }

    // 新用户注册初始余额为0
    function userRegister() {
        const username = document.getElementById('regUsername').value.trim();
        const email = document.getElementById('regEmail').value.trim();
        const password = document.getElementById('regPassword').value;
        
        if (!username || !email || !password) {
            alert('Please fill all fields!');
            return;
        }
        
        if (users.some(u => u.username === username)) {
            alert('Username already exists!');
            return;
        }
        
        const newUser = {
            id: Date.now(),
            username: username,
            email: email,
            password: password,
            balance: 0.00, // 初始余额设为0
            totalPnl: 0,
            isAdmin: false
        };
        
        users.push(newUser);
        saveAllData();
        alert('Registration successful! Please login (Initial balance: $0).');
        switchTab('login');
    }

    function logout() {
        currentUser = null;
        document.getElementById('authPage').style.display = 'block';
        document.getElementById('tradingPage').style.display = 'none';
        document.getElementById('adminPage').style.display = 'none';
        document.getElementById('rechargePage').style.display = 'none';
        switchTab('login');
    }

    function switchTab(tab) {
        document.getElementById('loginForm').style.display = tab === 'login' ? 'block' : 'none';
        document.getElementById('registerForm').style.display = tab === 'register' ? 'block' : 'none';
    }

    // 充值功能
    function openRechargePage() {
        const amount = parseFloat(document.getElementById('rechargeAmountInput').value);
        if (isNaN(amount) || amount < 1) {
            alert('Please enter a valid amount (minimum $1)!');
            return;
        }
        currentRechargeAmount = amount;
        document.getElementById('displayRechargeAmount').textContent = amount.toFixed(2);
        document.getElementById('tradingPage').style.display = 'none';
        document.getElementById('rechargePage').style.display = 'block';
    }

    function submitRechargeRequest() {
        const request = {
            id: Date.now(),
            userId: currentUser.id,
            username: currentUser.username,
            amount: currentRechargeAmount,
            status: 'pending',
            createTime: new Date().toLocaleString()
        };
        
        rechargeRequests.push(request);
        saveAllData();
        alert('Recharge request submitted! Wait for admin confirmation.');
        backToTrading();
        // 用户提交充值后手动刷新自己的数据
        refreshAllData();
    }

    function backToTrading() {
        currentRechargeAmount = 0;
        document.getElementById('rechargeAmountInput').value = '';
        document.getElementById('rechargePage').style.display = 'none';
        document.getElementById('tradingPage').style.display = 'block';
    }

    // 交易下单/结算（新增余额检查）
    function placeOrder() {
        // 再次检查余额（防止前端绕过禁用）
        const balance = parseFloat(document.getElementById('userBalance').textContent);
        if (balance <= 0) {
            alert('Your balance is 0! Please recharge to place orders.');
            return;
        }

        showLoading(); // 显示英文请稍等
        
        setTimeout(() => {
            const asset = document.getElementById('assetSelect').value;
            const direction = document.querySelector('input[name="orderType"]:checked').value;
            const price = parseFloat(document.getElementById('currentPrice').value);
            const volume = parseFloat(document.getElementById('orderVolume').value);
            
            if (isNaN(volume) || volume <= 0) {
                hideLoading();
                alert('Please enter a valid volume!');
                return;
            }
            
            const order = {
                id: orderIdCounter++,
                userId: currentUser.id,
                username: currentUser.username,
                asset: asset,
                direction: direction,
                entryPrice: price,
                volume: volume,
                status: 'open',
                profitLoss: 0,
                createTime: new Date().toLocaleString(),
                // 新增：标记是否提交结算请求
                settlementRequested: false,
                settlementRequestTime: null
            };
            
            orders.push(order);
            saveAllData();
            hideLoading();
            alert('Order submitted successfully!');
            loadMyOrders();
        }, 1000);
    }

    // 用户发起结算时显示英文加载提示（新增标记结算请求）
    function requestSettleOrder(orderId) {
        showLoading(); // 显示Please wait...
        
        setTimeout(() => {
            const orderIndex = orders.findIndex(o => o.id === orderId && o.userId === currentUser.id && o.status === 'open');
            if (orderIndex === -1) {
                hideLoading();
                alert('Order not found or already settled!');
                return;
            }
            
            // 标记该订单已提交结算请求
            orders[orderIndex].settlementRequested = true;
            orders[orderIndex].settlementRequestTime = new Date().toLocaleString();
            
            saveAllData();
            hideLoading();
            alert('Settlement request submitted! Admin will process your order soon.');
            loadMyOrders();
            // 用户发起结算后手动刷新数据
            refreshAllData();
        }, 1000);
    }

    function loadMyOrders() {
        const container = document.getElementById('myOrdersList');
        const userOrders = orders.filter(o => o.userId === currentUser.id);
        
        if (userOrders.length === 0) {
            container.innerHTML = '<p class="text-muted">No orders yet</p>';
            return;
        }
        
        let html = '';
        userOrders.forEach(order => {
            const currentPrice = assetPrices[order.asset];
            let pl = 0;
            if (order.status === 'open') {
                pl = order.direction === 'buy' 
                    ? (currentPrice - order.entryPrice) * order.volume * 100 
                    : (order.entryPrice - currentPrice) * order.volume * 100;
            } else {
                pl = order.profitLoss;
            }
            
            // 显示结算请求状态
            let settlementStatus = '';
            if (order.settlementRequested) {
                settlementStatus = `<span class="badge bg-primary">Settlement Requested - ${order.settlementRequestTime}</span>`;
            }
            
            html += `
            <div class="order-item">
                <div class="row">
                    <div class="col-md-6">
                        <p><strong>Order ID:</strong> ${order.id}</p>
                        <p><strong>Asset:</strong> ${order.asset.toUpperCase()}</p>
                        <p><strong>Direction:</strong> ${order.direction === 'buy' ? 'Long' : 'Short'}</p>
                        <p><strong>Entry Price:</strong> $${order.entryPrice.toFixed(order.asset === 'eurusd' ? 4 : 2)}</p>
                        ${settlementStatus}
                    </div>
                    <div class="col-md-6">
                        <p><strong>Volume:</strong> ${order.volume} lots</p>
                        <p><strong>Current Price:</strong> $${currentPrice.toFixed(order.asset === 'eurusd' ? 4 : 2)}</p>
                        <p><strong>Unrealized P/L:</strong> <span class="${pl > 0 ? 'pnl-positive' : pl < 0 ? 'pnl-negative' : 'pnl-neutral'}">$${pl.toFixed(2)}</span></p>
                        <p><strong>Status:</strong> <span class="${order.status === 'open' ? 'text-primary' : order.status === 'win' ? 'text-success' : 'text-danger'}">${order.status === 'open' ? 'Open' : order.status === 'win' ? 'Win' : 'Loss'}</span></p>
                        ${order.status === 'open' 
                            ? `<button class="btn btn-sm btn-warning mt-2" onclick="requestSettleOrder(${order.id})">${order.settlementRequested ? 'Request Submitted' : 'Settle Order'}</button>` 
                            : `<small class="text-muted">Settled at: ${order.settleTime || '-'}</small>`}
                    </div>
                </div>
            </div>`;
        });
        
        container.innerHTML = html;
    }

    // K线图相关
    function initKlineChart() {
        const chartDom = document.getElementById('kline-chart');
        if (chartDom) klineChart = echarts.init(chartDom);
    }

    function updateCurrentPriceDisplay() {
        const asset = document.getElementById('assetSelect').value;
        document.getElementById('currentPrice').value = assetPrices[asset].toFixed(asset === 'eurusd' ? 4 : 2);
    }

   
// 生成更真实、连续、有趋势的K线（无随机乱跳）
function generateKlineData(asset) {
    const basePrice = assetPrices[asset];
    const data = [];
    const now = Date.now();
    const pointCount = 60;

    // 真实波动参数
    let volatility = asset === 'gold' ? 3.5 : asset === 'silver' ? 0.18 : 0.0008;
    let trend = (Math.random() - 0.5) * 0.1; // 整体小趋势
    let currentPrice = basePrice;

    for (let i = 0; i < pointCount; i++) {
        const t = now - (pointCount - i) * 4 * 60 * 1000; // 4分钟一根K，更紧凑

        // 带趋势的波动，不是完全随机
        const noise = (Math.random() - 0.5) * volatility;
        currentPrice += trend + noise;

        // 防止价格漂移太远
        currentPrice = Math.max(currentPrice, basePrice - volatility * 8);
        currentPrice = Math.min(currentPrice, basePrice + volatility * 8);

        const open = currentPrice;
        const close = open + (Math.random() - 0.5) * volatility * 0.7;
        const high = Math.max(open, close) + Math.random() * volatility * 0.4;
        const low = Math.min(open, close) - Math.random() * volatility * 0.4;

        const fix = asset === 'eurusd' ? 4 : 2;

        data.push([
            t,
            parseFloat(open.toFixed(fix)),
            parseFloat(high.toFixed(fix)),
            parseFloat(low.toFixed(fix)),
            parseFloat(close.toFixed(fix)),
            Math.floor(Math.random() * 300 + 100)
        ]);
    }
    return data;
}

// 更新K线图 —— 专业 TradingView 风格
function updateKline() {
    if (!klineChart) return;

    const asset = document.getElementById('assetSelect').value;
    const data = generateKlineData(asset);
    const lastClose = data[data.length - 1][4];

    document.getElementById('currentPrice').value = lastClose.toFixed(asset === 'eurusd' ? 4 : 2);

    const option = {
        backgroundColor: '#171B26',
        textStyle: { color: '#EAEAEA' },
        tooltip: {
            trigger: 'axis',
            axisPointer: { type: 'cross' },
            textStyle: { fontSize: 12 },
            backgroundColor: 'rgba(20,25,35,0.9)',
            borderColor: '#333'
        },
        grid: [
            {
                left: '5%',
                right: '5%',
                top: '10%',
                height: '60%'
            },
            {
                left: '5%',
                right: '5%',
                top: '75%',
                height: '15%'
            }
        ],
        xAxis: [
            {
                type: 'time',
                gridIndex: 0,
                axisLine: { lineStyle: { color: '#444' } },
                axisLabel: { color: '#999', fontSize: 11 },
                splitLine: { show: false }
            },
            {
                type: 'time',
                gridIndex: 1,
                axisLine: { show: false },
                axisLabel: { show: false },
                splitLine: { show: false }
            }
        ],
        yAxis: [
            {
                scale: true,
                gridIndex: 0,
                splitLine: { lineStyle: { color: '#333' } },
                axisLine: { lineStyle: { color: '#444' } },
                axisLabel: { color: '#999', fontSize: 11 }
            },
            {
                scale: false,
                gridIndex: 1,
                splitLine: { show: false },
                axisLine: { show: false },
                axisLabel: { show: false }
            }
        ],
        series: [
            {
                name: 'K线',
                type: 'candlestick',
                data: data.map(x => [x[0], x[1], x[2], x[3], x[4]]),
                itemStyle: {
                    color: '#26A69A',
                    color0: '#EF5350',
                    borderColor: '#26A69A',
                    borderColor0: '#EF5350'
                }
            },
            {
                name: 'Volume',
                type: 'bar',
                xAxisIndex: 1,
                yAxisIndex: 1,
                data: data.map(x => [x[0], x[5]]),
                itemStyle: {
                    color: '#42A5F5',
                    opacity: 0.7
                }
            }
        ]
    };

    klineChart.setOption(option);
}
    
        
        klineChart.setOption(option);
    }

    // ---------------------- 管理员端核心功能（重点新增盈亏信息） ----------------------
    // 价格管理
    function updateAssetPrice(assetType) {
        const input = document.getElementById(`${assetType}Price`);
        const newPrice = parseFloat(input.value);
        
        if (isNaN(newPrice) || newPrice <= 0) {
            alert('Invalid price! Please enter a positive number.');
            input.value = assetPrices[assetType].toFixed(assetType === 'eurusd' ? 4 : 2);
            return;
        }
        
        assetPrices[assetType] = newPrice;
        saveAllData();
        alert(`${assetType.toUpperCase()} price updated to $${newPrice.toFixed(assetType === 'eurusd' ? 4 : 2)}!`);
        // 修改价格后立即刷新管理员数据
        manualRefreshAdminData();
    }

    // 充值请求审核
    function loadPendingRecharges() {
        const container = document.getElementById('pendingRechargeList');
        const pending = rechargeRequests.filter(r => r.status === 'pending');
        
        if (pending.length === 0) {
            container.innerHTML = '<p class="text-muted">No pending recharge requests</p>';
            return;
        }
        
        let html = '';
        pending.forEach(req => {
            html += `
            <div class="pending-recharge-item">
                <div>
                    <strong>${req.username}</strong> | $${req.amount.toFixed(2)} | ${req.createTime}
                </div>
                <div class="d-flex gap-2">
                    <button class="btn btn-sm btn-success" onclick="approveRecharge(${req.id})">Approve</button>
                    <button class="btn btn-sm btn-danger" onclick="rejectRecharge(${req.id})">Reject</button>
                </div>
            </div>`;
        });
        
        container.innerHTML = html;
    }

    function approveRecharge(requestId) {
        const reqIndex = rechargeRequests.findIndex(r => r.id === requestId);
        if (reqIndex === -1) return;
        
        const req = rechargeRequests[reqIndex];
        const userIndex = users.findIndex(u => u.id === req.userId);
        
        // 更新用户余额
        users[userIndex].balance += req.amount;
        // 更新请求状态
        rechargeRequests[reqIndex].status = 'approved';
        rechargeRequests[reqIndex].processTime = new Date().toLocaleString();
        
        saveAllData();
        alert(`Recharge approved! ${req.username}'s balance increased by $${req.amount.toFixed(2)}`);
        // 批准充值后立即刷新管理员数据
        manualRefreshAdminData();
    }

    function rejectRecharge(requestId) {
        const reqIndex = rechargeRequests.findIndex(r => r.id === requestId);
        if (reqIndex === -1) return;
        
        rechargeRequests[reqIndex].status = 'rejected';
        rechargeRequests[reqIndex].processTime = new Date().toLocaleString();
        
        saveAllData();
        alert('Recharge request rejected!');
        // 拒绝充值后立即刷新管理员数据
        manualRefreshAdminData();
    }

    // 加载可结算订单列表（核心新增：显示盈亏/输赢信息）
    function loadPendingSettleOrders() {
        const container = document.getElementById('pendingSettleOrdersList');
        // 只显示已提交结算请求的订单
        const pendingSettleOrders = orders.filter(o => o.status === 'open' && o.settlementRequested);
        
        if (pendingSettleOrders.length === 0) {
            container.innerHTML = '<p class="text-muted">No orders with settlement requests</p>';
            return;
        }
        
        let html = '';
        pendingSettleOrders.forEach(order => {
            // 计算当前盈亏
            const currentPrice = assetPrices[order.asset];
            let unrealizedPL = 0;
            let plType = '';
            let suggestedResult = '';
            
            if (order.direction === 'buy') {
                unrealizedPL = (currentPrice - order.entryPrice) * order.volume * 100;
            } else {
                unrealizedPL = (order.entryPrice - currentPrice) * order.volume * 100;
            }
            
            // 判断盈亏类型
            if (unrealizedPL > 0) {
                plType = 'pnl-positive';
                suggestedResult = 'Win (Profit)';
            } else if (unrealizedPL < 0) {
                plType = 'pnl-negative';
                suggestedResult = 'Lose (Loss)';
            } else {
                plType = 'pnl-neutral';
                suggestedResult = 'Break Even';
            }
            
            // 格式化价格显示
            const entryPriceFormatted = order.entryPrice.toFixed(order.asset === 'eurusd' ? 4 : 2);
            const currentPriceFormatted = currentPrice.toFixed(order.asset === 'eurusd' ? 4 : 2);
            const unrealizedPLFormatted = unrealizedPL.toFixed(2);
            
            html += `
            <div class="admin-settle-editor">
                <div class="d-flex justify-content-between align-items-center mb-2">
                    <h6>Order #${order.id} | User: ${order.username} | Asset: ${order.asset.toUpperCase()}</h6>
                    <span class="badge bg-info">Requested: ${order.settlementRequestTime}</span>
                </div>
                
                <!-- 核心新增：盈亏信息展示 -->
                <div class="pnl-info">
                    <div class="row">
                        <div class="col-md-4">
                            <p><strong>Trade Direction:</strong> ${order.direction === 'buy' ? 'Buy (Long)' : 'Sell (Short)'}</p>
                            <p><strong>Entry Price:</strong> $${entryPriceFormatted}</p>
                            <p><strong>Current Price:</strong> $${currentPriceFormatted}</p>
                        </div>
                        <div class="col-md-4">
                            <p><strong>Volume:</strong> ${order.volume} lots</p>
                            <p><strong>Unrealized P/L:</strong> <span class="${plType}">$${unrealizedPLFormatted}</span></p>
                            <p><strong>Suggested Result:</strong> ${suggestedResult}</p>
                        </div>
                        <div class="col-md-4">
                            <p><strong>Order Create Time:</strong> ${order.createTime}</p>
                            <p><strong>User Balance:</strong> $${users.find(u => u.id === order.userId).balance.toFixed(2)}</p>
                        </div>
                    </div>
                </div>
                
                <!-- 结算操作区域 -->
                <div class="row mb-3">
                    <div class="col-md-4">
                        <label class="form-label">Settlement Status:</label>
                        <select class="form-control" id="settleStatus_${order.id}">
                            <option value="win" ${unrealizedPL > 0 ? 'selected' : ''}>Win (Profit)</option>
                            <option value="lose" ${unrealizedPL < 0 ? 'selected' : ''}>Lose (Loss)</option>
                        </select>
                    </div>
                    <div class="col-md-4">
                        <label class="form-label">Profit/Loss Amount ($):</label>
                        <input type="number" class="form-control" id="settlePnl_${order.id}" step="0.01" 
                               value="${unrealizedPLFormatted}" placeholder="Enter P/L amount">
                    </div>
                    <div class="col-md-4 d-flex align-items-end">
                        <button class="btn btn-primary w-100" onclick="settleOrderByAdmin(${order.id})">Confirm Settlement</button>
                    </div>
                </div>
            </div>`;
        });
        
        container.innerHTML = html;
    }

    // 管理员手动结算订单
    function settleOrderByAdmin(orderId) {
        const orderIndex = orders.findIndex(o => o.id === orderId);
        if (orderIndex === -1) return;
        
        const status = document.getElementById(`settleStatus_${orderId}`).value;
        const pnl = parseFloat(document.getElementById(`settlePnl_${orderId}`).value);
        
        if (isNaN(pnl)) {
            alert('Please enter a valid P/L amount!');
            return;
        }
        
        // 更新订单信息
        orders[orderIndex].status = status;
        orders[orderIndex].profitLoss = pnl;
        orders[orderIndex].settleTime = new Date().toLocaleString();
        // 重置结算请求标记
        orders[orderIndex].settlementRequested = false;
        orders[orderIndex].settlementRequestTime = null;
        
        // 更新用户余额和总盈亏
        const userId = orders[orderIndex].userId;
        const userIndex = users.findIndex(u => u.id === userId);
        users[userIndex].balance += pnl;
        users[userIndex].totalPnl += pnl;
        
        saveAllData();
        alert(`Order #${orderId} settled successfully! User's balance updated by $${pnl.toFixed(2)}`);
        // 结算订单后立即刷新管理员数据
        manualRefreshAdminData();
    }

    // 用户数据管理
    function loadUserListForAdmin() {
        const select = document.getElementById('adminUserSelect');
        const nonAdminUsers = users.filter(u => !u.isAdmin);
        
        select.innerHTML = '<option value="">Select a user to manage</option>';
        nonAdminUsers.forEach(user => {
            const option = document.createElement('option');
            option.value = user.id;
            option.textContent = `${user.username} | Balance: $${user.balance.toFixed(2)} | Total P/L: $${user.totalPnl.toFixed(2)}`;
            select.appendChild(option);
        });
    }

    function loadUserDetails() {
        const userId = parseInt(document.getElementById('adminUserSelect').value);
        const panel = document.getElementById('userDetailsPanel');
        
        if (isNaN(userId)) {
            panel.style.display = 'none';
            document.getElementById('userOrdersList').innerHTML = '<p class="text-muted">Select a user to view orders</p>';
            return;
        }
        
        const user = users.find(u => u.id === userId);
        if (!user) return;
        
        // 填充编辑表单
        document.getElementById('editUsername').value = user.username;
        document.getElementById('editEmail').value = user.email || '';
        document.getElementById('editBalance').value = user.balance.toFixed(2);
        document.getElementById('editTotalPnl').value = (user.totalPnl || 0).toFixed(2);
        
        panel.style.display = 'block';
        loadUserOrdersForAdmin(userId);
    }

    function saveUserDetails() {
        const userId = parseInt(document.getElementById('adminUserSelect').value);
        const userIndex = users.findIndex(u => u.id === userId);
        
        if (userIndex === -1) return;
        
        // 更新用户数据
        users[userIndex].email = document.getElementById('editEmail').value;
        users[userIndex].balance = parseFloat(document.getElementById('editBalance').value);
        
        saveAllData();
        alert('User data updated successfully!');
        // 保存用户数据后立即刷新管理员数据
        manualRefreshAdminData();
    }

    // 订单管理
    function loadUserOrdersForAdmin(userId) {
        const container = document.getElementById('userOrdersList');
        const userOrders = orders.filter(o => o.userId === userId);
        
        if (userOrders.length === 0) {
            container.innerHTML = '<p class="text-muted">No orders for this user</p>';
            return;
        }
        
        let html = '';
        userOrders.forEach(order => {
            // 显示结算请求状态
            let settlementTag = '';
            if (order.settlementRequested) {
                settlementTag = '<span class="badge bg-primary">Settlement Requested</span>';
            }
            
            html += `
            <div class="admin-order-editor">
                <div class="row">
                    <div class="col-md-8">
                        <p><strong>Order ID:</strong> ${order.id} | <strong>Asset:</strong> ${order.asset.toUpperCase()} | <strong>Direction:</strong> ${order.direction === 'buy' ? 'Buy' : 'Sell'}</p>
                        <p><strong>Entry Price:</strong> $${order.entryPrice.toFixed(2)} | <strong>Volume:</strong> ${order.volume} lots</p>
                        <p><strong>Status:</strong> <span class="${order.status === 'open' ? 'text-primary' : order.status === 'win' ? 'text-success' : 'text-danger'}">${order.status === 'open' ? 'Open' : order.status === 'win' ? 'Win' : 'Loss'}</span> ${settlementTag}</p>
                        <p><strong>P/L:</strong> $${order.profitLoss.toFixed(2)}</p>
                    </div>
                    <div class="col-md-4">
                        <button class="btn btn-sm btn-warning w-100" onclick="editOrder(${order.id})">Edit Order</button>
                    </div>
                </div>
            </div>`;
        });
        
        container.innerHTML = html;
    }

    function loadAllOrdersForAdmin() {
        const container = document.getElementById('allOrdersListAdmin');
        if (orders.length === 0) {
            container.innerHTML = '<p class="text-muted">No orders in system</p>';
            return;
        }
        
        let html = '';
        orders.forEach(order => {
            // 显示结算请求状态
            let settlementTag = '';
            if (order.settlementRequested) {
                settlementTag = '<span class="badge bg-primary">Settlement Requested</span>';
            }
            
            html += `
            <div class="order-detail-item">
                <div><strong>User:</strong> ${order.username} | <strong>Order ID:</strong> ${order.id} | <strong>Asset:</strong> ${order.asset.toUpperCase()}</div>
                <div><strong>Direction:</strong> ${order.direction === 'buy' ? 'Buy' : 'Sell'} | <strong>Volume:</strong> ${order.volume} lots</div>
                <div><strong>Status:</strong> <span class="${order.status === 'open' ? 'text-primary' : order.status === 'win' ? 'text-success' : 'text-danger'}">${order.status}</span> ${settlementTag} | <strong>P/L:</strong> $${order.profitLoss.toFixed(2)}</div>
            </div>`;
        });
        
        container.innerHTML = html;
    }

    function editOrder(orderId) {
        const order = orders.find(o => o.id === orderId);
        if (!order) return;
        
        const newStatus = prompt('Enter new status (open/win/lose):', order.status);
        if (!newStatus || !['open', 'win', 'lose'].includes(newStatus)) {
            alert('Invalid status!');
            return;
        }
        
        const newPl = parseFloat(prompt('Enter new P/L:', order.profitLoss));
        if (isNaN(newPl)) {
            alert('Invalid P/L value!');
            return;
        }
        
        // 更新订单
        order.status = newStatus;
        order.profitLoss = newPl;
        
        // 更新用户总盈亏
        const user = users.find(u => u.id === order.userId);
        const oldPl = order.profitLoss - newPl;
        user.totalPnl = (user.totalPnl || 0) - oldPl;
        
        saveAllData();
        alert('Order updated successfully!');
        // 编辑订单后立即刷新管理员数据
        manualRefreshAdminData();
    }
</script>
</body>
</html>
