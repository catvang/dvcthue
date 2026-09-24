// ==UserScript==
// @name         meInvoice - Chọn Kỳ Đồng Bộ Hóa Đơn (Tối đa 90 ngày)
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Bảng chọn kỳ nhanh cho chức năng Đồng bộ bằng công cụ (Hạn mức <= 90 ngày)
// @author       You
// @match        https://meinvoice.vn/*
// @match        https://*.meinvoice.vn/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const thisYear = new Date().getFullYear();
    let currentYear = thisYear;

    // Helper: Định dạng dd/mm/yyyy
    function fmtDate(d, m, y) {
        return `${String(d).padStart(2, '0')}/${String(m).padStart(2, '0')}/${y}`;
    }

    // Helper: Kích hoạt sự kiện để Vue / meInvoice nhận giá trị ô input
    function setInputValue(input, val) {
        input.focus();
        input.value = val;
        input.dispatchEvent(new Event('input', { bubbles: true }));
        input.dispatchEvent(new Event('change', { bubbles: true }));
        input.dispatchEvent(new Event('blur', { bubbles: true }));
    }

    // Tìm và bấm nút mở hộp thoại "Đồng bộ bằng công cụ"
    function openSyncDialog() {
        const items = document.querySelectorAll('.item-text');
        for (let item of items) {
            if (item.innerText.includes('Đồng bộ bằng công cụ')) {
                item.click();
                return true;
            }
        }
        return false;
    }

    // Tìm nút chạy / xác nhận đồng bộ trong popup
    function clickRunSyncButton() {
        // Tìm button chứa chữ "Đồng bộ" hoặc class nút chính của MISA
        const buttons = document.querySelectorAll('button, .ms-button, .ms-button-text, .btn');
        for (let btn of buttons) {
            const txt = btn.innerText.trim();
            if (txt === 'Đồng bộ' || txt === 'Thực hiện' || txt === 'Đồng ý') {
                btn.click();
                console.log(`[Đồng bộ] Đã bấm nút: ${txt}`);
                return;
            }
        }
    }

    // Mở hộp thoại, điền ngày và chạy đồng bộ
    async function applySyncPeriod(fromDateStr, toDateStr) {
        // 1. Mở popup đồng bộ nếu đang đóng
        openSyncDialog();

        // Chờ 1.5 - 2 giây để popup render hoàn chỉnh
        await new Promise(r => setTimeout(r, 1800));

        // 2. Tìm 2 ô Từ ngày & Đến ngày bên trong popup đồng bộ
        const dateInputs = document.querySelectorAll('input.ms-input--datepicker[placeholder="dd/mm/yyyy"]');
        if (dateInputs.length >= 2) {
            // Lấy 2 input datepicker cuối cùng (thường thuộc về dialog đang active nổi lên trên)
            const fromInput = dateInputs[dateInputs.length - 2];
            const toInput = dateInputs[dateInputs.length - 1];

            setInputValue(fromInput, fromDateStr);
            setInputValue(toInput, toDateStr);
            console.log(`[Đồng bộ] Đã điền khoảng ngày: ${fromDateStr} -> ${toDateStr}`);

            // Chờ 400ms để form lưu state rồi bấm chạy đồng bộ
            await new Promise(r => setTimeout(r, 400));
            clickRunSyncButton();
        } else {
            console.warn('[Đồng bộ] Chưa tìm thấy 2 ô input Từ ngày / Đến ngày trong hộp thoại!');
        }
    }

    // Xử lý chọn Tháng / Quý
    function onSelectPeriod(type, val) {
        let fromDate = '', toDate = '';
        const y = currentYear;

        if (type === 'M') {
            const m = parseInt(val, 10);
            const lastDay = new Date(y, m, 0).getDate();
            fromDate = fmtDate(1, m, y);
            toDate = fmtDate(lastDay, m, y);
        } else if (type === 'Q') {
            const q = parseInt(val, 10);
            const startMonth = (q - 1) * 3 + 1;
            const endMonth = q * 3;
            const lastDay = new Date(y, endMonth, 0).getDate();
            fromDate = fmtDate(1, startMonth, y);
            toDate = fmtDate(lastDay, endMonth, y);
        }

        // Đổi trạng thái active cho nút bấm
        document.querySelectorAll('.mi-sync-btn').forEach(b => b.classList.remove('active'));
        const activeBtn = document.querySelector(`.mi-sync-btn[data-type="${type}"][data-val="${val}"]`);
        if (activeBtn) activeBtn.classList.add('active');

        applySyncPeriod(fromDate, toDate);
    }

    // Tạo bảng điều khiển nổi ở góc dưới bên phải
    function injectSyncPanel() {
        if (document.getElementById('mein-sync-panel')) return;

        // Danh sách 5 năm gần nhất
        const years = [];
        for (let i = 4; i >= 0; i--) {
            years.push(thisYear - i);
        }

        const panel = document.createElement('div');
        panel.id = 'mein-sync-panel';
        panel.innerHTML = `
            <style>
                #mein-sync-panel {
                    position: fixed;
                    bottom: 25px;
                    right: 25px;
                    z-index: 999999;
                    background: #ffffff;
                    border: 1px solid #cbd5e1;
                    box-shadow: 0 8px 24px rgba(0,0,0,0.18);
                    border-radius: 8px;
                    padding: 10px 12px;
                    width: 320px;
                    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
                }
                .sync-panel-header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    margin-bottom: 8px;
                }
                .sync-panel-title {
                    font-size: 13px;
                    font-weight: bold;
                    color: #0f172a;
                }
                .sync-year-stepper {
                    display: flex;
                    align-items: center;
                    gap: 3px;
                }
                .sync-btn-step {
                    width: 22px;
                    height: 22px;
                    border: 1px solid #cbd5e1;
                    background: #f8fafc;
                    cursor: pointer;
                    font-weight: bold;
                    border-radius: 4px;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                }
                .sync-year-val {
                    font-size: 12px;
                    font-weight: bold;
                    width: 44px;
                    text-align: center;
                    border: 1px solid #cbd5e1;
                    border-radius: 4px;
                    padding: 2px 0;
                }
                /* 5 năm gần nhất - chỉ đổi năm làm việc, không tự điền ngày */
                .sync-years-row {
                    display: grid;
                    grid-template-columns: repeat(5, 1fr);
                    gap: 4px;
                    margin-bottom: 8px;
                }
                .sync-btn-year {
                    padding: 4px 0;
                    font-size: 11px;
                    font-weight: bold;
                    border: 1px solid #cbd5e1;
                    background: #f1f5f9;
                    color: #334155;
                    border-radius: 4px;
                    cursor: pointer;
                    text-align: center;
                    transition: 0.1s;
                }
                .sync-btn-year:hover { background: #e2e8f0; }
                .sync-btn-year.active {
                    background: #059669;
                    color: #fff;
                    border-color: #047857;
                }

                .sync-btn-grid {
                    display: grid;
                    grid-template-columns: repeat(6, 1fr);
                    gap: 4px;
                }
                .mi-sync-btn {
                    padding: 5px 0;
                    font-size: 11.5px;
                    font-weight: 600;
                    border: 1px solid #cbd5e1;
                    background: #f8fafc;
                    color: #334155;
                    border-radius: 4px;
                    cursor: pointer;
                    text-align: center;
                    transition: 0.1s;
                }
                .mi-sync-btn:hover { background: #e2e8f0; }
                .mi-sync-btn.active {
                    background: #2563eb;
                    color: #fff;
                    border-color: #1d4ed8;
                }
                .grid-span-3 {
                    grid-column: span 3;
                }
            </style>

            <div class="sync-panel-header">
                <span class="sync-panel-title">🔄 Đồng bộ hóa đơn (≤ 90 ngày)</span>
                <div class="sync-year-stepper">
                    <button class="sync-btn-step" id="sync-btn-prev">−</button>
                    <input type="text" class="sync-year-val" id="sync-year-input" value="${currentYear}">
                    <button class="sync-btn-step" id="sync-btn-next">+</button>
                </div>
            </div>

            <!-- Dải 5 năm gần nhất: Chỉ chọn mốc năm, không điền ngày -->
            <div class="sync-years-row">
                ${years.map(y => `<button class="sync-btn-year ${y === currentYear ? 'active' : ''}" data-year="${y}">${y}</button>`).join('')}
            </div>

            <!-- Bảng nút Tháng và Quý (Đảm bảo thời gian <= 90 ngày) -->
            <div class="sync-btn-grid">
                <!-- Hàng 1: Tháng 1-6 -->
                <button class="mi-sync-btn" data-type="M" data-val="1">T01</button>
                <button class="mi-sync-btn" data-type="M" data-val="2">T02</button>
                <button class="mi-sync-btn" data-type="M" data-val="3">T03</button>
                <button class="mi-sync-btn" data-type="M" data-val="4">T04</button>
                <button class="mi-sync-btn" data-type="M" data-val="5">T05</button>
                <button class="mi-sync-btn" data-type="M" data-val="6">T06</button>

                <!-- Hàng 2: Tháng 7-12 -->
                <button class="mi-sync-btn" data-type="M" data-val="7">T07</button>
                <button class="mi-sync-btn" data-type="M" data-val="8">T08</button>
                <button class="mi-sync-btn" data-type="M" data-val="9">T09</button>
                <button class="mi-sync-btn" data-type="M" data-val="10">T10</button>
                <button class="mi-sync-btn" data-type="M" data-val="11">T11</button>
                <button class="mi-sync-btn" data-type="M" data-val="12">T12</button>

                <!-- Hàng 3: 4 Quý (mỗi quý 3 cột cân đối) -->
                <button class="mi-sync-btn grid-span-3" data-type="Q" data-val="1" style="grid-column: span 1.5;">Q1</button>
                <button class="mi-sync-btn grid-span-3" data-type="Q" data-val="2" style="grid-column: span 1.5;">Q2</button>
                <button class="mi-sync-btn grid-span-3" data-type="Q" data-val="3" style="grid-column: span 1.5;">Q3</button>
                <button class="mi-sync-btn grid-span-3" data-type="Q" data-val="4" style="grid-column: span 1.5;">Q4</button>
                <button class="mi-sync-btn" style="grid-column: span 2; font-size: 11px; background: #e0f2fe; color: #0369a1; border-color: #bae6fd;" id="btn-manual-sync">⚡ Mở đồng bộ</button>
            </div>
        `;

        document.body.appendChild(panel);

        // Đổi năm làm việc (không điền ngày)
        function changeWorkYear(y) {
            currentYear = y;
            document.getElementById('sync-year-input').value = currentYear;
            document.querySelectorAll('.sync-btn-year').forEach(b => {
                b.classList.toggle('active', parseInt(b.getAttribute('data-year'), 10) === currentYear);
            });
            // Xóa active của nút kỳ cũ nếu đổi năm
            document.querySelectorAll('.mi-sync-btn').forEach(b => b.classList.remove('active'));
        }

        document.getElementById('sync-btn-prev').onclick = () => changeWorkYear(currentYear - 1);
        document.getElementById('sync-btn-next').onclick = () => changeWorkYear(currentYear + 1);
        document.getElementById('sync-year-input').onchange = (e) => changeWorkYear(parseInt(e.target.value, 10) || thisYear);

        // Click nút năm: chỉ active năm làm việc
        panel.querySelectorAll('.sync-btn-year').forEach(btn => {
            btn.onclick = () => {
                const y = parseInt(btn.getAttribute('data-year'), 10);
                changeWorkYear(y);
            };
        });

        // Click nút Tháng, Quý
        panel.querySelectorAll('.mi-sync-btn[data-type]').forEach(btn => {
            btn.onclick = () => {
                const t = btn.getAttribute('data-type');
                const v = parseInt(btn.getAttribute('data-val'), 10);
                onSelectPeriod(t, v);
            };
        });

        // Nút mở hộp thoại thủ công nếu cần
        document.getElementById('btn-manual-sync').onclick = () => {
            openSyncDialog();
        };
    }

    // Đợi meInvoice tải xong DOM rồi chèn bảng
    const checkExist = setInterval(() => {
        if (document.body) {
            injectSyncPanel();
            clearInterval(checkExist);
        }
    }, 1000);
})();
