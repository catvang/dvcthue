(function () {
  // Tránh chèn trùng lặp nếu đã chạy trước đó
  const existingToolbar = document.getElementById('tax-filter-toolbar');
  if (existingToolbar) existingToolbar.remove();

  // Tạo khung widget nổi
  const toolbar = document.createElement('div');
  toolbar.id = 'tax-filter-toolbar';
  toolbar.innerHTML = `
    <style>
      #tax-filter-toolbar {
        position: fixed;
        bottom: 20px;
        right: 20px;
        background: #ffffff;
        border-radius: 12px;
        box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
        padding: 16px;
        z-index: 9999999;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
        width: 320px;
        border: 1px solid #cbd5e1;
        user-select: none;
      }
      .t-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 12px;
      }
      .t-title {
        font-weight: bold;
        font-size: 14px;
        color: #0f172a;
      }
      .t-year-control {
        display: flex;
        align-items: center;
        gap: 6px;
      }
      .t-btn-step {
        width: 28px;
        height: 28px;
        background: #e2e8f0;
        border: none;
        border-radius: 6px;
        font-weight: bold;
        cursor: pointer;
        color: #334155;
        display: flex;
        align-items: center;
        justify-content: center;
      }
      .t-btn-step:hover { background: #cbd5e1; }
      .t-year-input {
        width: 65px;
        text-align: center;
        font-weight: bold;
        font-size: 15px;
        border: 1px solid #cbd5e1;
        border-radius: 6px;
        padding: 3px 0;
      }
      .t-grid {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 6px;
        margin-bottom: 8px;
      }
      .t-btn-month {
        padding: 8px 2px;
        font-size: 12px;
        font-weight: 600;
        border: 1px solid #cbd5e1;
        background: #f8fafc;
        border-radius: 6px;
        cursor: pointer;
        color: #1e293b;
        transition: 0.15s;
      }
      .t-btn-month:hover:not(:disabled) {
        background: #2563eb;
        color: #fff;
        border-color: #2563eb;
      }
      .t-btn-month:disabled {
        opacity: 0.35;
        cursor: not-allowed;
        background: #f1f5f9;
        color: #94a3b8;
        border-color: #e2e8f0;
      }
      .t-btn-month.active-curr {
        border-color: #2563eb;
        color: #2563eb;
        background: #eff6ff;
      }
      .t-msg {
        font-size: 11px;
        color: #64748b;
        text-align: center;
        min-height: 14px;
      }
    </style>

    <div class="t-header">
      <span class="t-title">Chọn kỳ tra cứu</span>
      <div class="t-year-control">
        <button class="t-btn-step" id="t-prev-year">-</button>
        <input type="number" id="t-year-val" class="t-year-input" />
        <button class="t-btn-step" id="t-next-year">+</button>
      </div>
    </div>

    <div class="t-grid" id="t-months-grid"></div>
    <div class="t-msg" id="t-status">Bấm tháng để tra cứu</div>
  `;

  document.body.appendChild(toolbar);

  // Xử lý logic thời gian và kích hoạt form
  const now = new Date();
  const currentYear = now.getFullYear();
  const currentMonth = now.getMonth() + 1;

  const yearInput = document.getElementById('t-year-val');
  const monthsGrid = document.getElementById('t-months-grid');
  const statusMsg = document.getElementById('t-status');

  yearInput.value = currentYear;

  function pad(n) {
    return String(n).padStart(2, '0');
  }

  function formatDate(d) {
    return `${pad(d.getDate())}${pad(d.getMonth() + 1)}${d.getFullYear()}`;
  }

  // Cập nhật giá trị vào ô input có mask của AngularJS
  function setAngularInput(el, val) {
    if (!el) return;
    el.focus();
    el.value = val;
    el.dispatchEvent(new Event('input', { bubbles: true }));
    el.dispatchEvent(new Event('change', { bubbles: true }));
    el.blur();
  }

  function executeSearch(month, year) {
    const firstDate = new Date(year, month - 1, 1);
    let lastDate;

    // Nếu chọn đúng năm và tháng hiện tại -> ngày cuối là ngày hôm nay
    if (year === currentYear && month === currentMonth) {
      lastDate = now;
    } else {
      lastDate = new Date(year, month, 0);
    }

    const tuNgayRaw = formatDate(firstDate);
    const denNgayRaw = formatDate(lastDate);

    // Tìm ô Từ ngày & Đến ngày theo đúng cấu trúc của trang dịch vụ công
    const tuNgayInput = document.querySelector('#field5, [name="field5"], input[ng-model*="field5"]');
    const denNgayInput = document.querySelector('#field6, [name="field6"], input[ng-model*="field6"]');
    const btnSearch = document.getElementById('btnSearchAdvanced');

    if (!tuNgayInput || !denNgayInput) {
      statusMsg.textContent = 'Lỗi: Không tìm thấy ô nhập Từ ngày / Đến ngày!';
      statusMsg.style.color = '#dc2626';
      return;
    }

    // Điền dữ liệu
    setAngularInput(tuNgayInput, tuNgayRaw);
    setAngularInput(denNgayInput, denNgayRaw);

    statusMsg.textContent = `Đã điền: ${tuNgayRaw} - ${denNgayRaw}`;
    statusMsg.style.color = '#16a34a';

    // Nhấn tìm kiếm sau một nhịp chờ ngắn để Angular đồng bộ model
    setTimeout(() => {
      if (btnSearch) {
        btnSearch.click();
      }
    }, 200);
  }

  function renderGrid() {
    monthsGrid.innerHTML = '';
    const selectedYear = parseInt(yearInput.value, 10);

    for (let m = 1; m <= 12; m++) {
      const btn = document.createElement('button');
      btn.className = 't-btn-month';
      btn.textContent = `T${pad(m)}`;

      // 1. Làm mờ các tháng tương lai nếu năm chọn là năm hiện tại
      if (selectedYear === currentYear && m > currentMonth) {
        btn.disabled = true;
      } else if (selectedYear > currentYear) {
        btn.disabled = true;
      }

      // Đánh dấu tháng hiện tại
      if (selectedYear === currentYear && m === currentMonth) {
        btn.classList.add('active-curr');
      }

      btn.onclick = () => executeSearch(m, selectedYear);
      monthsGrid.appendChild(btn);
    }
  }

  document.getElementById('t-prev-year').onclick = () => {
    yearInput.value = parseInt(yearInput.value, 10) - 1;
    renderGrid();
  };

  document.getElementById('t-next-year').onclick = () => {
    yearInput.value = parseInt(yearInput.value, 10) + 1;
    renderGrid();
  };

  yearInput.oninput = renderGrid;

  renderGrid();
})();
