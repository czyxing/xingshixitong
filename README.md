<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>形式系统H · 编译器（增强版）</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Consolas', 'Monaco', 'Fira Code', monospace;
        }
        body {
            background: #0e1a1f;
            color: #d4e9e2;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            max-width: 1100px;
            width: 100%;
            background: #1a2a30;
            padding: 24px 28px;
            border-radius: 28px;
            box-shadow: 0 12px 28px rgba(0,0,0,0.7), inset 0 1px 3px rgba(255,255,255,0.05);
            border: 1px solid #3b5c5a;
        }
        h1 {
            font-size: 1.8rem;
            font-weight: 400;
            text-align: center;
            margin-top: 0;
            margin-bottom: 10px;
            color: #a9e6d4;
            letter-spacing: 1px;
            border-bottom: 1px dashed #3a6b63;
            padding-bottom: 16px;
            font-family: 'Segoe UI', system-ui, sans-serif;
        }
        .subtitle {
            text-align: center;
            color: #7fa8a0;
            font-size: 0.9rem;
            margin-bottom: 26px;
            font-family: 'Segoe UI', sans-serif;
        }
        .rules {
            background: #0f1e22;
            border-left: 6px solid #35b8a0;
            padding: 16px 20px;
            border-radius: 16px;
            margin-bottom: 28px;
            font-size: 0.9rem;
            line-height: 1.6;
            color: #bdd9d1;
            box-shadow: inset 0 2px 6px rgba(0,0,0,0.5);
        }
        .rules p {
            margin: 6px 0;
        }
        .rules span {
            color: #f5d88b;
            font-weight: 500;
        }

        /* 编辑器区域 */
        .editor-wrapper {
            position: relative;
            display: flex;
            border-radius: 20px;
            background: #0b181d;
            border: 1px solid #2f5e56;
            box-shadow: inset 0 4px 12px #00000055;
            transition: 0.2s;
            margin-bottom: 18px;
            min-height: 240px;
        }
        .editor-wrapper:focus-within {
            border-color: #52c9b2;
            box-shadow: 0 0 0 3px #2b6b5e55, inset 0 4px 12px #000;
        }
        .line-numbers {
            padding: 14px 10px 14px 16px;
            text-align: right;
            color: #3f6b62;
            font-size: 15px;
            line-height: 1.7;
            user-select: none;
            border-right: 1px solid #1e3f3a;
            background: #0d1c21;
            border-top-left-radius: 20px;
            border-bottom-left-radius: 20px;
            min-width: 50px;
            font-family: 'Fira Code', monospace;
            flex-shrink: 0;
        }
        .line-numbers span {
            display: block;
        }
        .editor-content {
            position: relative;
            flex: 1;
            display: flex;
            min-width: 0;
        }
        #proofInput {
            width: 100%;
            background: transparent;
            border: none;
            color: #e3f2ec;
            padding: 14px 18px;
            font-size: 15px;
            line-height: 1.7;
            resize: vertical;
            outline: none;
            font-family: 'Fira Code', 'Consolas', monospace;
            letter-spacing: 0.2px;
            min-height: 240px;
            z-index: 2;
            caret-color: #52c9b2;
        }
        #proofInput::selection {
            background: #2b6b5e88;
        }
        /* 覆盖层用于显示高亮 */
        .editor-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            pointer-events: none;
            z-index: 1;
            overflow: hidden;
            padding: 14px 18px;
            font-size: 15px;
            line-height: 1.7;
            font-family: 'Fira Code', 'Consolas', monospace;
            letter-spacing: 0.2px;
            white-space: pre-wrap;
            word-break: break-word;
            color: transparent;
            border-radius: 20px;
        }
        .editor-overlay .highlight-line {
            display: block;
            min-height: 1.7em;
            position: relative;
        }
        .editor-overlay .highlight-line.compile-error-line {
            background: rgba(200, 60, 60, 0.18);
            border-radius: 4px;
            box-shadow: -2px 0 0 #e06262, 2px 0 0 #e06262;
            border-left: 3px solid #ff5e5e;
            padding-left: 4px;
            margin-left: -4px;
        }
        .editor-overlay .highlight-line.no-basis-line {
            background: rgba(230, 180, 40, 0.15);
            border-radius: 4px;
            box-shadow: -2px 0 0 #d4a020, 2px 0 0 #d4a020;
            border-left: 3px solid #f0b020;
            padding-left: 4px;
            margin-left: -4px;
        }
        /* 右侧按钮列 */
        .line-buttons-column {
            flex-shrink: 0;
            width: 40px;
            padding: 14px 4px 14px 0;
            display: flex;
            flex-direction: column;
            background: #0d1c21;
            border-top-right-radius: 20px;
            border-bottom-right-radius: 20px;
            border-left: 1px solid #1e3f3a;
        }
        .line-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 28px;
            height: 26px;
            background: #1e353a;
            border-radius: 6px;
            color: #7faaa0;
            cursor: pointer;
            font-size: 16px;
            line-height: 1;
            border: 1px solid #2d5850;
            transition: 0.15s;
            opacity: 0.75;
            user-select: none;
            margin-bottom: 2px;
            font-weight: bold;
            letter-spacing: 1px;
        }
        .line-btn:hover {
            opacity: 1;
            background: #2a5450;
            color: #d4f2e8;
            border-color: #52c9b2;
        }
        /* 提示气泡 */
        .tooltip {
            position: fixed;
            background: #142a2e;
            border: 1px solid #3f8f7f;
            color: #dcf2ea;
            padding: 8px 14px;
            border-radius: 12px;
            font-size: 13px;
            max-width: 500px;
            box-shadow: 0 8px 24px #000000cc;
            pointer-events: none;
            z-index: 9999;
            font-family: 'Segoe UI', sans-serif;
            line-height: 1.5;
            white-space: pre-wrap;
            word-break: break-word;
            display: none;
        }
        .tooltip::after {
            content: '';
            position: absolute;
            left: 20px;
            bottom: -8px;
            border-width: 8px 8px 0 8px;
            border-style: solid;
            border-color: #3f8f7f transparent transparent transparent;
        }

        .button-group {
            display: flex;
            flex-wrap: wrap;
            gap: 14px;
            align-items: center;
            justify-content: space-between;
            margin: 10px 0 8px;
        }
        button {
            background: #256b5f;
            border: none;
            color: white;
            font-size: 1rem;
            font-weight: 500;
            padding: 12px 26px;
            border-radius: 40px;
            cursor: pointer;
            transition: 0.15s;
            box-shadow: 0 6px 0 #0b2e28, 0 6px 12px black;
            font-family: 'Segoe UI', sans-serif;
            letter-spacing: 0.8px;
            border: 1px solid #49b8a4;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        button:active {
            transform: translateY(5px);
            box-shadow: 0 1px 0 #0b2e28, 0 8px 16px black;
        }
        button.secondary {
            background: #2f4045;
            border-color: #4f7370;
            box-shadow: 0 6px 0 #192326, 0 6px 12px black;
        }
        .checkbox-group {
            display: flex;
            align-items: center;
            gap: 12px;
            background: #102227;
            padding: 6px 16px 6px 20px;
            border-radius: 40px;
            border: 1px solid #2c5850;
        }
        .checkbox-group label {
            cursor: pointer;
            font-weight: 400;
            color: #b1d4cb;
        }
        input[type="checkbox"] {
            width: 18px;
            height: 18px;
            accent-color: #2ea88f;
            transform: scale(1.1);
            cursor: pointer;
        }
        .output-area {
            background: #0b1419;
            border-radius: 18px;
            padding: 18px 22px;
            margin-top: 22px;
            border: 1px solid #1e3f3a;
            min-height: 140px;
            max-height: 300px;
            overflow-y: auto;
            font-size: 14px;
            line-height: 1.7;
            white-space: pre-wrap;
            word-break: break-word;
            box-shadow: inset 0 0 20px #00000055;
            color: #cbe3db;
            font-family: 'Fira Code', 'Consolas', monospace;
        }
        .output-area .error {
            color: #fdaaaa;
            background: #401e1e50;
            border-left: 6px solid #e06262;
            padding-left: 14px;
            border-radius: 6px;
        }
        .output-area .warning {
            color: #f5d88b;
            background: #40381e50;
            border-left: 6px solid #d4a020;
            padding-left: 14px;
            border-radius: 6px;
        }
        .output-area .success {
            color: #b1f0d9;
            background: #155a4d30;
            border-left: 6px solid #27d4a8;
            padding-left: 14px;
            border-radius: 6px;
        }
        .footer {
            text-align: center;
            margin-top: 18px;
            font-size: 0.8rem;
            color: #456f69;
            font-family: 'Segoe UI', sans-serif;
        }
    </style>
</head>
<body>
<div class="container">
    <h1>🧩 形式系统 H 编译器</h1>
    <div class="subtitle">数学女孩3 · 哥德尔不完备定理 · 第5章   (增强交互版)</div>

    <div class="rules">
        <p><span>📌 符号定义：</span>定义 (A)→(B) 为 (¬(A))∨(B)</p>
        <p><span>⚙️ 推理规则 MP：</span>由 A 和 (A)→(B) 可以得到 B</p>
        <p><span>📚 公理模式：</span></p>
        <p style="padding-left:20px; color:#a9d9cf;">L1: (A)→((A)∨(B)) &nbsp;|&nbsp; L2: ((A)∨(B))→((B)∨(A)) &nbsp;|&nbsp; L3: ((A)→(B))→(((C)∨(A))→((C)∨(B))) &nbsp;|&nbsp; 恒等: ((A)∨(A))→(A)</p>
    </div>

    <div class="editor-wrapper" id="editorWrapper">
        <div class="line-numbers" id="lineNumbers"></div>
        <div class="editor-content">
            <textarea id="proofInput" rows="9" spellcheck="false"></textarea>
            <div class="editor-overlay" id="editorOverlay"></div>
        </div>
        <div class="line-buttons-column" id="lineButtonsColumn"></div>
    </div>

    <div class="button-group">
        <div style="display: flex; gap: 12px; flex-wrap: wrap;">
            <button id="runBtn">▶ 编译并验证</button>
            <button id="clearBtn" class="secondary">🗑 清空输出</button>
        </div>
        <div class="checkbox-group">
            <input type="checkbox" id="traceBackCheck" checked>
            <label for="traceBackCheck">回溯检查（严格模式）</label>
        </div>
    </div>

    <div id="output" class="output-area">
        <span style="color:#7faaa0;">点击「编译并验证」开始...</span>
    </div>
    <div class="footer">Made by (-1)^0.5 · 移植至 HTML · 增强交互</div>
</div>

<!-- 悬浮提示框 -->
<div id="globalTooltip" class="tooltip"></div>

<script>
    // ============================================================
    // 1. 形式系统 H 核心 (移植自 main.py)
    // ============================================================
    const gongli0 = [
        '((A)∨(A))→(A)',
        '(A)→((A)∨(B))',
        '((A)∨(B))→((B)∨(A))',
        '((A)→(B))→(((C)∨(A))→((C)∨(B)))'
    ];

    let gongli = [];

    class CompileError extends Error {
        constructor(message, pos) {
            super(message);
            this.name = 'CompileError';
            this.pos = pos;
        }
    }

    class Struct {
        constructor() {
            this.a = null;
            this.symbol = '';
            this.b = null;
            this.name = '';
        }
        init(a, symbol, b) {
            this.a = a;
            this.symbol = symbol;
            this.b = b;
            return this;
        }
        toString() {
            let a = typeof this.a === 'string' ? this.a : this.a.toString();
            let b = typeof this.b === 'string' ? this.b : this.b.toString();
            if (a.length > 1) a = '(' + a + ')';
            if (b.length > 1) b = '(' + b + ')';
            if (this.symbol === '¬') return '¬' + b;
            return a + this.symbol + b;
        }
    }

    function isStruct(x) { return x instanceof Struct; }

    function find(word, start) {
        let kh = 0;
        let index0 = -1;
        let yunsuanbuming = false;
        for (let i = 0; i < word.length; i++) {
            const w = word[i];
            const o = w.charCodeAt(0);
            if (w === '(') kh += 1;
            else if (w === ')') {
                kh -= 1;
                if (kh < 0) throw new CompileError('括号异常, 整行表达式中")"更多', i + start);
            } else if (!((o >= 97 && o < 123) || (o >= 65 && o < 91) || '¬∨∧→'.includes(w))) {
                throw new CompileError('未知字符', i + start);
            } else if (kh === 0) {
                if (index0 !== -1) yunsuanbuming = true;
                index0 = i;
            }
        }
        if (kh > 0) throw new CompileError(`括号异常, 整行表达式中"("多${kh}个, 请仔细检查`, word.length - 1);
        if (yunsuanbuming) throw new CompileError('运算表达不明确, 请使用"()"将参与运算的部分括起来', index0 + start);
        if (index0 === -1) throw new CompileError('禁止连续两层括号', start);
        const symbol = word[index0];
        return { index: index0, symbol };
    }

    function transform(s) {
        if (s.includes('#')) s = s.split('#')[0];
        s = s.replace(/!/g, '¬').replace(/\|/g, '∨').replace(/or/g, '∨')
             .replace(/&/g, '∧').replace(/and/g, '∧').replace(/to/g, '→')
             .replace(/ /g, '').replace(/\*/g, '');
        s = s.replace(/[A-Za-z]/g, function(match, offset, string) {
            const start = offset, end = offset + match.length;
            if (start > 0 && end < string.length && string[start - 1] === '(' && string[end] === ')') {
                return match;
            }
            return `(${match})`;
        });
        return s;
    }

    function main(word, start = 0) {
        if (word.length === 1) {
            const a = word[0];
            if (!'¬∨∧→←'.includes(a)) return a;
        }
        const self = new Struct();
        const { index: i, symbol } = find(word, start);
        self.symbol = symbol;
        if (symbol === '¬') {
            if (i > 0) throw new CompileError('"¬"前禁止接其它内容', start + i - 1);
            self.a = ' ';
            self.b = main(word.slice(i + 2, -1), start + i + 2);
        } else {
            if (i === 0) throw new CompileError('运算符前缺少内容', i + start);
            else if (i === word.length - 1) throw new CompileError('运算符前缺少内容', i + start);
            self.a = main(word.slice(1, i - 1), start + 1);
            self.b = main(word.slice(i + 2, -1), start + i + 2);
        }
        return self;
    }

    function is2(line1, line2) {
        const dic = {};
        const result = is3(line1, line2, dic);
        return { result, dic };
    }
    function is3(line1, line2, dic) {
    if (typeof line2 === 'string') {
        if (line2 in dic) {
            // 使用结构相等比较
            const stored = dic[line2];
            if (typeof stored === 'string' && typeof line1 === 'string') {
                if (stored !== line1) return false;
            } else if (typeof stored === 'object' && typeof line1 === 'object') {
                if (!structEqual(stored, line1)) return false;
            } else {
                return false;
            }
        } else {
            dic[line2] = line1;
        }
        return true;
    }
    if (typeof line1 === 'string') return false;
    return line1.symbol === line2.symbol &&
        (line2.symbol === '¬' || is3(line1.a, line2.a, dic)) &&
        is3(line1.b, line2.b, dic);
    }

    function to2notor(line) {
        if (typeof line === 'string') return line;
        if (line.symbol === '→') {
            const notA = new Struct().init(' ', '¬', to2notor(line.a));
            return new Struct().init(notA, '∨', to2notor(line.b));
        }
        return line;
    }

    function structEqual(a, b) {
        if (typeof a === 'string' && typeof b === 'string') return a === b;
        if (typeof a !== typeof b) return false;
        if (typeof a === 'string') return false;
        if (a.symbol !== b.symbol) return false;
        if (a.symbol === '¬') return structEqual(a.b, b.b);
        return structEqual(a.a, b.a) && structEqual(a.b, b.b);
    }

    function panduan(line, lines, print_r, outputLines) {
        if (typeof line === 'string') return false;
        for (const P of [...gongli, ...lines]) {
            const { result: t, dic } = is2(line, P);
            if (t) {
                let w0 = '  其中, 替换 ';
                let w = '';
                for (const a in dic) {
                    const b = dic[a];
                    if (a !== b) w += `${a}:${b}, `;
                }
                if (w.length > 0) w = w0 + w.slice(0, -2);
                if (print_r) outputLines.push(`${line.name}: ${line}     依据: ${P.name}${w}`);
                return true;
            }
            const line2not = to2notor(line);
            const p2not = to2notor(P);
            if (structEqual(line2not, p2not)) {
                if (print_r) outputLines.push(`${line.name}: ${line}     依据: ${P.name}    等价变形`);
                return true;
            }
            if (P.symbol === '→') {
                for (const P2 of lines) {
                    if (structEqual(P2, P.a) && structEqual(line, P.b)) {
                        if (print_r) outputLines.push(`${line.name}: ${line}     依据: 推理规则MP  ${P2.name}  ${P.name}`);
                        return true;
                    }
                }
            }
        }
        return false;
    }

    function getJustification(line, lines) {
        if (typeof line === 'string') return '未知（非公式）';
        for (const P of [...gongli, ...lines]) {
            const { result: t, dic } = is2(line, P);
            if (t) {
                let w0 = '  其中, 替换 ';
                let w = '';
                for (const a in dic) {
                    const b = dic[a];
                    if (a !== b) w += `${a}:${b}, `;
                }
                if (w.length > 0) w = w0 + w.slice(0, -2);
                return `依据: ${P.name}${w}`;
            }
            const line2not = to2notor(line);
            const p2not = to2notor(P);
            if (structEqual(line2not, p2not)) {
                return `依据: ${P.name}    等价变形`;
            }
            if (P.symbol === '→') {
                for (const P2 of lines) {
                    if (structEqual(P2, P.a) && structEqual(line, P.b)) {
                        return `依据: 推理规则MP  ${P2.name}  ${P.name}`;
                    }
                }
            }
        }
        return '无法找到依据';
    }

    function test(prove, trace_back = false, print_r = true, n = 0, outputLines = []) {
        if (n > 500) {
            if (print_r) outputLines.push('引用次数过多, 请检查是否存在循环论证');
            return { success: false, outputLines, justifications: {}, errorType: 'compile', errorLine: -1 };
        }

        if (print_r) {
            outputLines.push(`定义(A)→(B)为(¬(A))∨(B)`);
            for (const P of gongli) outputLines.push(`${P.name}: ${P}`);
            outputLines.push('推理规则MP: 由A和(A)→(B)可以得到B');
            outputLines.push('');
        }

        const lines = [];
        let ind = 0;
        const dingli = [];
        const justifications = {};

        const proveLines = prove.split('\n');
        for (const word0 of proveLines) {
            ind += 1;
            if (word0.trim().startsWith('import ')) {
                if (print_r) outputLines.push(`第${ind}行: ${word0}   (网页版暂不支持文件导入，已忽略)`);
                continue;
            }

            const word = transform(word0);
            if (word.length === 0) {
                if (print_r) outputLines.push(`第${ind}行: ${word0}`);
                continue;
            }

            let line;
            try {
                line = main(word);
                if (typeof line === 'string') throw new CompileError('写这个是何意味? ', 0);
                line.name = `第${ind}行`;
            } catch (e) {
                if (print_r) {
                    outputLines.push(`编译第${ind}行时发生错误:`);
                    let mes = e.message;
                    let pos = e.pos !== undefined ? e.pos : 0;
                    outputLines.push(word);
                    outputLines.push(' '.repeat(pos) + '↑');
                    outputLines.push(mes);
                }
                justifications[ind] = `编译错误: ${e.message}`;
                return { success: false, outputLines, justifications, errorType: 'compile', errorLine: ind, errorPos: e.pos };
            }

            const just = getJustification(line, lines);
            justifications[ind] = just;

            if (!panduan(line, lines, print_r, outputLines)) {
                if (print_r) outputLines.push(`第${ind}行: ${line}     无法找到依据`);
                return { success: false, outputLines, justifications, errorType: 'no_basis', errorLine: ind, errorPos: null };
            }

            lines.push(line);
            if (word0.includes('*')) dingli.push(line);
        }

        if (dingli.length > 0 && print_r) {
            outputLines.push('总结, 已证明: ');
            for (const d of dingli) outputLines.push(d.toString());
        }
        return { success: true, outputLines, justifications, errorType: null, errorLine: -1 };
    }

    function initGongli() {
        gongli = [];
        for (let i = 0; i < gongli0.length; i++) {
            try {
                const transformed = transform(gongli0[i]);
                const obj = main(transformed);
                obj.name = `公理${i + 1}`;
                gongli.push(obj);
            } catch (e) {
                console.error('公理初始化失败', e);
            }
        }
    }
    initGongli();

    // ============================================================
    // 2. 界面交互与增强功能
    // ============================================================
    const proofInput = document.getElementById('proofInput');
    const outputDiv = document.getElementById('output');
    const runBtn = document.getElementById('runBtn');
    const clearBtn = document.getElementById('clearBtn');
    const traceBackCheck = document.getElementById('traceBackCheck');
    const editorOverlay = document.getElementById('editorOverlay');
    const lineNumbersDiv = document.getElementById('lineNumbers');
    const lineButtonsColumn = document.getElementById('lineButtonsColumn');
    const tooltip = document.getElementById('globalTooltip');

    let lastJustifications = {};
    let errorLine = -1;
    let errorType = null; // 'compile' | 'no_basis' | null

    // ---- 自动替换输入符号 ----
    proofInput.addEventListener('input', function(e) {
        let text = this.value;
        const originalText = text;
        text = text.replace(/!/g, '¬')
                   .replace(/\|/g, '∨')
                   .replace(/&/g, '∧')
                   .replace(/to/g, '→')
                   .replace(/or/g, '∨')
                   .replace(/and/g, '∧');
        if (text !== originalText) {
            const cursorPos = this.selectionStart;
            const oldLen = originalText.length;
            const newLen = text.length;
            this.value = text;
            const diff = newLen - oldLen;
            this.selectionStart = this.selectionEnd = cursorPos + diff;
        }
        updateEditorVisuals();
    });

    // ---- 更新行号、覆盖层与行按钮 ----
    function updateEditorVisuals() {
        const text = proofInput.value;
        const lines = text.split('\n');
        // 更新行号
        lineNumbersDiv.innerHTML = lines.map((_, i) => `<span>${i + 1}</span>`).join('');

        // 更新覆盖层（错误行高亮）
        editorOverlay.innerHTML = '';
        for (let i = 0; i < lines.length; i++) {
            const lineText = lines[i];
            const lineDiv = document.createElement('div');
            lineDiv.className = 'highlight-line';
            lineDiv.style.position = 'relative';
            lineDiv.style.whiteSpace = 'pre-wrap';
            lineDiv.style.wordBreak = 'break-word';
            lineDiv.style.minHeight = '1.7em';

            const isError = (i + 1 === errorLine);
            if (isError) {
                if (errorType === 'compile') {
                    lineDiv.classList.add('compile-error-line');
                } else if (errorType === 'no_basis') {
                    lineDiv.classList.add('no-basis-line');
                }
            }

            const textSpan = document.createElement('span');
            textSpan.textContent = lineText || ' ';
            textSpan.style.color = 'transparent';
            textSpan.style.whiteSpace = 'pre-wrap';
            lineDiv.appendChild(textSpan);

            editorOverlay.appendChild(lineDiv);
        }

        // 更新右侧按钮列
        lineButtonsColumn.innerHTML = '';
        for (let i = 0; i < lines.length; i++) {
            const btn = document.createElement('div');
            btn.className = 'line-btn';
            btn.textContent = '⋯';
            btn.dataset.line = i + 1;
            btn.addEventListener('mouseenter', (e) => {
                const lineNum = parseInt(e.target.dataset.line);
                const just = lastJustifications[lineNum] || '尚未验证，请先点击「编译并验证」';
                showTooltip(e.target, just);
            });
            btn.addEventListener('mouseleave', hideTooltip);
            lineButtonsColumn.appendChild(btn);
        }

        // 同步滚动
        syncScroll();
    }

    // 同步 textarea 与 overlay 的滚动
    function syncScroll() {
        if (editorOverlay && proofInput) {
            editorOverlay.scrollTop = proofInput.scrollTop;
            editorOverlay.scrollLeft = proofInput.scrollLeft;
        }
        if (lineButtonsColumn && proofInput) {
            // 按钮列与 textarea 垂直滚动同步
            lineButtonsColumn.scrollTop = proofInput.scrollTop;
        }
    }

    proofInput.addEventListener('scroll', syncScroll);

    window.addEventListener('resize', () => {
        syncScroll();
    });

    // ---- 悬浮提示框 ----
    function showTooltip(element, text) {
        const rect = element.getBoundingClientRect();
        tooltip.textContent = text;
        tooltip.style.display = 'block';
        const tooltipRect = tooltip.getBoundingClientRect();
        let left = rect.left + window.scrollX - 40;
        let top = rect.top + window.scrollY - tooltipRect.height - 8;
        if (left < 10) left = 10;
        if (top < 10) top = rect.bottom + window.scrollY + 8;
        tooltip.style.left = left + 'px';
        tooltip.style.top = top + 'px';
    }

    function hideTooltip() {
        tooltip.style.display = 'none';
    }

    // ---- 显示输出 ----
    function displayOutput(lines, isError = false) {
        outputDiv.innerHTML = '';
        const frag = document.createDocumentFragment();
        lines.forEach(line => {
            const span = document.createElement('span');
            span.className = 'line';
            // 区分编译错误和无法找到依据
            if (line.includes('无法找到依据')) {
                span.classList.add('warning');
            } else if (line.includes('编译') && line.includes('错误') || line.includes('异常')) {
                span.classList.add('error');
            } else if (line.includes('合法') || line.includes('证明是合法')) {
                span.classList.add('success');
            }
            span.textContent = line;
            frag.appendChild(span);
            frag.appendChild(document.createElement('br'));
        });
        outputDiv.appendChild(frag);
        outputDiv.scrollTop = 0;
    }

    // ---- 运行验证 ----
    runBtn.addEventListener('click', () => {
        const proveText = proofInput.value;
        const traceBack = traceBackCheck.checked;
        const outputLines = [];

        outputLines.push('正在检验 证明\n');
        const result = test(proveText, traceBack, true, 0, outputLines);

        lastJustifications = result.justifications || {};
        errorLine = result.errorLine !== undefined ? result.errorLine : -1;
        errorType = result.errorType || null;

        if (result.success) {
            result.outputLines.push('\n经检验, 证明是合法的');
        } else {
            result.outputLines.push('\n证明验证失败');
        }
        displayOutput(result.outputLines);

        updateEditorVisuals();
    });

    clearBtn.addEventListener('click', () => {
        outputDiv.innerHTML = '<span style="color:#7faaa0;">输出已清空。点击「编译并验证」重新开始。</span>';
        errorLine = -1;
        errorType = null;
        updateEditorVisuals();
    });

    // ---- 初始化示例 ----
    window.addEventListener('load', () => {
        proofInput.value = `# 假言三段论
(B→C)→(((¬A)∨B)→((¬A)∨C))
(B→C)→((A→B)→(A→C))  *
#*abb ((B→C),(A→B))→(A→C):(B→C)→((A→B)→(A→C));(A→B)→(A→C);A→C
# 自己推自己
(A∨A)→A
A→(A∨A)
((A∨A)→A)to((A→(A∨A))to(AtoA))
(A→(A∨A))to(AtoA)
A→A  *
(¬A)∨A
((¬A)∨A)→(A∨(¬A))
A∨(¬A)  *
# 双重否定表肯定
(¬A)→(¬A)
(¬(¬A))∨(¬A)
(¬A)∨(¬(¬A))
A→(¬(¬A))  *
`;
        updateEditorVisuals();
        setTimeout(() => runBtn.click(), 200);
    });

    window.HSystem = { main, transform, test, gongli };
</script>
</body>
</html>
