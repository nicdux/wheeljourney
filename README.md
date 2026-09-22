# Identity Journey Demo

Experiência guiada e educacional para demonstrar conceitos do Microsoft Entra External ID em três aplicações web.

O launcher apresenta a jornada de criação de conta, o comportamento esperado de Single Sign-On (SSO) e uma aplicação sensível que pode exigir autenticação mais forte de acordo com a política do tenant. A interface usa identidade visual genérica, dark mode permanente e layout responsivo.

> Este projeto é uma prova de conceito educacional. A resposta de autenticação é redirecionada para `jwt.ms`; por isso, o launcher não recebe a resposta e não consegue confirmar autenticação, sessão, emissão de token, SSO, MFA ou conclusão do logout.

## Demonstração

Aplicação publicada no Azure Static Web Apps:

https://happy-meadow-002b01910.3.azurestaticapps.net

## Funcionalidades

- App01: **Create account / Onboarding**.
- App02: **Single Sign-On application**.
- App03: **Sensitive application / MFA**.
- Alternância entre Business View e Technical View sem alterar a autenticação.
- Jornada visual com estados limitados ao que o launcher pode observar.
- Metadados técnicos seguros, identificadores de aplicativo mascarados e URLs expansíveis.
- Claims fictícias e claramente identificadas como dados estáticos de demonstração.
- Geração e exibição das URLs de autorização OpenID Connect.
- Cópia individual ou conjunta das URLs de autenticação.
- Solicitação de logout no endpoint do tenant.
- Interface responsiva e acessível, com suporte a redução de movimento.

## Aplicações da demonstração

| Aplicação | Finalidade apresentada                                                                                       |
| --------- | ------------------------------------------------------------------------------------------------------------ |
| App01     | Iniciar uma nova jornada de identidade ou entrar com uma conta existente.                                    |
| App02     | Abrir outra aplicação usando a sessão de identidade que se espera ter sido estabelecida no tenant.           |
| App03     | Solicitar acesso a uma aplicação sensível, na qual a política do tenant pode exigir autenticação mais forte. |

As três aplicações mantêm seus Client IDs e mapeamentos originais. As URLs não incluem `prompt=login`, preservando o comportamento de SSO da PoC.

## Visualizações

### Business View

- Destaca os benefícios da jornada de identidade.
- Exibe os três aplicativos e a jornada guiada.
- Oculta tenant, User Flow, redirect URI, protocolo, scopes, response type, identificadores e URLs completas.

### Technical View

- Exibe apenas metadados técnicos seguros.
- Mascara os Client IDs na interface.
- Permite expandir e copiar as URLs de autorização existentes.
- Inclui claims fictícias para fins de apresentação.

O toggle altera somente a apresentação. Ele não modifica parâmetros, URLs, mapeamentos ou comportamento de autenticação.

## Estados da jornada

O launcher pode registrar apenas a intenção local iniciada por um clique:

- `Ready`
- `Action requested`
- `Not observable by this launcher`
- `Depends on tenant policy`

Após a navegação para o provedor de identidade, todos os resultados são externos ao launcher. A interface não afirma que autenticação, sessão, token, SSO, MFA ou sign-out foram concluídos.

## Estrutura

```text
.
|-- .github/
|   `-- workflows/
|       `-- azure-static-web-apps-happy-meadow-002b01910.yml
|-- index.html
|-- staticwebapp.config.json
`-- README.md
```

| Arquivo                                                              | Responsabilidade                                                                 |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `index.html`                                                         | Interface, visualizações, jornada guiada, dark mode, configuração e lógica OIDC. |
| `staticwebapp.config.json`                                           | Cabeçalhos HTTP de segurança da aplicação publicada.                             |
| `.github/workflows/azure-static-web-apps-happy-meadow-002b01910.yml` | Publicação automática no Azure Static Web Apps.                                  |
| `README.md`                                                          | Documentação funcional e operacional do projeto.                                 |

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

### Evolução para novos casos de uso

A aplicação atual é intencionalmente estática e independente de AKS, máquinas virtuais, registros de contêiner e cofres de segredo. Novos cenários puramente demonstrativos podem reutilizar o mesmo padrão de configuração no `index.html`, desde que usem apenas identificadores públicos e endpoints de autorização.

Antes de adicionar um novo caso de uso:

1. Defina o objetivo observável da jornada e evite afirmar resultados que o launcher não consegue verificar.
2. Crie ou selecione o registro de aplicativo e o User Flow no tenant de External ID.
3. Cadastre a URI de redirecionamento exata e mantenha secrets fora do frontend e do repositório.
4. Adicione textos equivalentes em `pt-BR`, `en-US` e `es-ES`.
5. Preserve os headers de segurança de `staticwebapp.config.json` e valide a publicação em HTTPS.

Casos que precisem receber o retorno de autenticação, validar tokens ou chamar APIs devem ser implementados como uma aplicação separada com Authorization Code Flow e PKCE. Essa evolução não deve substituir nem acoplar infraestrutura à demo estática existente.

## Protocolo de autenticação

A demonstração monta uma requisição para o endpoint `/oauth2/v2.0/authorize` com os parâmetros:

| Parâmetro       | Valor                                 |
| --------------- | ------------------------------------- |
| `p`             | User Flow configurado em `USER_FLOW`. |
| `client_id`     | Client ID do aplicativo selecionado.  |
| `nonce`         | Valor aleatório gerado no navegador.  |
| `redirect_uri`  | `https://jwt.ms`.                     |
| `scope`         | `openid profile`.                     |
| `response_type` | `id_token`.                           |

O logout usa `/oauth2/v2.0/logout`, o mesmo User Flow e `post_logout_redirect_uri`.

### Comportamento esperado e observabilidade

O tenant processa a interação de identidade e redireciona a resposta para `jwt.ms`. O launcher apenas inicia a solicitação e, portanto:

- não lê a resposta retornada;
- não extrai, decodifica, valida, armazena ou registra tokens;
- não confirma que uma sessão foi estabelecida;
- não confirma que SSO ou MFA ocorreram;
- não confirma que o logout foi concluído;
- não afirma que `jwt.ms` valida a decisão de autorização da aplicação.

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

| Propriedade           | Valor                                                    |
| --------------------- | -------------------------------------------------------- |
| Branch de produção    | `main`                                                   |
| Origem da aplicação   | `/`                                                      |
| Diretório de saída    | `/.`                                                     |
| API                   | Não utilizada                                            |
| Secret de implantação | `AZURE_STATIC_WEB_APPS_API_TOKEN_HAPPY_MEADOW_002B01910` |

Pull requests direcionados ao `main` também criam ambientes de preview. O ambiente é removido quando o pull request é fechado.

Para acompanhar uma publicação:

```powershell
gh run list --workflow "Azure Static Web Apps CI/CD"
```

## Limitações e segurança

Este projeto usa o fluxo implícito com `response_type=id_token` e redireciona a resposta para `jwt.ms` exclusivamente no contexto da demonstração educacional.

Client IDs são identificadores públicos, não credenciais. Mesmo assim, nunca coloque client secrets, credenciais, tokens, dados pessoais ou outros valores sensíveis no frontend, no repositório ou em logs.

Para produção:

- Use Authorization Code Flow com PKCE.
- Valide tokens no backend ou em uma camada confiável.
- Valide issuer, audience, assinatura, expiração e nonce.
- Não exponha tokens, segredos ou dados pessoais em logs.
- Não armazene nem copie tokens para o clipboard.
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
