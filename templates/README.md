# Setu AI Enterprise Email Design System & Templates

This directory (`plans/templates/`) contains the complete system of **16 production email templates**, standalone pre-rendered mock previews (`*-mock.html`), and an interactive showcase workbench (`index.html`) for **Setu AI** (`AI-Enabled Workspace & IDP`).

All templates follow the user-approved **V1 Design System** (crisp 560px card container, brand gradient header accent strip, emerald emblem mark, responsive table structure, Outlook VML fallbacks, and zero hardcoded "Vidhya" references).

---

## 📁 Complete Inventory (16 Templates)

| # | Template Code | Event Type | Category | Production File | Mock Preview File |
|---|:---|:---|:---|:---|:---|
| **01** | `AUTH_EMAIL_VERIFICATION_REQUESTED` | `EMAIL_VERIFICATION_REQUESTED` | Onboarding | [`01-welcome-email-verification.html`](01-welcome-email-verification.html) | [`01-welcome-email-verification-mock.html`](01-welcome-email-verification-mock.html) |
| **02** | `AUTH_EMPLOYEE_ACTIVATION` | `EMPLOYEE_ACCOUNT_ACTIVATION` | Onboarding | [`02-employee-account-activation.html`](02-employee-account-activation.html) | [`02-employee-account-activation-mock.html`](02-employee-account-activation-mock.html) |
| **03** | `AUTH_PASSWORD_RESET_REQUESTED` | `PASSWORD_RESET_REQUESTED` | Security | [`03-password-reset-requested.html`](03-password-reset-requested.html) | [`03-password-reset-requested-mock.html`](03-password-reset-requested-mock.html) |
| **04** | `AUTH_PASSWORD_RESET_COMPLETED` | `PASSWORD_RESET_COMPLETED` | Security | [`04-password-reset-completed.html`](04-password-reset-completed.html) | [`04-password-reset-completed-mock.html`](04-password-reset-completed-mock.html) |
| **05** | `AUTH_NEW_DEVICE_LOGIN` | `AUTH_NEW_DEVICE_LOGIN` | Security Alert | [`05-security-new-device-login.html`](05-security-new-device-login.html) | [`05-security-new-device-login-mock.html`](05-security-new-device-login-mock.html) |
| **06** | `AUTH_OTP_REQUESTED_EMAIL` | `OTP_REQUESTED` | MFA / Code | [`06-otp-verification-code.html`](06-otp-verification-code.html) | [`06-otp-verification-code-mock.html`](06-otp-verification-code-mock.html) |
| **07** | `GOV_APPROVAL_STEP_ASSIGNED` | `GOVERNANCE_APPROVAL_STEP_ASSIGNED` | Governance | [`07-governance-approval-request.html`](07-governance-approval-request.html) | [`07-governance-approval-request-mock.html`](07-governance-approval-request-mock.html) |
| **08** | `GOV_ACCESS_APPROVED` | `GOVERNANCE_ACCESS_APPROVED` | Governance | [`08-governance-access-approved.html`](08-governance-access-approved.html) | [`08-governance-access-approved-mock.html`](08-governance-access-approved-mock.html) |
| **09** | `GOV_ACCESS_REJECTED` | `GOVERNANCE_ACCESS_REJECTED` | Governance | [`09-governance-access-rejected.html`](09-governance-access-rejected.html) | [`09-governance-access-rejected-mock.html`](09-governance-access-rejected-mock.html) |
| **10** | `GOV_REVIEW_ASSIGNED` | `GOVERNANCE_ACCESS_REVIEW_ASSIGNED` | Governance | [`10-governance-review-assigned.html`](10-governance-review-assigned.html) | [`10-governance-review-assigned-mock.html`](10-governance-review-assigned-mock.html) |
| **11** | `GOV_REVIEW_REMINDER` | `GOVERNANCE_ACCESS_REVIEW_REMINDER` | Governance | [`11-governance-review-reminder.html`](11-governance-review-reminder.html) | [`11-governance-review-reminder-mock.html`](11-governance-review-reminder-mock.html) |
| **12** | `GOV_REVIEW_ESCALATED` | `GOVERNANCE_ACCESS_REVIEW_ESCALATED` | Governance | [`12-governance-review-escalated.html`](12-governance-review-escalated.html) | [`12-governance-review-escalated-mock.html`](12-governance-review-escalated-mock.html) |
| **13** | `NOTIFICATION_DIGEST` | `NOTIFICATION_DIGEST` | Digests | [`13-notification-digest.html`](13-notification-digest.html) | [`13-notification-digest-mock.html`](13-notification-digest-mock.html) |
| **14** | `AUTH_EMAIL_VERIFICATION_RESENT` | `EMAIL_VERIFICATION_RESENT` | Onboarding | [`14-auth-email-verification-resent.html`](14-auth-email-verification-resent.html) | [`14-auth-email-verification-resent-mock.html`](14-auth-email-verification-resent-mock.html) |
| **15** | `AUTH_EMAIL_VERIFIED` | `EMAIL_VERIFIED` | Onboarding | [`15-auth-email-verified.html`](15-auth-email-verified.html) | [`15-auth-email-verified-mock.html`](15-auth-email-verified-mock.html) |
| **16** | `AUTH_OTP_RESENT_EMAIL` | `OTP_RESENT` | MFA / Code | [`16-otp-resent-verification-code.html`](16-otp-resent-verification-code.html) | [`16-otp-resent-verification-code-mock.html`](16-otp-resent-verification-code-mock.html) |

---

## 🎨 Setu AI Design System Tokens

- **Brand Identity**: Circular gradient emblem (`#059669` to `#047857`) with white letter `S`.
- **Wordmark**: `Setu AI` with subtitle `AI-Enabled Workspace` / `Identity Governance`.
- **Canvas & Card**: Canvas `#f8fafc`, container card `#ffffff`, max-width `560px`, `border-radius: 16px`, `border: 1px solid #e2e8f0`.
- **Typography**: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`.
- **Color Coding by Intent**:
  - **Emerald Success & Standard**: Top accent `#059669` ➔ `#10b981`. Buttons `#059669`. Badges `#ecfdf5` text `#065f46`.
  - **Amber Notice & Reminders**: Top accent `#d97706` ➔ `#f59e0b`. Badges `#fffbeb` text `#b45309`.
  - **Rose / Red Alerts & Escalations**: Top accent `#e11d48` ➔ `#f43f5e`. Badges `#fef2f2` text `#991b1b`.
  - **Slate / Indigo Certification**: Top accent `#4f46e5` ➔ `#6366f1`. Badges `#eff6ff` text `#1d4ed8`.
- **Email Compatibility**:
  - Strict table-based layout compatible with Outlook 2016-2021, Apple Mail, Gmail, Android, and iOS.
  - Microsoft Office VML (`<v:roundrect>`) for curved CTA buttons on Outlook desktop.
  - Safe CSS media queries with `.email-container` and `.btn-cta` fluid sizing on screens < 620px.

---

## 🖥️ How to Preview

1. **Interactive Workbench**:
   Open [`plans/templates/index.html`](file:///c:/Users/yntec/Music/Chat/plans/templates/index.html) in any browser to select any of the 16 templates from the top dropdown, toggle between **Desktop (560px)** and **Mobile (375px)** views, inspect the raw HTML source, and view the variable placeholders schema.
2. **Direct Mock Files**:
   Each template has a corresponding `*-mock.html` file that opens immediately in any web browser without needing any backend or server.
