# rftributaria-posts

Fila de publicação do Instagram **@rftributaria**. O Claude gera o card e a legenda e faz push para `queue/`; o GitHub Actions publica pela API oficial da Meta.

```
Claude (tarefa agendada)          GitHub                         Instagram
  gera card.jpg + caption.txt  →  push em queue/AAAA-MM-DD-HHhMM/
                                  Action "Publicar no Instagram"
                                  └ POST /media + /media_publish  →  post no @rftributaria
                                  └ grava published.json (permalink)
```

## Configuração (uma vez só)

### 1. Repositório
Crie um repositório **público** chamado `rftributaria-posts` e suba estes arquivos. Precisa ser público porque a Meta baixa a imagem por um link aberto (o conteúdo vai ser público no Instagram de qualquer jeito).

### 2. App na Meta
1. developers.facebook.com → **Meus apps → Criar app** → tipo **Empresa**.
2. Adicione o produto **Instagram** (opção "API com Login do Facebook").
3. O app pode ficar em modo **Desenvolvimento** — funciona para as suas próprias contas.

### 3. Token que não expira (recomendado: Usuário do Sistema)
1. business.facebook.com → **Configurações do negócio → Usuários → Usuários do sistema → Adicionar** (função Administrador).
2. **Atribuir ativos**: a Página do Facebook ligada ao @rftributaria (controle total) e o app criado no passo 2.
3. **Gerar token** → escolha o app → validade **Nunca** → marque:
   `instagram_basic`, `instagram_content_publish`, `pages_show_list`, `pages_read_engagement`, `business_management`.
4. Copie o token. **Não cole no chat** — ele vai direto para o GitHub no passo 5.

### 4. Descobrir o IG_USER_ID
No Graph API Explorer (developers.facebook.com/tools/explorer), com o token acima:
```
GET me/accounts?fields=name,instagram_business_account{id,username}
```
O `id` dentro de `instagram_business_account` (com `username: rftributaria`) é o **IG_USER_ID** — um número de ~17 dígitos.

### 5. Secrets no GitHub
Repositório → **Settings → Secrets and variables → Actions → New repository secret**:
| Nome | Valor |
|---|---|
| `IG_USER_ID` | o número do passo 4 |
| `IG_ACCESS_TOKEN` | o token do passo 3 |

Depois rode **Actions → Checar token do Instagram → Run workflow**. Verde = pronto.

### 6. Acesso de escrita para o Claude
GitHub → **Settings (do seu perfil) → Developer settings → Fine-grained tokens → Generate**:
- Repository access: **Only select repositories** → `rftributaria-posts`
- Permissions → Repository → **Contents: Read and write** (nada mais)
- Expiração: 1 ano

Esse token só consegue mexer neste repositório. É ele que vai na tarefa agendada do Claude.

## Uso manual
Crie `queue/2026-10-01-08h00/` com `card.jpg` e `caption.txt`, faça commit na `main` e o post sai em ~30 s. Resultado em `published.json`; falha em `error.txt` + e-mail do GitHub.

## Limites
- Instagram aceita até 100 publicações por API a cada 24 h (usamos 5).
- Imagem: JPEG, proporção entre 4:5 e 1.91:1 — o card 1080x1350 é 4:5, no limite certo.
- Legenda: até 2.200 caracteres e 30 hashtags.
