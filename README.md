<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>택시모아 ESCROW - 개인택시 양도양수 에스크로 플랫폼</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    :root {
      --primary: #2563eb;
      --primary-dark: #1d4ed8;
      --bg: #f8fafc;
      --card: #ffffff;
      --text: #0f172a;
      --muted: #64748b;
      --border: #e2e8f0;
      --accent: #10b981;
      --danger: #ef4444;
      --warning: #f59e0b;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Apple SD Gothic Neo", sans-serif; }
    body { background-color: var(--bg); color: var(--text); line-height: 1.5; padding-bottom: 60px; }

    /* 헤더 */
    header { background: var(--card); border-bottom: 1px solid var(--border); padding: 12px 24px; display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; z-index: 50; }
    .logo { font-size: 20px; font-weight: 800; color: var(--primary); cursor: pointer; }
    .nav-user-area { display: flex; align-items: center; gap: 12px; }
    .user-badge { font-size: 13px; font-weight: 600; padding: 4px 10px; border-radius: 20px; }
    .user-badge.buyer { background: #dcfce7; color: #15803d; }
    .user-badge.seller { background: #ffedd5; color: #c2410c; }
    .user-badge.admin { background: #fee2e2; color: #b91c1c; font-weight: 800; }
    .user-badge.guest { background: #e2e8f0; color: #475569; }

    /* 알림 센터 드롭다운 */
    .noti-wrapper { position: relative; }
    .noti-btn { background: none; border: 1px solid var(--border); border-radius: 50%; width: 36px; height: 36px; font-size: 16px; cursor: pointer; position: relative; display: flex; align-items: center; justify-content: center; }
    .noti-badge { position: absolute; top: -4px; right: -4px; background: var(--danger); color: #fff; font-size: 10px; font-weight: 800; width: 18px; height: 18px; border-radius: 50%; display: none; align-items: center; justify-content: center; border: 2px solid #fff; }
    .noti-dropdown { display: none; position: absolute; right: 0; top: 44px; width: 330px; background: #fff; border: 1px solid var(--border); border-radius: 10px; box-shadow: 0 10px 25px rgba(0,0,0,0.15); z-index: 100; overflow: hidden; }
    .noti-dropdown.active { display: block; }
    .noti-header { padding: 10px 14px; font-weight: 700; font-size: 13px; border-bottom: 1px solid var(--border); background: #fafafa; display: flex; justify-content: space-between; align-items: center; }
    .noti-list { max-height: 280px; overflow-y: auto; }
    .noti-item { padding: 10px 14px; border-bottom: 1px solid var(--border); font-size: 12px; }
    .noti-item:hover { background: #f8fafc; }
    .noti-time { font-size: 10px; color: var(--muted); margin-top: 3px; }

    /* 공지사항 아코디언 */
    .notice-section { background: #fff; border: 1px solid var(--border); border-radius: 12px; margin: 16px auto 0; max-width: 1200px; overflow: hidden; }
    .notice-header-bar { padding: 12px 18px; background: #eff6ff; border-bottom: 1px solid #dbeafe; display: flex; justify-content: space-between; align-items: center; }
    .notice-item { border-bottom: 1px solid var(--border); }
    .notice-item:last-child { border-bottom: none; }
    .notice-title-row { padding: 12px 18px; display: flex; justify-content: space-between; align-items: center; cursor: pointer; font-size: 13px; font-weight: 600; }
    .notice-title-row:hover { background: #f8fafc; }
    .notice-arrow { font-size: 11px; color: var(--muted); transition: transform 0.2s ease; }
    .notice-content { display: none; padding: 14px 18px; background: #fafafa; font-size: 12px; color: #475569; line-height: 1.6; border-top: 1px dashed var(--border); }
    .notice-item.open .notice-content { display: block; }
    .notice-item.open .notice-arrow { transform: rotate(180deg); }

    /* 네비게이션 탭 */
    .nav-tabs { display: flex; align-items: center; gap: 8px; max-width: 1200px; margin: 16px auto 0; padding: 0 16px; flex-wrap: wrap; }
    .tab-btn { padding: 8px 16px; border-radius: 8px; font-size: 13px; font-weight: 700; border: 1px solid var(--border); background: #fff; cursor: pointer; }
    .tab-btn.active { background: var(--primary); color: #fff; border-color: var(--primary); }

    .container { max-width: 1200px; margin: 16px auto; padding: 0 16px; }
    .card { background: var(--card); border: 1px solid var(--border); border-radius: 12px; padding: 20px; margin-bottom: 20px; }
    .card-title { font-size: 16px; font-weight: 700; margin-bottom: 14px; display: flex; justify-content: space-between; align-items: center; }

    /* 버튼 공통 */
    .btn { padding: 8px 14px; border-radius: 6px; font-size: 13px; font-weight: 600; cursor: pointer; border: none; }
    .btn-primary { background: var(--primary); color: #fff; }
    .btn-primary:hover { background: var(--primary-dark); }
    .btn-outline { background: #fff; border: 1px solid var(--border); color: var(--text); }
    .btn-danger { background: #fee2e2; color: #b91c1c; border: 1px solid #fca5a5; }
    .btn-danger:hover { background: #fecaca; }
    .btn-sm { padding: 5px 9px; font-size: 12px; }

    /* 매물 카드 */
    .listing-card { border: 1px solid var(--border); border-radius: 10px; padding: 16px; margin-bottom: 14px; background: #fff; }
    .listing-top { display: flex; justify-content: space-between; align-items: flex-start; }
    .price { font-size: 20px; font-weight: 800; margin: 4px 0; }
    .tag { font-size: 11px; font-weight: 600; padding: 2px 8px; border-radius: 4px; background: #eff6ff; color: var(--primary); border: 1px solid #bfdbfe; }
    .status-badge { font-size: 12px; font-weight: 700; padding: 3px 8px; border-radius: 6px; background: #ecfdf5; color: #047857; }

    /* 채팅 인박스 */
    .chat-inbox-grid { display: grid; grid-template-columns: 320px 1fr; border: 1px solid var(--border); border-radius: 12px; height: 580px; overflow: hidden; background: #fff; }
    @media (max-width: 768px) { .chat-inbox-grid { grid-template-columns: 1fr; } }
    .chat-room-list { border-right: 1px solid var(--border); overflow-y: auto; background: #fafafa; }
    .chat-room-item { padding: 14px 16px; border-bottom: 1px solid var(--border); cursor: pointer; transition: background 0.15s; }
    .chat-room-item:hover, .chat-room-item.active { background: #eff6ff; }
    .chat-room-item .title { font-size: 13px; font-weight: 700; }
    .chat-room-item .last-msg { font-size: 12px; color: var(--muted); margin-top: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

    .chat-room-view { display: flex; flex-direction: column; height: 100%; background: #fff; }
    .chat-room-header { padding: 12px 16px; border-bottom: 1px solid var(--border); font-weight: 700; font-size: 14px; background: #fff; display: flex; justify-content: space-between; align-items: center; }
    .chat-messages { flex: 1; padding: 16px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; background: #f1f5f9; }
    .msg-row { display: flex; flex-direction: column; width: 100%; }
    .msg-row.me { align-items: flex-end; }
    .msg-row.partner { align-items: flex-start; }
    .sender-name { font-size: 11px; color: var(--muted); margin-bottom: 2px; padding: 0 4px; }
    .bubble { max-width: 75%; padding: 9px 13px; border-radius: 14px; font-size: 13px; line-height: 1.4; word-break: break-all; }
    .bubble.my-bubble { background-color: var(--primary); color: #fff; border-bottom-right-radius: 2px; }
    .bubble.partner-bubble { background-color: #fff; color: var(--text); border: 1px solid var(--border); border-bottom-left-radius: 2px; }
    .chat-input-bar { border-top: 1px solid var(--border); padding: 10px; display: flex; gap: 6px; background: #fff; }
    .chat-input-bar input { flex: 1; border: 1px solid var(--border); border-radius: 6px; padding: 8px 10px; font-size: 13px; outline: none; }

    /* 모달 */
    .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 100; justify-content: center; align-items: center; }
    .modal-overlay.active { display: flex; }
    .modal { background: #fff; width: 92%; max-width: 520px; border-radius: 12px; padding: 24px; max-height: 90vh; overflow-y: auto; }
    .form-group { margin-bottom: 12px; }
    .form-group label { display: block; font-size: 12px; color: var(--muted); margin-bottom: 4px; font-weight: 700; }
    .form-group input, .form-group select, .form-group textarea { width: 100%; padding: 9px 12px; border: 1px solid var(--border); border-radius: 6px; font-size: 13px; outline: none; }
    .input-with-btn { display: flex; gap: 6px; }
    .input-with-btn input { flex: 1; }

    /* 에스크로 */
    .escrow-step-bar { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; margin: 14px 0; }
    .escrow-step-chip { text-align: center; padding: 8px 4px; border: 1px solid var(--border); border-radius: 6px; font-size: 12px; }
    .escrow-step-chip.active { background: #eff6ff; border-color: var(--primary); color: var(--primary); font-weight: 700; }
    .escrow-step-chip.done { background: #ecfdf5; border-color: var(--accent); color: var(--accent); font-weight: 700; }
    .escrow-panel { background: #f8fafc; border: 1px solid var(--border); border-radius: 8px; padding: 14px; margin-bottom: 12px; font-size: 13px; }
    .calc-row { display: flex; justify-content: space-between; margin-bottom: 6px; }
    .calc-row.total { font-weight: 800; border-top: 1px solid var(--border); padding-top: 6px; margin-top: 6px; }
  </style>
</head>
<body>

  <!-- 헤더 -->
  <header>
    <div class="logo" onclick="switchView('listings')">택시모아 ESCROW</div>
    <div class="nav-user-area">
      <!-- 거래 알림 센터 (종 아이콘) -->
      <div class="noti-wrapper">
        <button class="noti-btn" onclick="toggleNotiDropdown()" title="거래 알림">
          🔔
          <span class="noti-badge" id="notiBadge">0</span>
        </button>
        <div class="noti-dropdown" id="notiDropdown">
          <div class="noti-header">
            <span>실시간 거래 알림</span>
            <span style="font-size: 11px; color: var(--primary); cursor: pointer;" onclick="clearNotifications()">모두 읽음</span>
          </div>
          <div class="noti-list" id="notiList"></div>
        </div>
      </div>

      <span class="user-badge guest" id="headerUserBadge">로그인 필요</span>
      <div id="headerAuthArea">
        <button class="btn btn-primary btn-sm" onclick="openAuthModal('login')">로그인 / 회원가입</button>
      </div>
    </div>
  </header>

  <!-- 관리자/공식 아코디언 공지사항 -->
  <div class="notice-section">
    <div class="notice-header-bar">
      <span style="font-weight: 700; font-size: 13px; color: var(--primary);">📢 플랫폼 공식 공지사항 (클릭하여 열람)</span>
      <button class="btn btn-primary btn-sm" id="btnAdminNotice" style="display:none;" onclick="openNoticeModal('create')">+ 공지사항 작성 (관리자)</button>
    </div>
    <div id="accordionNoticeList">
      <div style="padding: 14px 18px; font-size: 12px; color: var(--muted);">공지사항을 불러오는 중...</div>
    </div>
  </div>

  <!-- 상단 네비게이션 탭 -->
  <div class="nav-tabs" id="navTabsContainer"></div>

  <!-- 메인 뷰 -->
  <div class="container">
    <!-- 1. 매물 탐색 뷰 -->
    <div id="viewListings">
      <div class="card">
        <div class="card-title">
          <span id="viewTitle">인증된 개인택시 실거래 매물</span>
          <span style="font-size: 11px; color: var(--accent);">● 실시간 통신 연결됨</span>
        </div>
        <div id="listingList"></div>
      </div>
    </div>

    <!-- 2. 1:1 대화 목록 뷰 -->
    <div id="viewChat" style="display: none;">
      <div class="card" style="padding: 0; overflow: hidden;">
        <div class="chat-inbox-grid">
          <div class="chat-room-list" id="chatRoomListContainer">
            <div style="padding: 20px; text-align: center; color: var(--muted); font-size: 12px;">참여 중인 대화방이 없습니다.</div>
          </div>
          <div class="chat-room-view">
            <div class="chat-room-header">
              <span id="activeRoomHeaderTitle">대화방을 선택해 주세요</span>
            </div>
            <div class="chat-messages" id="chatMessageList">
              <div style="text-align: center; padding: 60px 0; color: var(--muted); font-size: 13px;">
                왼쪽 목록에서 대화방을 선택하거나,<br>매물에서 <strong>[1:1 문의 채팅]</strong>을 눌러 대화를 시작하세요.
              </div>
            </div>
            <div class="chat-input-bar">
              <input type="text" id="chatInputBox" placeholder="1:1 메시지를 입력하세요..." onkeypress="if(event.key==='Enter') sendChatMessage()">
              <button class="btn btn-primary btn-sm" onclick="sendChatMessage()">전송</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 공지사항 등록 및 수정 모달 (관리자 전용) -->
  <div class="modal-overlay" id="noticeModal">
    <div class="modal">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
        <h3 style="font-size:17px; font-weight:800;" id="noticeModalTitle">[관리자] 공지사항 관리</h3>
        <button class="btn btn-outline btn-sm" onclick="closeModal('noticeModal')">닫기</button>
      </div>
      <form onsubmit="handleSaveNotice(event)">
        <input type="hidden" id="noticeEditId">
        <div class="form-group">
          <label>공지사항 제목</label>
          <input type="text" id="noticeFormTitle" placeholder="공지 제목 입력" required>
        </div>
        <div class="form-group">
          <label>공지 상세 내용</label>
          <textarea id="noticeFormContent" style="height: 130px;" placeholder="공지 내용을 입력하세요." required></textarea>
        </div>
        <button type="submit" class="btn btn-primary" style="width:100%; padding:10px;" id="btnNoticeSubmit">공지 등록</button>
      </form>
    </div>
  </div>

  <!-- 판매글 등록 및 수정 모달 (판매자/관리자) -->
  <div class="modal-overlay" id="saleModal">
    <div class="modal">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
        <h3 style="font-size:17px; font-weight:800;" id="saleModalTitle">면허 판매글 등록</h3>
        <button class="btn btn-outline btn-sm" onclick="closeModal('saleModal')">닫기</button>
      </div>
      <form onsubmit="handleSaveListing(event)">
        <input type="hidden" id="saleEditId">
        <div class="form-group">
          <label>관할 지자체</label>
          <select id="saleRegion"><option>서울 관할</option><option>경기 수원</option><option>경기 성남</option><option>인천 관할</option><option>부산 관할</option></select>
        </div>
        <div class="form-group">
          <label>양도 희망금액 (만 원 단위)</label>
          <input type="number" id="salePrice" placeholder="예: 11500" required>
        </div>
        <div class="form-group">
          <label>매물 상세 설명</label>
          <textarea id="saleDesc" placeholder="30년 무사고 은퇴, 구청 서류 완비" required></textarea>
        </div>
        <button type="submit" class="btn btn-primary" style="width:100%; padding:10px;" id="btnSaleSubmit">실시간 등록</button>
      </form>
    </div>
  </div>

  <!-- 에스크로 3단계 안전결제 모달 (구매자 결제 & 단계 진행 완전 복원) -->
  <div class="modal-overlay" id="escrowModal">
    <div class="modal">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
        <h3 style="font-size:17px; font-weight:800;">에스크로 3단계 안심결제</h3>
        <button class="btn btn-outline btn-sm" onclick="closeModal('escrowModal')">닫기</button>
      </div>
      <p style="font-size:12px; color:var(--muted); margin-bottom:10px;" id="esTargetTitle">-</p>
      <div class="escrow-step-bar">
        <div class="escrow-step-chip" id="chip1">1단계: 가계약금</div>
        <div class="escrow-step-chip" id="chip2">2단계: 서류/중도금</div>
        <div class="escrow-step-chip" id="chip3">3단계: 잔금정산</div>
      </div>
      <div id="escrowBodyArea"></div>
    </div>
  </div>

  <!-- 회원가입/로그인 모달 -->
  <div class="modal-overlay" id="authModal">
    <div class="modal">
      <div style="display:flex; border-bottom:2px solid var(--border); margin-bottom:16px;">
        <div id="tabLogin" style="flex:1; padding:8px; text-align:center; font-weight:700; cursor:pointer; color:var(--primary);" onclick="switchAuthTab('login')">로그인</div>
        <div id="tabReg" style="flex:1; padding:8px; text-align:center; font-weight:700; cursor:pointer; color:var(--muted);" onclick="switchAuthTab('reg')">회원가입</div>
      </div>

      <!-- 로그인 -->
      <form id="loginForm" onsubmit="handleLogin(event)">
        <div class="form-group">
          <label>아이디</label>
          <input type="text" id="loginId" placeholder="아이디 입력 (관리자: admin1234)" required>
        </div>
        <div class="form-group">
          <label>비밀번호</label>
          <input type="password" id="loginPw" placeholder="비밀번호 입력 (관리자: admin9999)" required>
        </div>
        <button type="submit" class="btn btn-primary" style="width:100%; padding:10px;">로그인</button>
      </form>

      <!-- 회원가입 -->
      <form id="regForm" style="display:none;" onsubmit="handleRegister(event)">
        <div class="form-group">
          <label>회원 구분 (필수)</label>
          <select id="regRole">
            <option value="양수인">양수인 (개인택시 면허 구매 희망자)</option>
            <option value="양도인">양도인 (개인택시 은퇴/판매자)</option>
          </select>
        </div>
        <div class="form-group">
          <label>사용할 아이디</label>
          <div class="input-with-btn">
            <input type="text" id="regId" placeholder="영문/숫자 4자 이상" required>
            <button type="button" class="btn btn-outline btn-sm" onclick="checkDuplicateId()">중복 확인</button>
          </div>
          <span id="idCheckStatus" style="font-size: 11px; margin-top: 2px; display: block;"></span>
        </div>
        <div class="form-group">
          <label>비밀번호</label>
          <input type="password" id="regPw" placeholder="비밀번호 입력" required>
        </div>
        <div class="form-group">
          <label>성함 (실명)</label>
          <input type="text" id="regName" placeholder="예: 홍길동" required>
        </div>
        <div class="form-group">
          <label>휴대폰 번호 (본인인증)</label>
          <div class="input-with-btn">
            <input type="tel" id="regPhone" placeholder="01012345678" required>
            <button type="button" class="btn btn-outline btn-sm" onclick="sendAuthSms()">인증번호 발송</button>
          </div>
        </div>
        <div class="form-group">
          <label>SMS 인증번호</label>
          <div class="input-with-btn">
            <input type="text" id="regCode" placeholder="4자리 숫자" maxlength="4">
            <button type="button" class="btn btn-outline btn-sm" onclick="verifyAuthSms()">인증 확인</button>
          </div>
          <span id="smsCheckStatus" style="font-size: 11px; margin-top: 2px; display: block;"></span>
        </div>
        <button type="submit" class="btn btn-primary" style="width:100%; padding:10px; margin-top:6px;">회원가입 완료</button>
      </form>
    </div>
  </div>

  <script>
    /* =========================================================
       1. Supabase 클라이언트 & 전역 상태
    ========================================================= */
    const SUPABASE_URL = "https://fjkpahvmezmofxnlpnvl.supabase.co";
    const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImZqa3BhaHZtZXptb2Z4bmxwbnZsIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODk2MDkxMzcsImV4cCI6MjEwNTE4NTEzN30.fWl2sgCu07acPGW7bEG5JVxTDMDI6Xc5FjwYuXB9Xw4";
    const supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    let currentUser = JSON.parse(localStorage.getItem('taximoa_user_v6') || 'null');
    let currentListings = [];
    let currentNotices = [];
    let currentView = 'listings';
    let currentEscrowItem = null;

    let activeRoomId = null;
    let activeListingTitle = '';
    let isIdChecked = false;
    let isSmsPassed = false;
    let mockSmsCode = null;

    window.addEventListener('DOMContentLoaded', () => {
      updateLayout();
      fetchListings();
      fetchNotices();
      fetchMyNotifications();
      subscribeRealtime();
    });

    function openModal(id) { document.getElementById(id).classList.add('active'); }
    function closeModal(id) { document.getElementById(id).classList.remove('active'); }

    /* =========================================================
       2. 판매자 실시간 알림 시스템 (DB 기반)
    ========================================================= */
    function toggleNotiDropdown() {
      document.getElementById('notiDropdown').classList.toggle('active');
    }

    async function sendNotificationToUser(receiverId, message) {
      if (!receiverId) return;
      await supabaseClient.from('notifications').insert([{
        receiver_id: receiverId,
        sender_name: currentUser ? currentUser.name : '시스템',
        message: message,
        is_read: false
      }]);
    }

    async function fetchMyNotifications() {
      if (!currentUser) return;
      const { data } = await supabaseClient
        .from('notifications')
        .select('*')
        .eq('receiver_id', currentUser.id)
        .order('created_at', { ascending: false });

      const container = document.getElementById('notiList');
      const badge = document.getElementById('notiBadge');
      container.innerHTML = '';

      if (!data || data.length === 0) {
        badge.style.display = 'none';
        container.innerHTML = `<div style="padding:20px; text-align:center; color:var(--muted); font-size:12px;">새로운 거래 알림이 없습니다.</div>`;
        return;
      }

      const unreadCount = data.filter(n => !n.is_read).length;
      badge.textContent = unreadCount;
      badge.style.display = unreadCount > 0 ? 'flex' : 'none';

      data.forEach(n => {
        const item = document.createElement('div');
        item.className = 'noti-item';
        item.style.fontWeight = n.is_read ? 'normal' : '700';
        item.innerHTML = `<div>${n.message}</div><div class="noti-time">${new Date(n.created_at).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</div>`;
        container.appendChild(item);
      });
    }

    async function clearNotifications() {
      if (!currentUser) return;
      await supabaseClient.from('notifications').update({ is_read: true }).eq('receiver_id', currentUser.id);
      fetchMyNotifications();
    }

    /* =========================================================
       3. 공지사항 아코디언 & [관리자 수정/삭제]
    ========================================================= */
    async function fetchNotices() {
      const { data } = await supabaseClient.from('notices').select('*').order('created_at', { ascending: false });
      currentNotices = data || [];
      renderNotices();
    }

    function renderNotices() {
      const container = document.getElementById('accordionNoticeList');
      container.innerHTML = '';

      if (currentNotices.length === 0) {
        container.innerHTML = `<div style="padding:14px 18px; font-size:12px; color:var(--muted);">등록된 공지사항이 없습니다.</div>`;
        return;
      }

      const isAdmin = currentUser && currentUser.role === '관리자';

      currentNotices.forEach((notice, idx) => {
        const item = document.createElement('div');
        item.className = `notice-item ${idx === 0 ? 'open' : ''}`;

        let adminButtons = '';
        if (isAdmin) {
          adminButtons = `
            <div style="margin-top: 10px; display:flex; gap:6px;">
              <button class="btn btn-outline btn-sm" onclick="openNoticeModal('edit', '${notice.id}')">수정</button>
              <button class="btn btn-danger btn-sm" onclick="deleteNotice('${notice.id}')">삭제</button>
            </div>
          `;
        }

        item.innerHTML = `
          <div class="notice-title-row" onclick="toggleNotice(this)">
            <span>📌 ${notice.title} <small style="color:var(--muted); font-weight:normal; margin-left:6px;">(${notice.author || '운영팀'})</small></span>
            <span class="notice-arrow">▼</span>
          </div>
          <div class="notice-content">
            <div>${(notice.content || '').replace(/\n/g, '<br>')}</div>
            ${adminButtons}
          </div>
        `;
        container.appendChild(item);
      });
    }

    function toggleNotice(el) { el.parentElement.classList.toggle('open'); }

    function openNoticeModal(mode, noticeId) {
      const modalTitle = document.getElementById('noticeModalTitle');
      const formTitle = document.getElementById('noticeFormTitle');
      const formContent = document.getElementById('noticeFormContent');
      const editId = document.getElementById('noticeEditId');
      const submitBtn = document.getElementById('btnNoticeSubmit');

      if (mode === 'edit') {
        const notice = currentNotices.find(n => n.id === noticeId);
        if (!notice) return;
        modalTitle.textContent = '[관리자] 공지사항 수정';
        editId.value = notice.id;
        formTitle.value = notice.title;
        formContent.value = notice.content;
        submitBtn.textContent = '수정사항 저장';
      } else {
        modalTitle.textContent = '[관리자] 신규 공지 등록';
        editId.value = '';
        formTitle.value = '';
        formContent.value = '';
        submitBtn.textContent = '공지 등록';
      }
      openModal('noticeModal');
    }

    async function handleSaveNotice(e) {
      e.preventDefault();
      const editId = document.getElementById('noticeEditId').value;
      const title = document.getElementById('noticeFormTitle').value.trim();
      const content = document.getElementById('noticeFormContent').value.trim();

      if (editId) {
        // 수정
        const { error } = await supabaseClient.from('notices').update({ title, content }).eq('id', editId);
        if (error) { alert('수정 실패: ' + error.message); return; }
        alert('공지사항이 수정되었습니다.');
      } else {
        // 신규 등록
        const { error } = await supabaseClient.from('notices').insert([{ title, content, author: '대표 관리자' }]);
        if (error) { alert('등록 실패: ' + error.message); return; }
        alert('신규 공지가 등록되었습니다.');
      }
      closeModal('noticeModal');
      fetchNotices();
    }

    async function deleteNotice(noticeId) {
      if (!confirm('정말 이 공지사항을 삭제하시겠습니까?')) return;
      const { error } = await supabaseClient.from('notices').delete().eq('id', noticeId);
      if (error) { alert('삭제 실패: ' + error.message); return; }
      alert('공지사항이 삭제되었습니다.');
      fetchNotices();
    }

    /* =========================================================
       4. 매물 렌더링 & [판매자/관리자 매물 수정·삭제 권한]
    ========================================================= */
    async function fetchListings() {
      const { data } = await supabaseClient.from('listings').select('*').order('created_at', { ascending: false });
      currentListings = data || [];
      renderListings();
    }

    function renderListings() {
      const container = document.getElementById('listingList');
      container.innerHTML = '';

      if (currentListings.length === 0) {
        container.innerHTML = `<div style="text-align:center; padding:40px 0; color:var(--muted); font-size:13px;">등록된 매물이 없습니다.</div>`;
        return;
      }

      currentListings.forEach(item => {
        const card = document.createElement('div');
        card.className = 'listing-card';

        const isOwner = currentUser && currentUser.id === item.seller_id;
        const isAdmin = currentUser && currentUser.role === '관리자';

        let actionButtons = '';
        if (!currentUser) {
          actionButtons = `<button class="btn btn-outline btn-sm" onclick="openAuthModal('login')">로그인 후 이용 가능</button>`;
        } else if (currentUser.role === '양도인') {
          if (isOwner) {
            actionButtons = `
              <button class="btn btn-outline btn-sm" onclick="openEscrowProcess('${item.id}', true)">에스크로 진행 현황</button>
              <button class="btn btn-outline btn-sm" onclick="openSaleModal('edit', '${item.id}')">매물 수정</button>
              <button class="btn btn-danger btn-sm" onclick="deleteListing('${item.id}')">매물 삭제</button>
            `;
          } else {
            actionButtons = `<span style="font-size:12px; color:var(--muted);">타 양도인 매물</span>`;
          }
        } else if (currentUser.role === '양수인') {
          actionButtons = `
            <button class="btn btn-primary btn-sm" onclick="startChatFromListing('${item.id}', '${item.seller_name}', '${item.seller_id}', '${item.region} ${(item.price/10000).toLocaleString()}만')">1:1 문의 채팅</button>
            <button class="btn btn-outline btn-sm" onclick="openEscrowProcess('${item.id}', false)">안전 에스크로 진행</button>
          `;
        }

        // 관리자는 어떤 매물이든 강제 삭제 가능
        if (isAdmin) {
          actionButtons += `<button class="btn btn-danger btn-sm" style="margin-left:auto;" onclick="deleteListing('${item.id}')">[관리자] 매물 강제삭제</button>`;
        }

        card.innerHTML = `
          <div class="listing-top">
            <div>
              <span class="status-badge">${item.step === 0 ? '거래 가능' : item.step + '단계 에스크로 진행 중'}</span>
              <span class="tag" style="margin-left:4px;">${item.region}</span>
              <div class="price">${(item.price / 10000).toLocaleString()}만 원</div>
            </div>
            <span style="font-size:12px; color:var(--muted);">양도인: ${item.seller_name} 기사님</span>
          </div>
          <p style="font-size:13px; color:var(--muted); margin:6px 0;">${item.desc_text}</p>
          <div style="margin-top:10px; display:flex; gap:8px; align-items:center;">${actionButtons}</div>
        `;
        container.appendChild(card);
      });
    }

    function openSaleModal(mode, listingId) {
      const modalTitle = document.getElementById('saleModalTitle');
      const region = document.getElementById('saleRegion');
      const price = document.getElementById('salePrice');
      const desc = document.getElementById('saleDesc');
      const editId = document.getElementById('saleEditId');
      const submitBtn = document.getElementById('btnSaleSubmit');

      if (mode === 'edit') {
        const item = currentListings.find(i => i.id === listingId);
        if (!item) return;
        modalTitle.textContent = '내 면허 판매글 수정';
        editId.value = item.id;
        region.value = item.region;
        price.value = item.price / 10000;
        desc.value = item.desc_text;
        submitBtn.textContent = '수정 완료';
      } else {
        modalTitle.textContent = '면허 판매글 등록';
        editId.value = '';
        price.value = '';
        desc.value = '';
        submitBtn.textContent = '실시간 등록';
      }
      openModal('saleModal');
    }

    async function handleSaveListing(e) {
      e.preventDefault();
      const editId = document.getElementById('saleEditId').value;
      const region = document.getElementById('saleRegion').value;
      const price = parseInt(document.getElementById('salePrice').value, 10) * 10000;
      const desc_text = document.getElementById('saleDesc').value.trim();

      if (editId) {
        // 매물 수정
        const { error } = await supabaseClient.from('listings').update({ region, price, desc_text }).eq('id', editId);
        if (error) { alert('수정 실패: ' + error.message); return; }
        alert('매물 정보가 수정되었습니다.');
      } else {
        // 신규 매물 등록
        const { error } = await supabaseClient.from('listings').insert([{
          region, price, desc_text, seller_id: currentUser.id, seller_name: currentUser.name, step: 0
        }]);
        if (error) { alert('등록 실패: ' + error.message); return; }
        alert('새 매물이 등록되었습니다!');
      }
      closeModal('saleModal');
      fetchListings();
    }

    async function deleteListing(listingId) {
      if (!confirm('정말 이 판매글을 삭제하시겠습니까?')) return;
      const { error } = await supabaseClient.from('listings').delete().eq('id', listingId);
      if (error) { alert('삭제 실패: ' + error.message); return; }
      alert('매물이 삭제되었습니다.');
      fetchListings();
    }

    /* =========================================================
       5. [에스크로 3단계 완전 복원] & 판매자 실시간 알림 발송
    ========================================================= */
    function openEscrowProcess(id, isSellerView) {
      currentEscrowItem = currentListings.find(i => i.id === id);
      if (!currentEscrowItem) return;
      renderEscrowBody(isSellerView);
      openModal('escrowModal');
    }

    function renderEscrowBody(isSellerView) {
      const item = currentEscrowItem;
      document.getElementById('esTargetTitle').textContent = `[${item.region}] ${item.desc_text} (${(item.price/10000).toLocaleString()}만 원)`;

      const chip1 = document.getElementById('chip1');
      const chip2 = document.getElementById('chip2');
      const chip3 = document.getElementById('chip3');
      const body = document.getElementById('escrowBodyArea');

      chip1.className = 'escrow-step-chip ' + (item.step >= 1 ? 'done' : (item.step === 0 ? 'active' : ''));
      chip2.className = 'escrow-step-chip ' + (item.step >= 2 ? 'done' : (item.step === 1 ? 'active' : ''));
      chip3.className = 'escrow-step-chip ' + (item.step === 3 ? 'done' : (item.step === 2 ? 'active' : ''));

      const mid = Math.round(item.price * 0.1);
      const bal = item.price - 1000000 - mid;
      const fee = Math.round(item.price * 0.005);

      if (isSellerView) {
        body.innerHTML = `
          <div class="escrow-panel">
            <div style="font-weight:700; color:var(--primary); margin-bottom:6px;">● 현재 에스크로 상태: ${item.step}단계 진행 중</div>
            <div class="calc-row"><span>총 매매대금</span><strong>${item.price.toLocaleString()} 원</strong></div>
            <div class="calc-row"><span>1단계 가계약금 예치 (100만 원)</span><span>${item.step >= 1 ? '✔ 입금완료' : '대기중'}</span></div>
            <div class="calc-row"><span>2단계 중도금/접수증 (10%)</span><span>${item.step >= 2 ? '✔ 승인완료' : '대기중'}</span></div>
            <div class="calc-row"><span>3단계 잔금 및 수수료 정산</span><span>${item.step >= 3 ? '✔ 정산완료' : '대기중'}</span></div>
          </div>
          <p style="font-size:12px; color:var(--muted);">구매자가 각 단계를 결제할 때마다 상단 알림(🔔)으로 실시간 전송됩니다.</p>
        `;
        return;
      }

      // [구매자 전용] 1단계 -> 2단계 -> 3단계 결제 로직
      if (item.step === 0) {
        body.innerHTML = `
          <div class="escrow-panel">
            <div class="calc-row"><span>총 매매대금</span><strong>${item.price.toLocaleString()} 원</strong></div>
            <div class="calc-row" style="color:var(--primary); font-weight:700;"><span>1단계 가계약금 예치</span><span>1,000,000 원</span></div>
            <div class="calc-row"><span>2단계 중도금 (10%)</span><span>${mid.toLocaleString()} 원</span></div>
            <div class="calc-row"><span>3단계 잔금 정산</span><span>${bal.toLocaleString()} 원</span></div>
          </div>
          <button class="btn btn-primary" style="width:100%; padding:10px;" onclick="advanceEscrowStep(1)">1단계 가계약금 100만 원 예치하기</button>
        `;
      } else if (item.step === 1) {
        body.innerHTML = `
          <div class="escrow-panel">
            <div style="color:var(--accent); font-weight:700; margin-bottom:6px;">✔ 1단계 가계약금(100만 원)이 안전하게 예치되었습니다.</div>
            <div class="calc-row"><span>2단계 중도금 (10%)</span><strong>${mid.toLocaleString()} 원</strong></div>
            <div style="margin-top:10px;">
              <label style="font-size:12px; color:var(--muted); font-weight:700;">구청 인가 신청 접수증 첨부</label>
              <input type="file" style="margin-top:4px;">
            </div>
          </div>
          <button class="btn btn-primary" style="width:100%; padding:10px;" onclick="advanceEscrowStep(2)">2단계 중도금 예치 및 접수증 등록</button>
        `;
      } else if (item.step === 2) {
        body.innerHTML = `
          <div class="escrow-panel">
            <div style="color:var(--accent); font-weight:700; margin-bottom:6px;">✔ 구청 심사 및 중도금 예치가 확인되었습니다.</div>
            <div class="calc-row"><span>최종 납부 잔금</span><strong>${bal.toLocaleString()} 원</strong></div>
            <div class="calc-row"><span>에스크로 수수료 (0.5%)</span><span>${fee.toLocaleString()} 원</span></div>
            <div class="calc-row total" style="color:var(--primary);"><span>최종 납부 합계</span><span>${(bal + fee).toLocaleString()} 원</span></div>
          </div>
          <button class="btn btn-primary" style="width:100%; padding:10px;" onclick="advanceEscrowStep(3)">구청 인가 확인 및 최종 잔금 결제/정산</button>
        `;
      } else if (item.step === 3) {
        body.innerHTML = `
          <div style="text-align:center; padding:20px 0;">
            <div style="font-size:36px;">🎉</div>
            <h4 style="color:var(--accent); font-size:16px; font-weight:800; margin-top:6px;">소유권 이전 및 대금 정산 완료!</h4>
            <p style="font-size:12px; color:var(--muted); margin-top:4px;">모든 결제와 지자체 이전 절차가 안전하게 마무리되었습니다.</p>
          </div>
        `;
      }
    }

    async function advanceEscrowStep(nextStep) {
      const { error } = await supabaseClient
        .from('listings')
        .update({ step: nextStep })
        .eq('id', currentEscrowItem.id);

      if (error) { alert('에스크로 진행 오류: ' + error.message); return; }

      // 판매자에게 단계별 알림 발송
      const stepNames = {
        1: '1단계 가계약금(100만 원)을 예치했습니다. 거래 락(Lock)이 체결되었습니다.',
        2: '2단계 중도금(10%) 예치 및 구청 접수증을 등록했습니다.',
        3: '최종 잔금 정산을 완료했습니다. 양도 대금이 정산 계좌로 송금됩니다.'
      };
      await sendNotificationToUser(currentEscrowItem.seller_id, `[에스크로 알림] 구매자(${currentUser.name} 님)가 ${stepNames[nextStep]}`);

      currentEscrowItem.step = nextStep;
      alert(`에스크로 ${nextStep}단계 결제가 완료되었습니다! 판매자에게 실시간 알림이 전송되었습니다.`);
      renderEscrowBody(false);
      fetchListings();
    }

    /* =========================================================
       6. 1:1 채팅 인박스 & 메시징
    ========================================================= */
    function startChatFromListing(listingId, sellerName, sellerId, title) {
      activeRoomId = `room_${listingId}_${sellerId}_${currentUser.id}`;
      activeListingTitle = `${title} (${sellerName} 기사님)`;
      switchView('chat');
      enterChatRoom(activeRoomId, activeListingTitle);
    }

    async function loadMyChatRooms() {
      if (!currentUser) return;
      const { data } = await supabaseClient.from('chat_messages').select('*').order('created_at', { ascending: false });
      const container = document.getElementById('chatRoomListContainer');
      container.innerHTML = '';
      if (!data) return;

      const myRooms = {};
      data.forEach(msg => {
        if (!msg.room_id) return;
        const parts = msg.room_id.split('_');
        const sellerId = parts[2];
        const buyerId = parts[3];

        if (currentUser.id === sellerId || currentUser.id === buyerId || currentUser.role === '관리자') {
          if (!myRooms[msg.room_id]) {
            myRooms[msg.room_id] = { title: msg.listing_title || '1:1 상담방', lastMsg: msg.message };
          }
        }
      });

      const roomKeys = Object.keys(myRooms);
      if (roomKeys.length === 0) {
        container.innerHTML = `<div style="padding: 30px; text-align: center; color: var(--muted); font-size: 12px;">진행 중인 1:1 대화방이 없습니다.</div>`;
        return;
      }

      roomKeys.forEach(rId => {
        const item = document.createElement('div');
        item.className = `chat-room-item ${activeRoomId === rId ? 'active' : ''}`;
        item.onclick = () => enterChatRoom(rId, myRooms[rId].title);
        item.innerHTML = `<div class="title">${myRooms[rId].title}</div><div class="last-msg">${myRooms[rId].lastMsg}</div>`;
        container.appendChild(item);
      });
    }

    function enterChatRoom(roomId, title) {
      activeRoomId = roomId;
      document.getElementById('activeRoomHeaderTitle').textContent = title || '1:1 안심 상담방';
      loadMessages();
      loadMyChatRooms();
    }

    async function loadMessages() {
      if (!activeRoomId) return;
      const { data } = await supabaseClient.from('chat_messages').select('*').eq('room_id', activeRoomId).order('created_at', { ascending: true });
      const container = document.getElementById('chatMessageList');
      container.innerHTML = '';

      if (!data || data.length === 0) {
        container.innerHTML = `<div style="text-align:center; padding:30px; color:var(--muted); font-size:12px;">대화방이 개설되었습니다. 첫 메시지를 남겨보세요!</div>`;
        return;
      }

      data.forEach(m => {
        const isMe = (m.sender_id === currentUser.id);
        const row = document.createElement('div');
        row.className = `msg-row ${isMe ? 'me' : 'partner'}`;
        row.innerHTML = `
          ${!isMe ? `<div class="sender-name">${m.sender_name}</div>` : ''}
          <div class="bubble ${isMe ? 'my-bubble' : 'partner-bubble'}">${m.message}</div>
        `;
        container.appendChild(row);
      });
      container.scrollTop = container.scrollHeight;
    }

    async function sendChatMessage() {
      const input = document.getElementById('chatInputBox');
      const text = input.value.trim();
      if (!text || !activeRoomId) return;

      const { error } = await supabaseClient.from('chat_messages').insert([{
        room_id: activeRoomId,
        sender_id: currentUser.id,
        sender_name: currentUser.name,
        message: text,
        listing_title: activeListingTitle || '1:1 안심 상담'
      }]);

      if (error) { alert('메시지 전송 실패: ' + error.message); return; }

      // 상대방에게 알림 전송
      const parts = activeRoomId.split('_');
      const partnerId = (currentUser.id === parts[2]) ? parts[3] : parts[2];
      await sendNotificationToUser(partnerId, `[새 메시지] ${currentUser.name}: ${text}`);

      input.value = '';
      loadMessages();
    }

    /* =========================================================
       7. 인증 & 세션 제어
    ========================================================= */
    function switchView(view) {
      currentView = view;
      document.getElementById('viewListings').style.display = (view === 'listings') ? 'block' : 'none';
      document.getElementById('viewChat').style.display = (view === 'chat') ? 'block' : 'none';
      if (view === 'chat') loadMyChatRooms();
      updateLayout();
    }

    function openAuthModal(tab = 'login') { openModal('authModal'); switchAuthTab(tab); }
    function switchAuthTab(tab) {
      document.getElementById('tabLogin').style.color = tab === 'login' ? 'var(--primary)' : 'var(--muted)';
      document.getElementById('tabReg').style.color = tab === 'reg' ? 'var(--primary)' : 'var(--muted)';
      document.getElementById('loginForm').style.display = tab === 'login' ? 'block' : 'none';
      document.getElementById('regForm').style.display = tab === 'reg' ? 'block' : 'none';
    }

    function checkDuplicateId() {
      const id = document.getElementById('regId').value.trim();
      const status = document.getElementById('idCheckStatus');
      if (id.length < 4) {
        status.style.color = 'red'; status.textContent = '✕ 4자 이상 입력해 주세요.';
        isIdChecked = false; return;
      }
      if (id === 'admin1234') {
        status.style.color = 'red'; status.textContent = '✕ 예약된 관리자 아이디입니다.';
        isIdChecked = false; return;
      }
      const users = JSON.parse(localStorage.getItem('taximoa_all_users_v6') || '{}');
      if (users[id]) {
        status.style.color = 'red'; status.textContent = '✕ 이미 사용 중인 아이디입니다.';
        isIdChecked = false;
      } else {
        status.style.color = 'green'; status.textContent = '✔ 사용 가능한 아이디입니다.';
        isIdChecked = true;
      }
    }

    function sendAuthSms() {
      const phone = document.getElementById('regPhone').value.trim();
      if (!phone || phone.length < 10) { alert('올바른 휴대폰 번호를 입력하세요.'); return; }
      mockSmsCode = Math.floor(1000 + Math.random() * 9000).toString();
      alert(`[택시모아 본인인증]\n인증번호: [ ${mockSmsCode} ]`);
    }

    function verifyAuthSms() {
      const code = document.getElementById('regCode').value.trim();
      const status = document.getElementById('smsCheckStatus');
      if (code && code === mockSmsCode) {
        isSmsPassed = true; status.style.color = 'green'; status.textContent = '✔ 인증되었습니다.';
      } else {
        isSmsPassed = false; status.style.color = 'red'; status.textContent = '✕ 번호가 일치하지 않습니다.';
      }
    }

    function handleRegister(e) {
      e.preventDefault();
      if (!isIdChecked) { alert('아이디 중복확인을 완료해 주세요.'); return; }
      if (!isSmsPassed) { alert('휴대폰 SMS 인증을 완료해 주세요.'); return; }

      const role = document.getElementById('regRole').value;
      const id = document.getElementById('regId').value.trim();
      const pw = document.getElementById('regPw').value.trim();
      const name = document.getElementById('regName').value.trim();
      const phone = document.getElementById('regPhone').value.trim();

      const users = JSON.parse(localStorage.getItem('taximoa_all_users_v6') || '{}');
      users[id] = { id, pw, name, phone, role };
      localStorage.setItem('taximoa_all_users_v6', JSON.stringify(users));

      currentUser = users[id];
      localStorage.setItem('taximoa_user_v6', JSON.stringify(currentUser));
      alert(`회원가입 완료! ${name} 님 환영합니다.`);
      closeModal('authModal');
      e.target.reset();
      updateLayout();
      fetchMyNotifications();
    }

    function handleLogin(e) {
      e.preventDefault();
      const id = document.getElementById('loginId').value.trim();
      const pw = document.getElementById('loginPw').value.trim();

      if (id === 'admin1234') {
        if (pw === 'admin9999') {
          currentUser = { id: 'admin1234', name: '대표 관리자', role: '관리자' };
        } else {
          alert('관리자 비밀번호가 일치하지 않습니다.'); return;
        }
      } else {
        const users = JSON.parse(localStorage.getItem('taximoa_all_users_v6') || '{}');
        if (!users[id] || users[id].pw !== pw) {
          alert('아이디 또는 비밀번호가 올바르지 않습니다.'); return;
        }
        currentUser = users[id];
      }

      localStorage.setItem('taximoa_user_v6', JSON.stringify(currentUser));
      closeModal('authModal');
      e.target.reset();
      alert(`${currentUser.name} 님 환영합니다!`);
      updateLayout();
      fetchMyNotifications();
    }

    function handleLogout() {
      currentUser = null;
      localStorage.removeItem('taximoa_user_v6');
      activeRoomId = null;
      switchView('listings');
      fetchMyNotifications();
    }

    function updateLayout() {
      const badge = document.getElementById('headerUserBadge');
      const authArea = document.getElementById('headerAuthArea');
      const navTabs = document.getElementById('navTabsContainer');
      const btnAdmin = document.getElementById('btnAdminNotice');

      if (!currentUser) {
        badge.textContent = '로그인 필요'; badge.className = 'user-badge guest';
        authArea.innerHTML = `<button class="btn btn-primary btn-sm" onclick="openAuthModal('login')">로그인 / 회원가입</button>`;
        navTabs.innerHTML = `<button class="tab-btn ${currentView==='listings'?'active':''}" onclick="switchView('listings')">전체 실거래 매물</button>`;
        btnAdmin.style.display = 'none';
      } else {
        badge.textContent = `${currentUser.name} (${currentUser.role})`;
        badge.className = `user-badge ${currentUser.role === '관리자' ? 'admin' : (currentUser.role === '양도인' ? 'seller' : 'buyer')}`;
        authArea.innerHTML = `<button class="btn btn-outline btn-sm" onclick="handleLogout()">로그아웃</button>`;
        btnAdmin.style.display = (currentUser.role === '관리자') ? 'block' : 'none';

        let navHtml = `
          <button class="tab-btn ${currentView==='listings'?'active':''}" onclick="switchView('listings')">전체 실거래 매물</button>
          <button class="tab-btn ${currentView==='chat'?'active':''}" onclick="switchView('chat')">💬 1:1 대화 목록</button>
        `;
        if (currentUser.role === '양도인') {
          navHtml += `<button class="btn btn-primary btn-sm" style="margin-left:auto;" onclick="openSaleModal('create')">+ 판매글 등록</button>`;
        }
        navTabs.innerHTML = navHtml;
      }
      renderNotices();
      renderListings();
    }

    /* =========================================================
       8. Supabase Realtime 리스너
    ========================================================= */
    function subscribeRealtime() {
      supabaseClient.channel('realtime_channel_v6')
        .on('postgres_changes', { event: '*', schema: 'public', table: 'listings' }, fetchListings)
        .on('postgres_changes', { event: '*', schema: 'public', table: 'notices' }, fetchNotices)
        .on('postgres_changes', { event: '*', schema: 'public', table: 'notifications' }, () => {
          fetchMyNotifications();
        })
        .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'chat_messages' }, () => {
          if (activeRoomId) loadMessages();
          if (currentView === 'chat') loadMyChatRooms();
        }).subscribe();
    }
  </script>
</body>
</html>
