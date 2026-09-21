
# wheeljourney — Launcher (SSO com Microsoft Entra External ID)

Este projeto é um **launcher web estático** (HTML único) que demonstra:
- **Onboarding** no aplicativo principal (**PoC-Rodas-App**) com Microsoft Entra External ID.
- **SSO** entre submódulos (**Rodas-App2** e **Rodas-App3**) usando o mesmo *User Flow*.
- Redirecionamento para `jwt.ms` para visualizar o **ID Token**.
- Botão de **logout global** (single sign-out) do tenant/policy.

> **Uso em PoC**: foco em demonstrar jornada de autenticação, SSO e visualização de token.
> Em produção, utilize **Authorization Code + PKCE** e backend para validação segura do token.

---

## 📁 Estrutura

- `index.html` — arquivo único com:
  - UI/UX com identidade visual genérica e ícones SVG.
  - Lógica de construção das URLs `/authorize` e `/logout`.
  - Botões para **Entrar**, **Copiar URL**, **Sair**.

---

## ⚙️ Configuração (no `index.html`)

Edite estes valores na seção de configuração:

```js
const TENANT_HOST   = "irondom33.ciamlogin.com";
const TENANT_DOMAIN = "irondom33.onmicrosoft.com";
const USER_FLOW     = "PocRodarFlow";
const REDIRECT_URI  = "https://jwt.ms";

// Client IDs dos aplicativos associados ao mesmo User Flow:
const CLIENT_ID_APP1 = "f7170ba0-8f71-4a82-b7f3-728f15ffd27b"; // PoC-Rodas-App (Onboarding)
const CLIENT_ID_APP2 = "7265df95-7b7e-47b1-8a0c-907ce989cd21"; // Rodas-App2
const CLIENT_ID_APP3 = "1295c81e-f12
