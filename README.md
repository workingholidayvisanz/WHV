# WHV Scripts — Setup Guide (Netlify + Apps Script)

## Por que Netlify em vez de GitHub Pages?

| | GitHub Pages | Netlify |
|---|---|---|
| Repositório privado | ❌ Não suporta (plano gratuito) | ✅ Suporta |
| Deploy gratuito | ✅ | ✅ |
| HTTPS automático | ✅ | ✅ |
| Código visível ao público | ⚠️ Sim | ✅ Não |

Com Netlify, o repositório fica **privado no GitHub** e apenas o site compilado fica público. Ninguém vê seu código-fonte.

## Arquitetura de segurança

```
┌─────────────────────────────────────────────────────┐
│  index.html (público no Netlify)                    │
│  ✅ Só tem o formulário e a lista de países         │
│  ❌ Não tem nenhuma lógica de geração de arquivos   │
│  ❌ Não tem nenhuma lógica de negócio copiável      │
└─────────────────────────┬───────────────────────────┘
                          │ POST com dados brutos
                          ▼
┌─────────────────────────────────────────────────────┐
│  Code.gs (Google Apps Script — PRIVADO)             │
│  ✅ Toda a lógica de geração dos 7 JSONs fica aqui  │
│  ✅ Cria o ZIP, salva no Drive, envia email         │
│  ❌ Ninguém externo tem acesso a este código        │
└─────────────────────────────────────────────────────┘
```

Se alguém clonar o `index.html`, vai ter apenas um formulário que envia dados para uma URL — sem a URL válida do seu Apps Script, não gera nada.

---

## PASSO 1 — Configurar o Google Drive

1. Acesse [drive.google.com](https://drive.google.com)
2. Crie uma pasta chamada **`WHV Submissions`**
3. Abra a pasta, copie o **ID** da URL:
   ```
   https://drive.google.com/drive/folders/ESTE_TRECHO_É_O_ID
   ```

---

## PASSO 2 — Criar a Planilha de Controle

1. Acesse [sheets.google.com](https://sheets.google.com)
2. Crie uma planilha em branco → chame de **`WHV Controle`**
3. Copie o **ID** da URL:
   ```
   https://docs.google.com/spreadsheets/d/ESTE_TRECHO_É_O_ID/edit
   ```

---

## PASSO 3 — Configurar o Google Apps Script

1. Acesse [script.google.com](https://script.google.com)
2. Clique em **+ Novo projeto** → nomeie `WHV Scripts API`
3. Apague todo o código padrão e cole o conteúdo do arquivo `Code.gs`
4. Preencha as 3 variáveis no topo:
   ```javascript
   var CONFIG = {
     SEU_EMAIL:       'seu@email.com',
     DRIVE_FOLDER_ID: 'id_copiado_no_passo_1',
     SHEET_ID:        'id_copiado_no_passo_2',
   };
   ```
5. Salve (Ctrl+S)
6. Clique em **Implantar → Nova implantação**
7. Clique no ⚙️ ao lado de "Selecionar tipo" → **App da Web**
8. Configure:
   - Executar como: **Eu mesmo**
   - Quem tem acesso: **Qualquer pessoa**
9. Clique em **Implantar** e autorize as permissões solicitadas
10. **Copie a URL** gerada — começa com:
    ```
    https://script.google.com/macros/s/XXXXXXXX/exec
    ```

---

## PASSO 4 — Conectar o formulário ao Apps Script

Abra o `index.html` e substitua na linha indicada:

```javascript
// Antes:
var APPS_SCRIPT_URL = 'COLE_AQUI_A_URL_DO_SEU_APPS_SCRIPT';

// Depois:
var APPS_SCRIPT_URL = 'https://script.google.com/macros/s/XXXXXXXX/exec';
```

---

## PASSO 5 — Publicar no Netlify (repositório privado)

### 5a. Criar o repositório privado no GitHub
1. Acesse [github.com](https://github.com) → **New repository**
2. Nome: `whv-scripts` (ou qualquer nome)
3. Marque **Private** ← importante
4. Clique em **Create repository**
5. Faça upload do arquivo `index.html`

### 5b. Conectar ao Netlify
1. Acesse [netlify.com](https://netlify.com) e crie uma conta gratuita
2. Clique em **Add new site → Import an existing project**
3. Escolha **Deploy with GitHub**
4. Autorize o Netlify a acessar sua conta do GitHub
5. Selecione o repositório `whv-scripts`
6. Configurações de build (deixe tudo em branco — é só HTML estático):
   - Build command: *(vazio)*
   - Publish directory: *(vazio ou `.`)*
7. Clique em **Deploy site**
8. Aguarde ~30 segundos. Seu site estará em:
   ```
   https://nome-aleatorio.netlify.app
   ```
9. Para ter uma URL personalizada: **Site settings → Domain management → Add custom domain**

---

## PASSO 6 — Testar

1. Acesse a URL do Netlify
2. Preencha o formulário com dados fictícios
3. Clique em "Enviar meus dados"
4. Verifique:
   - ✅ Você recebeu o email de notificação
   - ✅ O ZIP apareceu na pasta do Drive (verificar se os 6 ou 7 arquivos estão corretos)
   - ✅ A linha foi adicionada na planilha
   - ✅ O cliente vê a tela "Dados enviados!"

---

## Atualizando o site depois

Quando precisar mudar algo no `index.html`:
1. Edite o arquivo e salve
2. No GitHub, abra o arquivo → clique no lápis → cole o novo conteúdo → **Commit changes**
3. O Netlify detecta automaticamente e republica em ~30 segundos

---

## Atualizando o Apps Script depois

Quando precisar mudar a lógica de geração dos arquivos:
1. Edite o `Code.gs` no script.google.com
2. **Implantar → Gerenciar implantações → ✏️ Editar → Nova versão → Implantar**
3. A URL permanece a mesma

---

## Fluxo de trabalho após setup

```
📬 Email chega: "Nova submissão WHV: [Nome]"
   ↓
💳 Você entra em contato com o cliente por email
   Envia instruções de pagamento (Pix, PayPal, etc.)
   ↓
✅ Pagamento confirmado
   ↓
📂 Drive → pasta "WHV Submissions"
   Baixa o ZIP do cliente
   ↓
📧 Envia o ZIP por email para o cliente
   ↓
📝 Planilha → Status: "PAGO E ENVIADO"
```

---

## Solução de problemas

**"Erro ao enviar" no formulário**
- Verifique se a URL do Apps Script está correta no `index.html`
- Confirme que o deploy foi feito com acesso "Qualquer pessoa"
- Após qualquer mudança no `Code.gs`, sempre faça um novo deploy

**ZIP não aparece no Drive**
- Confirme o `DRIVE_FOLDER_ID` — copie só o trecho final da URL da pasta
- A pasta deve pertencer à mesma conta Google que fez o deploy do Apps Script

**Email não chegou**
- Verifique spam
- No Apps Script: menu **Execuções** → veja se há erros em vermelho

**"CORS error" no console do browser**
- Confirme que o deploy do Apps Script tem acesso "Qualquer pessoa"
- Tente fazer um novo deploy (criar nova versão)

---

## Custo total

| Serviço | Limite gratuito |
|---------|----------------|
| Netlify | 100 GB/mês de banda, deploys ilimitados |
| GitHub | Repositórios privados ilimitados |
| Google Apps Script | 6 min/execução, 100 emails/dia |
| Google Drive | 15 GB de armazenamento |

Para o volume inicial de vendas, **R$ 0**.
