---
title: "璃月人民大学教务处百分制均分计算器"
date: 2025-07-03T14:41:31+08:00
draft: false
description: "自动计算教务系统成绩查询页面的百分制课程加权平均分的油猴脚本"
image : ""
categories:
- 技术分享
---
在AI加持下完成的自动计算璃月人民大学教务系统成绩查询页面的百分制课程加权平均分的油猴脚本，比较狗屎就不挂Github了。

```
// ==UserScript==
// @name         璃月人民大学教务处百分制均分计算器
// @version      1.0
// @description  自动计算教务系统成绩查询页面的百分制课程加权平均分，并显示在右下角。
// @author       xZhongjie
// @match        https://jw.ruc.edu.cn/Njw2017/index.html
// @grant        none
// @license      MIT
// ==/UserScript==

(function() {
    'use strict';

    // 只有在成绩查询页面才执行
    if (window.location.hash !== '#/student/course-score-search/') {
        // 使用 hashchange 事件来监听 URL 的变化，以适应单页面应用 (SPA)
        window.addEventListener('hashchange', function() {
            if (window.location.hash === '#/student/course-score-search/') {
                // 延迟初始化以确保页面元素加载完毕
                setTimeout(initialize, 1000);
            }
        });
        return; // 如果初始加载不是目标页面，则退出
    }

    // --- 样式和UI元素创建 ---
    function createDisplayBox() {
        const box = document.createElement('div');
        box.id = 'gpa-calculator-box';
        Object.assign(box.style, {
            position: 'fixed',
            bottom: '20px',
            right: '20px',
            width: '240px',
            padding: '15px',
            backgroundColor: '#f0f9eb',
            border: '1px solid #e1f3d8',
            borderRadius: '8px',
            boxShadow: '0 2px 12px 0 rgba(0, 0, 0, 0.1)',
            color: '#67c23a',
            fontSize: '16px',
            lineHeight: '1.5',
            zIndex: '9999',
            fontFamily: 'Helvetica, Arial, sans-serif',
            transition: 'opacity 0.3s'
        });
        document.body.appendChild(box);
        return box;
    }

    // --- 核心计算逻辑 ---
    function calculateAndDisplay() {
        const displayBox = document.getElementById('gpa-calculator-box');
        if (!displayBox) return; // 如果窗口不存在则不执行

        displayBox.innerHTML = '正在计算...';

        const table = document.querySelector('table.table-border');
        if (!table) {
            displayBox.innerHTML = '未找到成绩表格';
            return;
        }

        const headers = Array.from(table.querySelectorAll('th'));
        // 通过查找表头文字来确定列的索引，使脚本更健壮
        const creditIndex = headers.findIndex(th => th.textContent.trim() === '学分');
        const scoreIndex = headers.findIndex(th => th.textContent.trim() === '最终成绩');

        if (creditIndex === -1 || scoreIndex === -1) {
            displayBox.style.backgroundColor = '#fef0f0';
            displayBox.style.color = '#f56c6c';
            displayBox.innerHTML = '错误：无法定位“学分”或“最终成绩”列。';
            return;
        }

        const rows = table.querySelectorAll('tbody > tr');
        let totalScorePoints = 0;
        let totalCredits = 0;
        let courseCount = 0;

        rows.forEach(row => {
            const cells = row.querySelectorAll('td');
            // 过滤掉学期小计和总计行（这些行通常只有一个单元格或带有 colspan）
            if (cells.length < Math.max(creditIndex, scoreIndex) + 1 || cells[0].hasAttribute('colspan')) {
                return;
            }

            const creditText = cells[creditIndex].textContent.trim();
            const scoreText = cells[scoreIndex].textContent.trim();

            // 尝试将成绩转换为数字，如果失败（例如成绩是'A', 'P'等），则忽略该行
            const score = parseFloat(scoreText);
            if (!isNaN(score) && isFinite(score)) {
                const credit = parseFloat(creditText);
                if (!isNaN(credit) && credit > 0) {
                    totalScorePoints += score * credit;
                    totalCredits += credit;
                    courseCount++;
                }
            }
        });

        if (totalCredits > 0) {
            const average = (totalScorePoints / totalCredits).toFixed(3); // 保留三位小数
            displayBox.innerHTML = `
                <strong style="font-size: 18px;">百分制均分: ${average}</strong><br>
                <small style="color: #909399;">(基于 ${courseCount} 门课程, 总计 ${totalCredits} 学分)</small>
            `;
        } else {
            displayBox.innerHTML = '无符合条件的百分制课程';
        }
    }

    // --- 延迟和防抖动 ---
    function debounce(func, delay) {
        let timeout;
        return function(...args) {
            clearTimeout(timeout);
            timeout = setTimeout(() => func.apply(this, args), delay);
        };
    }

    // --- 初始化和DOM监听 ---
    function initialize() {
        // 检查显示窗口是否已存在
        if (document.getElementById('gpa-calculator-box')) {
            calculateAndDisplay(); // 如果存在，直接计算
            return;
        }

        const displayBox = createDisplayBox();
        const debouncedCalc = debounce(calculateAndDisplay, 500); // 500ms 防抖

        // 使用 MutationObserver 监听 DOM 变化，以应对 Vue.js 动态加载内容
        const observer = new MutationObserver((mutations) => {
            for (const mutation of mutations) {
                if (mutation.type === 'childList') {
                    // 只要内容区域发生变化，就触发重新计算
                    debouncedCalc();
                    return; // 找到变化后即可，无需继续遍历
                }
            }
        });

        // 监听整个应用容器的变化
        const targetNode = document.querySelector('.content-tabs') || document.getElementById('app');
        if (targetNode) {
            observer.observe(targetNode, {
                childList: true, // 监视子节点的添加或删除
                subtree: true      // 监视所有后代节点
            });
            // 首次加载时手动执行一次计算
            debouncedCalc();
        } else {
            displayBox.innerHTML = '脚本初始化失败: 未找到监听目标。';
        }
    }

    // 主程序入口
    // 延迟 1 秒执行，以确保 Vue 应用已挂载
    setTimeout(initialize, 1000);

})();
```

