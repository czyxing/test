<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>24点游戏求解器</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            max-width: 800px;
            width: 100%;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            backdrop-filter: blur(10px);
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 10px;
            font-size: 32px;
            font-weight: 700;
            background: linear-gradient(135deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .subtitle {
            text-align: center;
            color: #666;
            margin-bottom: 25px;
            font-size: 14px;
        }

        .input-section {
            display: flex;
            gap: 15px;
            justify-content: center;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 5px;
        }

        .input-group label {
            font-size: 14px;
            color: #555;
            font-weight: 600;
        }

        .input-group input {
            width: 70px;
            height: 50px;
            text-align: center;
            font-size: 24px;
            font-weight: 700;
            border: 2px solid #ddd;
            border-radius: 12px;
            transition: all 0.3s;
            background: #f8f9fa;
            color: #333;
        }

        .input-group input:focus {
            border-color: #667eea;
            outline: none;
            background: #fff;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.2);
        }

        .input-group input::-webkit-inner-spin-button {
            -webkit-appearance: none;
        }

        .button-group {
            display: flex;
            gap: 10px;
            justify-content: center;
            flex-wrap: wrap;
            margin-bottom: 20px;
        }

        .btn {
            padding: 12px 30px;
            border: none;
            border-radius: 12px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            color: white;
        }

        .btn-primary {
            background: linear-gradient(135deg, #667eea, #764ba2);
        }

        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4);
        }

        .btn-primary:active {
            transform: translateY(0);
        }

        .btn-secondary {
            background: #6c757d;
        }

        .btn-secondary:hover {
            background: #5a6268;
            transform: translateY(-2px);
        }

        .btn-success {
            background: #28a745;
        }

        .btn-success:hover {
            background: #218838;
            transform: translateY(-2px);
        }

        .btn-info {
            background: #17a2b8;
        }

        .btn-info:hover {
            background: #138496;
            transform: translateY(-2px);
        }

        .result-section {
            background: #f8f9fa;
            border-radius: 12px;
            padding: 20px;
            min-height: 100px;
            max-height: 500px;
            overflow-y: auto;
            margin-top: 15px;
            border: 2px solid #e9ecef;
        }

        .result-section h3 {
            color: #333;
            margin-bottom: 15px;
            font-size: 18px;
        }

        .result-item {
            background: white;
            padding: 10px 15px;
            border-radius: 8px;
            margin-bottom: 8px;
            border-left: 4px solid #667eea;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
            font-family: 'Courier New', monospace;
            font-size: 15px;
            color: #333;
            word-break: break-all;
        }

        .result-item:last-child {
            margin-bottom: 0;
        }

        .result-stats {
            background: #e9ecef;
            padding: 8px 15px;
            border-radius: 8px;
            margin-top: 10px;
            font-weight: 600;
            color: #495057;
            text-align: center;
        }

        .result-empty {
            text-align: center;
            color: #6c757d;
            padding: 30px 0;
            font-size: 16px;
        }

        .result-empty .icon {
            font-size: 48px;
            margin-bottom: 10px;
            display: block;
        }

        .warning {
            color: #dc3545;
            font-weight: 600;
            background: #f8d7da;
            padding: 10px 15px;
            border-radius: 8px;
            margin-bottom: 15px;
            border: 1px solid #f5c6cb;
        }

        .known-solution {
            background: #d1ecf1;
            border-left-color: #17a2b8;
        }

        .mode-tag {
            display: inline-block;
            padding: 3px 10px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 600;
            margin-left: 10px;
        }

        .mode-basic {
            background: #28a745;
            color: white;
        }

        .mode-extended {
            background: #ffc107;
            color: #333;
        }

        .mode-known {
            background: #17a2b8;
            color: white;
        }

        .mode-none {
            background: #dc3545;
            color: white;
        }

        .loading {
            text-align: center;
            padding: 20px;
            color: #6c757d;
        }

        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 10px auto;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        @media (max-width: 600px) {
            .container {
                padding: 15px;
            }
            
            .input-section {
                gap: 8px;
            }
            
            .input-group input {
                width: 55px;
                height: 45px;
                font-size: 20px;
            }
            
            .btn {
                padding: 10px 20px;
                font-size: 14px;
            }
            
            .result-item {
                font-size: 13px;
                padding: 8px 12px;
            }
        }

        /* 滚动条样式 */
        .result-section::-webkit-scrollbar {
            width: 6px;
        }

        .result-section::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 3px;
        }

        .result-section::-webkit-scrollbar-thumb {
            background: #667eea;
            border-radius: 3px;
        }

        .result-section::-webkit-scrollbar-thumb:hover {
            background: #764ba2;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🃏 24点求解器</h1>
        <div class="subtitle">输入4个数字 (1-13)，尝试用加减乘除、根号、阶乘得到24</div>

        <div class="input-section">
            <div class="input-group">
                <label>数字 1</label>
                <input type="number" id="num1" value="1" min="1" max="13">
            </div>
            <div class="input-group">
                <label>数字 2</label>
                <input type="number" id="num2" value="2" min="1" max="13">
            </div>
            <div class="input-group">
                <label>数字 3</label>
                <input type="number" id="num3" value="3" min="1" max="13">
            </div>
            <div class="input-group">
                <label>数字 4</label>
                <input type="number" id="num4" value="4" min="1" max="13">
            </div>
        </div>

        <div class="button-group">
            <button class="btn btn-primary" onclick="solve()">🎯 求解</button>
            <button class="btn btn-secondary" onclick="clearResult()">🗑️ 清空</button>
            <button class="btn btn-info" onclick="loadExample()">📝 示例</button>
        </div>

        <div id="resultArea" class="result-section">
            <h3>📊 求解结果</h3>
            <div class="result-empty">
                <span class="icon">🎲</span>
                输入4个数字，点击"求解"开始<br>
                <small style="color: #999;">支持数字范围: 1-13</small>
            </div>
        </div>
    </div>

    <script>
        // ============ 核心算法 ============
        const fuhao = ['+', '-', '*', '/'];

        function j(a, b, c) {
            if (c === 0) return a + b;
            if (c === 1) return a - b;
            if (c === 2) return a * b;
            if (c === 3) {
                if (b !== 0) return a / b;
                throw new Error('1/0');
            }
            return 0;
        }

        function gcd(a, b) {
            a = Math.abs(a);
            b = Math.abs(b);
            while (b) {
                [a, b] = [b, a % b];
            }
            return a;
        }

        function formatFraction(val) {
            if (Number.isInteger(val)) return val.toString();
            // 尝试用分数表示
            const precision = 1e-10;
            let bestNum = 1;
            let bestDen = 1;
            let bestError = Infinity;
            for (let den = 1; den <= 1000; den++) {
                const num = Math.round(val * den);
                const error = Math.abs(val - num / den);
                if (error < bestError) {
                    bestError = error;
                    bestNum = num;
                    bestDen = den;
                    if (error < precision) break;
                }
            }
            const g = gcd(bestNum, bestDen);
            bestNum /= g;
            bestDen /= g;
            if (bestDen === 1) return bestNum.toString();
            return `${bestNum}/${bestDen}`;
        }

        function p(a, a2, b2, c2, d2, e, f, g, h, targetVal) {
            // 构建表达式字符串
            let word = '';
            if (e === 0) {
                word = `(${formatExpr(a2, b2, f)})${fuhao[h]}(${formatExpr(c2, d2, g)})`;
            } else if (e === 1) {
                word = `((${formatExpr(a2, b2, f)})${fuhao[g]}${c2})${fuhao[h]}${d2}`;
            } else if (e === 2) {
                word = `(${a2}${fuhao[g]}(${formatExpr(b2, c2, f)}))${fuhao[h]}${d2}`;
            } else if (e === 3) {
                word = `${a2}${fuhao[h]}((${formatExpr(b2, c2, f)})${fuhao[g]}${d2})`;
            } else if (e === 4) {
                word = `${a2}${fuhao[h]}(${b2}${fuhao[g]}(${formatExpr(c2, d2, f)}))`;
            }

            // 应用根号和阶乘
            if (targetVal === 4) {
                word = `(${word})!`;
            } else if (targetVal === 16) {
                word = `(sqrt(${word}))!`;
            } else if (targetVal === 576) {
                word = `sqrt(${word})`;
            } else if (targetVal === 24) {
                // 不加修饰
            } else {
                throw new Error('未知的target');
            }
            return word;
        }

        function formatExpr(a, b, op) {
            return `${a}${fuhao[op]}${b}`;
        }

        function solve24(a, b, c, d, targets, callback) {
            const shunxu = [
                [a, b, c, d], [a, b, d, c], [a, c, b, d], [a, c, d, b], [a, d, b, c], [a, d, c, b],
                [b, a, c, d], [b, a, d, c], [b, c, a, d], [b, c, d, a], [b, d, a, c], [b, d, c, a],
                [c, b, a, d], [c, b, d, a], [c, a, b, d], [c, a, d, b], [c, d, b, a], [c, d, a, b],
                [d, b, c, a], [d, b, a, c], [d, c, b, a], [d, c, a, b], [d, a, b, c], [d, a, c, b]
            ];

            const results = new Set();
            const targetSet = new Set(targets);

            for (const [a2, b2, c2, d2] of shunxu) {
                for (let e = 0; e < 5; e++) {
                    for (let f = 0; f < 4; f++) {
                        for (let g = 0; g < 4; g++) {
                            for (let h = 0; h < 4; h++) {
                                try {
                                    let val;
                                    if (e === 0) {
                                        val = j(j(a2, b2, f), j(c2, d2, g), h);
                                    } else if (e === 1) {
                                        val = j(j(j(a2, b2, f), c2, g), d2, h);
                                    } else if (e === 2) {
                                        val = j(j(a2, j(b2, c2, f), g), d2, h);
                                    } else if (e === 3) {
                                        val = j(a2, j(j(b2, c2, f), d2, g), h);
                                    } else if (e === 4) {
                                        val = j(a2, j(b2, j(c2, d2, f), g), h);
                                    }

                                    // 检查是否匹配目标值
                                    for (const target of targetSet) {
                                        if (Math.abs(val - target) < 1e-10) {
                                            const word = p(val, a2, b2, c2, d2, e, f, g, h, target);
                                            results.add(word);
                                        }
                                    }
                                } catch (ex) {
                                    // 忽略除零错误
                                    if (ex.message !== '1/0') {
                                        // console.error(ex);
                                    }
                                }
                            }
                        }
                    }
                }
            }

            callback(results);
        }

        // ============ 已知结果库 ============
        const knownSolutions = {
            '1 1 10 10': '(10-(sqrt(10-1*1))!)!',
            '1 7 7 7': '',
            '1 7 7 13': '',
            '1 7 10 10': '',
            '1 10 10 10': '(sqrt(10*10)-(sqrt(10-1))!)!',
            '1 10 10 11': '',
            '1 10 10 13': '',
            '1 10 11 11': '(11/11+sqrt(10-1))!',
            '1 11 11 11': '',
            '1 13 13 13': '',
            '3 3 10 10': '(sqrt((10-3!)(10-3!)))!',
            '3 3 13 13': '(sqrt(sqrt((3+13)(3+13))))!',
            '3 7 10 12': '12/sqrt(3*(10-7))',
            '3 8 10 13': '8*(3!-13+10)',
            '3 10 12 12': '12-3!*(10-12)',
            '3 10 12 13': '(12*3!)/(13-10)',
            '5 5 5 7': '(5!/(5*7-5))!',
            '5 5 5 13': '(sqrt(5-13+5!/5))!',
            '7 10 10 13': '(sqrt(10*10)+7-13)!',
            '9 9 10 10': '(sqrt((10-(sqrt(9))!)*(10-(sqrt(9))!)))!',
            '10 10 13 13': '(10-(sqrt(13*13)-10)!)!'
        };

        // ============ UI 逻辑 ============
        function getInputs() {
            const n1 = parseInt(document.getElementById('num1').value) || 0;
            const n2 = parseInt(document.getElementById('num2').value) || 0;
            const n3 = parseInt(document.getElementById('num3').value) || 0;
            const n4 = parseInt(document.getElementById('num4').value) || 0;
            return [n1, n2, n3, n4];
        }

        function showResult(html) {
            document.getElementById('resultArea').innerHTML = html;
        }

        function clearResult() {
            showResult(`
                <h3>📊 求解结果</h3>
                <div class="result-empty">
                    <span class="icon">🎲</span>
                    输入4个数字，点击"求解"开始<br>
                    <small style="color: #999;">支持数字范围: 1-13</small>
                </div>
            `);
        }

        function loadExample() {
            document.getElementById('num1').value = 1;
            document.getElementById('num2').value = 1;
            document.getElementById('num3').value = 10;
            document.getElementById('num4').value = 10;
            solve();
        }

        function solve() {
            const nums = getInputs();
            const [a, b, c, d] = nums;
            
            // 检查输入
            const invalid = nums.some(n => isNaN(n) || n < 1);
            if (invalid) {
                showResult(`
                    <h3>📊 求解结果</h3>
                    <div class="warning">⚠️ 请输入有效的正整数 (≥1)</div>
                `);
                return;
            }

            const warning = nums.some(n => n > 13);

            // 排序
            const sorted = [...nums].sort((x, y) => x - y);
            const key = sorted.join(' ');

            // 显示加载状态
            showResult(`
                <h3>📊 求解结果</h3>
                <div class="loading">
                    <div class="spinner"></div>
                    <p>正在计算...</p>
                </div>
            `);

            // 使用 setTimeout 避免阻塞 UI
            setTimeout(() => {
                // 第一步：仅用加减乘除
                const basicTargets = [24];
                solve24(a, b, c, d, basicTargets, (basicResults) => {
                    const basicList = Array.from(basicResults);

                    if (basicList.length > 0) {
                        // 有基本解
                        let html = `<h3>📊 求解结果 <span class="mode-tag mode-basic">加减乘除</span></h3>`;
                        html += `<div class="result-stats">找到 ${basicList.length} 个解</div>`;
                        for (const expr of basicList) {
                            html += `<div class="result-item">${expr} = 24</div>`;
                        }
                        showResult(html);
                        return;
                    }

                    // 第二步：尝试根号和阶乘
                    const extendedTargets = [4, 16, 24, 576];
                    solve24(a, b, c, d, extendedTargets, (extendedResults) => {
                        const extendedList = Array.from(extendedResults);

                        if (extendedList.length > 0) {
                            let html = `<h3>📊 求解结果 <span class="mode-tag mode-extended">根号+阶乘</span></h3>`;
                            html += `<div class="result-stats">找到 ${extendedList.length} 个解</div>`;
                            for (const expr of extendedList) {
                                html += `<div class="result-item">${expr} = 24</div>`;
                            }
                            showResult(html);
                            return;
                        }

                        // 第三步：检查已知结果库
                        if (key in knownSolutions && knownSolutions[key]) {
                            let html = `<h3>📊 求解结果 <span class="mode-tag mode-known">已知解</span></h3>`;
                            html += `<div class="result-stats">使用预存已知解</div>`;
                            html += `<div class="result-item known-solution">${knownSolutions[key]} = 24</div>`;
                            if (warning) {
                                html += `<div class="warning">⚠️ 警告: 输入超出标准范围 (1-13)，结果可能不准确</div>`;
                            }
                            showResult(html);
                            return;
                        }

                        // 无解
                        let html = `<h3>📊 求解结果 <span class="mode-tag mode-none">无解</span></h3>`;
                        html += `<div class="result-empty">
                            <span class="icon">😔</span>
                            <p>仅用加减乘除、根号、阶乘均无解</p>
                        </div>`;
                        if (key in knownSolutions && knownSolutions[key] === '') {
                            html += `<div class="result-item known-solution">⚠️ 该组合已知无解</div>`;
                        } else if (!warning) {
                            html += `<div class="result-item" style="border-left-color: #ffc107;">
                                💡 如果你发现了解法，欢迎告诉我！
                            </div>`;
                        }
                        if (warning) {
                            html += `<div class="warning">⚠️ 警告: 输入超出标准范围 (1-13)，结果可能不准确</div>`;
                        }
                        showResult(html);
                    });
                });
            }, 100);
        }

        // ============ 键盘支持 ============
        document.addEventListener('keydown', function(e) {
            if (e.key === 'Enter') {
                const active = document.activeElement;
                if (active && active.tagName === 'INPUT' && active.type === 'number') {
                    solve();
                }
            }
        });

        // 页面加载完成后自动求解一次（显示示例）
        window.onload = function() {
            // 显示默认示例
            setTimeout(solve, 300);
        };
    </script>
</body>
</html>
