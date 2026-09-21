# Wheel Journey

Launcher web estático para demonstrar uma jornada de onboarding e Single Sign-On (SSO) com Microsoft Entra External ID.

O portal reúne três aplicações em uma única interface, reaproveitando a sessão do tenant entre os acessos. A experiência usa identidade visual genérica, dark mode permanente e layout responsivo.

## Demonstração

Aplicação publicada no Azure Static Web Apps:

https://happy-meadow-002b01910.3.azurestaticapps.net

## Funcionalidades

- Onboarding inicial pelo App01.
- SSO entre App01, App02 e App03 usando o mesmo User Flow.
- Solicitação de MFA no fluxo do App03, conforme a política configurada no tenant.
- Geração e exibição das URLs de autorização OpenID Connect.
- Cópia individual ou conjunta das URLs de autenticação.
- Logout global da sessão do tenant.
- Redirecionamento do ID Token para `jwt.ms` durante a demonstração.
- Interface responsiva e acessível, com suporte a redução de movimento.

## Jornada de acesso

1. O usuário seleciona **Entrar** no App01 e conclui o onboarding.
2. O Microsoft Entra External ID cria a sessão no domínio `*.ciamlogin.com`.
3. O usuário retorna ao portal e acessa App02 ou App03 na mesma aba.
4. Como os aplicativos compartilham tenant e User Flow, a sessão existente permite SSO.
5. O App03 pode solicitar MFA adicional de acordo com sua política de acesso.

As URLs não incluem `prompt=login`, pois esse parâmetro forçaria uma nova autenticação e impediria o reaproveitamento da sessão.

## Estrutura

```text
.
|-- .github/
|   `-- workflows/
|       `-- azure-static-web-apps-happy-meadow-002b01910.yml
|-- index.html
`-- README.md
```

| Arquivo | Responsabilidade |
| --- | --- |
| `index.html` | Interface, dark mode, configuração do ambiente e lógica OIDC. |
| `.github/workflows/azure-static-web-apps-happy-meadow-002b01910.yml` | Publicação automática no Azure Static Web Apps. |
| `README.md` | Documentação funcional e operacional do projeto. |

O projeto não possui etapa de build nem dependências de runtime. Todo o código necessário está no `index.html`.

## Configuração do Entra External ID

As configurações ficam no bloco JavaScript ao final de `index.html`:

```js
const TENANT_HOST = "irondom33.ciamlogin.com";
const TENANT_DOMAIN = "irondom33.onmicrosoft.com";
const USER_FLOW = "PocRodarFlow";
const REDIRECT_URI = "https://jwt.ms";

const CLIENT_ID_APP1 = "f7170ba0-8f71-4a82-b7f3-728f15ffd27b";
const CLIENT_ID_APP2 = "7265df95-7b7e-47b1-8a0c-907ce989cd21";
const CLIENT_ID_APP3 = "1259c81e-ff2b-46f7-9317-2cf7ef240bf0";
```

Para usar outro ambiente:

1. Substitua o host e o domínio do tenant.
2. Informe o User Flow associado aos três aplicativos.
3. Atualize os Client IDs dos registros de aplicativo.
4. Cadastre o `REDIRECT_URI` em cada registro de aplicativo.
5. Confirme que as políticas de MFA estão associadas aos fluxos esperados.

Client IDs são identificadores públicos e não concedem acesso isoladamente. Não adicione client secrets, tokens ou credenciais ao HTML.

## Protocolo de autenticação

A demonstração monta uma requisição para o endpoint `/oauth2/v2.0/authorize` com os parâmetros:

| Parâmetro | Valor |
| --- | --- |
| `p` | User Flow configurado em `USER_FLOW`. |
| `client_id` | Client ID do aplicativo selecionado. |
| `nonce` | Valor aleatório gerado no navegador. |
| `redirect_uri` | `https://jwt.ms`. |
| `scope` | `openid profile`. |
| `response_type` | `id_token`. |

O logout usa `/oauth2/v2.0/logout`, o mesmo User Flow e `post_logout_redirect_uri`.

## Execução local

Como o projeto é estático, ele pode ser servido por qualquer servidor HTTP. Por exemplo:

```powershell
npx serve .
```

Abra a URL exibida no terminal. O uso de `localhost` também permite que a API de clipboard funcione em navegadores modernos.

Também é possível abrir `index.html` diretamente, mas alguns navegadores restringem clipboard e outros recursos quando a página usa o protocolo `file://`.

## Publicação

O workflow de GitHub Actions publica automaticamente no Azure Static Web Apps quando há push no branch `main`.

Configuração atual:

| Propriedade | Valor |
| --- | --- |
| Branch de produção | `main` |
| Origem da aplicação | `/` |
| Diretório de saída | `/.` |
| API | Não utilizada |
| Secret de implantação | `AZURE_STATIC_WEB_APPS_API_TOKEN_HAPPY_MEADOW_002B01910` |

Pull requests direcionados ao `main` também criam ambientes de preview. O ambiente é removido quando o pull request é fechado.

Para acompanhar uma publicação:

```powershell
gh run list --workflow "Azure Static Web Apps CI/CD"
```

## Limitações e segurança

Este projeto é uma prova de conceito. Ele usa o fluxo implícito com `response_type=id_token` e direciona o token para `jwt.ms` exclusivamente para inspeção durante a demonstração.

Para produção:

- Use Authorization Code Flow com PKCE.
- Valide tokens no backend ou em uma camada confiável.
- Valide issuer, audience, assinatura, expiração e nonce.
- Não exponha tokens, segredos ou dados pessoais em logs.
- Substitua `jwt.ms` por uma URI controlada pela aplicação.
- Configure Content Security Policy e cabeçalhos HTTP de segurança.

## Solução de problemas

### O SSO solicita login novamente

- Confirme que os aplicativos usam o mesmo tenant e User Flow.
- Verifique se a navegação ocorre na mesma aba.
- Remova `prompt=login` da URL de autorização.
- Confira se cookies do domínio `*.ciamlogin.com` estão permitidos.

### A autenticação retorna erro de redirect URI

- Verifique se `REDIRECT_URI` está cadastrado exatamente igual no registro do aplicativo.
- Confirme protocolo, domínio, caminho e barra final.

### O botão Copiar URL não funciona

- Execute o portal em `localhost` ou HTTPS.
- Confira a permissão de clipboard do navegador.
- Use a URL exibida no card como alternativa.

### A publicação não inicia

- Confirme que o push foi feito no branch `main`.
- Verifique o workflow em **Actions** no GitHub.
- Confirme que o secret de implantação existe e está válido.

## Compatibilidade

A interface foi projetada para versões atuais de Edge, Chrome, Firefox e Safari, em desktop e dispositivos móveis.
