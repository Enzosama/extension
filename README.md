

## 1. Đăng ký / Cấu hình SSO cho OpenAI

Để cài đặt SSO, bạn cần đáp ứng các điều kiện tiên quyết sau:

### Điều kiện tiên quyết (Prerequisites)

1. **Có tài khoản OpenAI** với quyền truy cập **Global Admin Console**
   - Đây là phần bắt buộc — bạn phải có plan OpenAI (Team/Enterprise) hỗ trợ Admin Console

2. **Là Global Admin**
   - Chỉ Global Admin mới có quyền cấu hình SSO

> **Lưu ý:** Trước khi tiến hành, hãy đọc qua tài liệu **SSO Overview** và **User Management** trên trang OpenAI để nắm rõ kiến trúc SSO trước khi cấu hình.

### Các bước thực hiện

1. Đăng nhập vào **OpenAI Admin Console** tại [https://platform.openai.com/settings/organization/general](https://platform.openai.com/settings/organization/general)
2. Vào mục **SSO, SCIM, and User Management**
3. Chọn **Configuring SSO**
4. Làm theo hướng dẫn trên màn hình để cấu hình Identity Provider (IdP) của bạn (Okta, Azure AD, Google Workspace, v.v.)
5. Sau khi cấu hình xong, kiểm tra lại quyền truy cập của người dùng

---

## 2. Bật F12 — Lấy gói 13 Credits

Mở trang **Checkout/Payment** của OpenAI, sau đó mở **Developer Tools** (nhấn `F12` hoặc `Ctrl+Shift+I`) → chọn tab **Console** → dán toàn bộ đoạn code dưới đây vào và nhấn **Enter**.

```javascript
(async () => {
  try {
    // 1. 获取 accessToken
    const sessionRes = await fetch("/api/auth/session", {
      credentials: "include"
    });

    if (!sessionRes.ok) {
      throw new Error(`获取 session 失败 (HTTP ${sessionRes.status})`);
    }

    const sessionData = await sessionRes.json();
    const accessToken = sessionData?.accessToken;

    if (!accessToken) {
      throw new Error("未找到 accessToken，请确认已登录");
    }

    // 2. 从 URL 获取 checkout_session_id
    const checkoutSessionId = window.location.pathname
      .split("/")
      .pop()
      .split("?")[0];

    if (!checkoutSessionId) {
      throw new Error("未找到 checkout_session_id");
    }

    // 3. 发起 update 请求
    const res = await fetch("https://chatgpt.com/backend-api/payments/checkout/update", {
      method: "POST",
      credentials: "include",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${accessToken}`,
        "oai-device-id": localStorage.getItem("oai-device-id") || "",
        "oai-language": "zh-CN",
        Referer: window.location.href
      },
      body: JSON.stringify({
        checkout_session_id: checkoutSessionId,
        processor_entity: "openai_llc",
        credit_purchase_quantity: 13
      })
    });

    if (!res.ok) {
      const text = await res.text().catch(() => "");
      throw new Error(`请求失败 (HTTP ${res.status}): ${text.slice(0, 120)}`);
    }

    const data = await res.json();
    console.log("成功:", data);

    // 4. 执行完成，刷新页面
    window.location.reload();
  }
  } catch (err) {
    console.error("失败:", err?.message || err);
  }
})();
```

---

## 3. Các repo liên quan

| Repo | Mô tả |
|------|-------|
| [AutoTeam](https://github.com/cnitlrt/AutoTeam) | Tự động đăng ký & xoay vòng tài khoản ChatGPT Team theo quota, đồng bộ sang CPA/Sub2API. |
| [cpa-plugin-codex-invite](https://github.com/LTbinglingfeng/cpa-plugin-codex-invite) | Plugin CPA gửi email mời referral Codex qua Management API, build bằng Go. |
| [cloudflare-openai-oidc-sso](https://github.com/catoncat/cloudflare-openai-oidc-sso) | OIDC Provider nhẹ trên Cloudflare Workers cho Custom OIDC SSO của OpenAI Enterprise. |
| [OC-Codex](https://github.com/angusdevgo/OC-Codex) | GUI trên Windows chuyển đổi giữa Codex OAuth mode và CPAMC/API mode mà không mất lịch sử chat. |
| [kyl-sub2api-extension](https://github.com/hearthealt/kyl-sub2api-extension) | Chrome Extension tự động hóa flow Kyl Invite → OAuth → Sub2API (batch, Manifest V3). |
| [GPTSession2CPAandSub2API](https://github.com/gtxx3600/GPTSession2CPAandSub2API) | Công cụ chuyển đổi session ChatGPT Web sang nhiều định dạng auth (CPA, Sub2API, 9router, Codex...), chạy thuần frontend. |
