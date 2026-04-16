# 🗂️ API Exemplo — Node.js + Express + Prisma + Supabase + Render

> Stack: **Node.js + Express + Prisma + PostgreSQL (Supabase) + Storage (Supabase) + Render**

---

## 📋 Pré-requisitos

- Conta no [GitHub](https://github.com)
- Conta no [Supabase](https://supabase.com)
- Conta no [Render](https://render.com)
- Node.js instalado localmente
- Postman para testes

---

## 📁 Estrutura do Projeto

```
api-exemplo/
├── prisma/
│   ├── migrations/
│   ├── schema.prisma
│   └── seed.js
├── src/
│   ├── controllers/
│   │   ├── exemploController.js
│   │   └── arquivoController.js
│   ├── lib/
│   │   ├── helpers/
│   │   │   └── arquivoHelper.js
│   │   ├── middlewares/
│   │   │   ├── apiKey.js
│   │   │   └── fileGate.js
│   │   └── services/
│   │       ├── prismaClient.js
│   │       └── supabase.js
│   ├── models/
│   │   └── ExemploModel.js
│   ├── routes/
│   │   ├── exemploRoute.js
│   │   └── arquivoRoute.js
│   └── server.js
├── .env                ← nunca suba para o GitHub
├── .gitignore
├── package.json
└── prisma.config.js
```

---

## 🔁 Fluxo da Requisição

```
Cliente (Postman)
   ↓
Render (API Node rodando aqui)
   ↓
apiKey.js (valida X-API-Key)
   ↓
Multer + fileGate.js (recebe arquivo em memória)
   ↓
Validação + regra de negócio
   ↓
Supabase Storage (salva foto ou documento)
   ↓
Prisma + PostgreSQL Supabase (salva URL no banco)
```

---

## 🛣️ Rotas da API

| Método | Rota                          | Descrição                |
| ------ | ----------------------------- | ------------------------ |
| GET    | `/`                           | Status da API            |
| POST   | `/api/exemplos`               | Criar exemplo            |
| GET    | `/api/exemplos`               | Listar exemplos          |
| GET    | `/api/exemplos/:id`           | Buscar exemplo por ID    |
| PUT    | `/api/exemplos/:id`           | Atualizar exemplo        |
| DELETE | `/api/exemplos/:id`           | Deletar exemplo          |
| POST   | `/api/exemplos/:id/foto`      | Upload de foto           |
| GET    | `/api/exemplos/:id/foto`      | Retorna URL da foto      |
| DELETE | `/api/exemplos/:id/foto`      | Remover foto             |
| POST   | `/api/exemplos/:id/documento` | Upload de documento      |
| GET    | `/api/exemplos/:id/documento` | Retorna URL do documento |
| DELETE | `/api/exemplos/:id/documento` | Remover documento        |

---

## 💻 Rodando o Projeto em um PC Novo (Ex: PC da Escola)

> Siga esses passos toda vez que clonar o repositório em uma máquina nova.

### 1. Clonar o repositório

```bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO
```

### 2. Instalar as dependências

```bash
npm i
```

### 3. Criar o arquivo `.env`

O `.env` **nunca é enviado ao GitHub** (está no `.gitignore`), então você precisa criá-lo manualmente. Crie o arquivo `.env` na raiz do projeto com o seguinte conteúdo:

```env
PORT=3000
API_KEY="exemplo-backend-2026"

# Supabase — conexão via pooler (aplicação)
DATABASE_URL="postgresql://postgres.SEUPROJETO:SUASENHA@aws-1-sa-east-1.pooler.supabase.com:6543/postgres?pgbouncer=true"

# Supabase — conexão direta (migrations)
DIRECT_URL="postgresql://postgres.SEUPROJETO:SUASENHA@aws-1-sa-east-1.pooler.supabase.com:5432/postgres"

# Supabase Storage
SUPABASE_URL="https://SEUPROJETO.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
```

> ⚠️ Substitua os valores pelas suas credenciais reais do Supabase. As URLs você pega em **Connect → ORM → Prisma** e a Service Role Key em **Project Settings → API Keys**.

### 4. Gerar o Prisma Client

```bash
npx prisma generate
```

### 5. Rodar o servidor

```bash
npm run dev
```

Deve aparecer:
```
🚀 Servidor rodando em http://localhost:3000
```

> ✅ Pronto! O banco já está no Supabase, então não precisa rodar migrations novamente.

---

## Etapa 1 — Desenvolvimento Local

### 1.1 Configurar `.env` local

Crie o arquivo `.env` na raiz do projeto com as credenciais do **PostgreSQL local**:

```env
# Servidor
PORT=3000

# Banco de dados local
DATABASE_URL="postgresql://postgres:amods@localhost:7777/exemplo_db"
```

### 1.2 Schema do banco

`prisma/schema.prisma`:

```prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "postgresql"
}

model Exemplo {
    id        Int       @id @default(autoincrement())
    nome      String
    foto      String?
    documento String?

    @@map("exemplo")
}
```

### 1.3 Testar rotas no Postman

Use `http://localhost:3000` diretamente. Teste cada rota abaixo:

**Criar exemplo:**
```
POST http://localhost:3000/api/exemplos
Content-Type: application/json

{
    "nome": "Omega"
}
```

**Listar exemplos:**
```
GET http://localhost:3000/api/exemplos
```

**Buscar por ID:**
```
GET http://localhost:3000/api/exemplos/1
```

**Atualizar exemplo:**
```
PUT http://localhost:3000/api/exemplos/1
Content-Type: application/json

{
    "nome": "Exemplo Omega"
}
```

**Deletar exemplo:**
```
DELETE http://localhost:3000/api/exemplos/1
```

> ✅ CRUD funcionando? Avance para a Etapa 2.

---

## Etapa 2 — Banco de Dados no Supabase

### 2.1 Criar projeto no Supabase

1. Acesse [supabase.com](https://supabase.com) e faça login
2. Clique em **New project**
3. Preencha:
   - **Name:** `exemplo-db`
   - **Database Password:** anote essa senha
   - **Region:** `South America (São Paulo)`
4. Em **Security**, configure:
   - **Enable Data API:** ❌ desativado
   - **Enable automatic RLS:** ❌ desativado
5. Clique em **Create new project** e aguarde ~2 minutos

> ⚠️ Ambos desativados pois usamos conexão direta ao PostgreSQL via `DATABASE_URL` com Prisma. O controle de acesso é feito pela API Key da nossa aplicação.

### 2.2 Pegar a Connection String

1. No painel do projeto, clique em **Connect**
2. Selecione **ORM** → **Prisma**
3. Copie as variáveis exibidas em `.env.local` e adicione no seu `.env`
4. Comente a `DATABASE_URL` local e adicione as duas novas:

```env
# Servidor
PORT=3000
API_KEY="exemplo-backend-2026"

# Banco de dados local (comentado)
# DATABASE_URL="postgresql://postgres:senha@localhost:7777/exemplo_db"

# Supabase — conexão via pooler (aplicação)
DATABASE_URL="postgresql://..."

# Supabase — conexão direta (migrations)
DIRECT_URL="postgresql://..."
```

5. Substitua `[YOUR-PASSWORD]` pela senha anotada na etapa 2.1

> ⚠️ Os colchetes `[YOUR-PASSWORD]` fazem parte do placeholder — apague-os junto com o texto e coloque só a senha. Exemplo: se sua senha é `minhasenha123`, fica `...pooler.supabase.com:5432/postgres` com `:minhasenha123@` no meio.

### 2.3 Ajustar o `prisma.config.js`

O Supabase usa PgBouncer (pooling) por padrão. O Prisma precisa de conexão direta para migrations, por isso usa `DIRECT_URL`. O arquivo deve ficar assim:

```js
import 'dotenv/config';
import { defineConfig } from 'prisma/config';

export default defineConfig({
    schema: 'prisma/schema.prisma',
    migrate: {
        datasource: {
            url: process.env['DIRECT_URL'],
        },
    },
});
```

> ⚠️ A `datasource` com `url` precisa estar **dentro de `migrate`** — se estiver no nível raiz, o Prisma não encontra a URL e retorna erro.

### 2.4 Rodar a migration

```bash
npx prisma migrate dev --name init
npx prisma generate
```

> ⚠️ Se aparecer o erro `Drift detected` ou `migration history out of sync`, rode:
> ```bash
> npx prisma migrate reset
> ```
> Confirme com `y`. Isso apaga e recria tudo — use só em desenvolvimento.

### 2.5 Rodar o Seed

```bash
npx prisma db seed
```

> Navegue no Supabase: **Table Editor**, **SQL Editor** e **Schema Visualizer** para confirmar os dados.

### 2.6 Testar rotas no Postman

A API ainda roda local — apenas o banco agora é o Supabase. Em **todas** as requisições, adicione o header:

| Key | Value |
|-----|-------|
| `X-API-Key` | `exemplo-backend-2026` |

**Criar exemplo:**
```
POST http://localhost:3000/api/exemplos
Content-Type: application/json

{
    "nome": "Omega"
}
```

**Listar exemplos:**
```
GET http://localhost:3000/api/exemplos
```

**Buscar por ID:**
```
GET http://localhost:3000/api/exemplos/1
```

**Atualizar exemplo:**
```
PUT http://localhost:3000/api/exemplos/1
Content-Type: application/json

{
    "nome": "Exemplo Omega"
}
```

**Deletar exemplo:**
```
DELETE http://localhost:3000/api/exemplos/1
```

> ✅ CRUD funcionando com banco no Supabase? Avance para a Etapa 3.

---

## Etapa 3 — Rotas de Arquivo (foto e documento)

### 3.1 Criar `src/lib/middlewares/apiKey.js`

```js
export const apiKey = (req, res, next) => {
    const key = req.headers['x-api-key'];

    if (!key || key !== process.env.API_KEY) {
        return res.status(401).json({ error: 'API Key inválida ou ausente.' });
    }

    next();
};
```

### 3.2 Adicionar no `.env`

```env
# Servidor
PORT=3000
API_KEY="exemplo-backend-2026"
```

### 3.3 Registrar em `src/server.js`

```js
import { apiKey } from './lib/middlewares/apiKey.js';

app.use('/api/exemplos', apiKey, exemplosRoutes);
```

### 3.4 Testar no Postman

**Sem API Key — deve retornar 401**

**Com API Key — deve funcionar normalmente**

> No Postman, vá em **Headers** e adicione `X-API-Key: exemplo-backend-2026` em todas as requisições.

### 3.5 Criar `src/lib/middlewares/fileGate.js`

```js
import multer from 'multer';

const storage = multer.memoryStorage();

const tiposPermitidos = [
    'image/jpeg',
    'image/png',
    'image/webp',
    'image/gif',
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'application/vnd.ms-excel',
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    'text/plain',
    'text/csv',
];

const fileFilter = (req, file, cb) => {
    tiposPermitidos.includes(file.mimetype)
        ? cb(null, true)
        : cb(new Error('Tipo de arquivo não permitido.'));
};

export const upload = multer({
    storage,
    fileFilter,
    limits: { fileSize: 1 * 1024 * 1024 }, // 1MB
});
```

> ⚠️ Arquivos ficam em memória (`memoryStorage`) — não são salvos em disco, vão direto pro Supabase Storage.
> ⚠️ Os tipos permitidos são verificados pelo `mimetype` do arquivo, não pela extensão, para evitar falsificação.

---

## Etapa 4 — Storage no Supabase (foto e documento)

### 4.1 Criar bucket no Supabase

1. No painel do Supabase, acesse **Storage → New bucket**
2. Preencha:
   - **Name:** `arquivos`
   - **Public bucket:** ✅ ativado
   - **Restrict file size:** ❌ desativado
   - **Restrict MIME types:** ❌ desativado
3. Clique em **Save**

> ⚠️ O bucket é público pois toda a segurança é gerenciada no backend via API Key.

### 4.2 Criar Policy no bucket

Após criar o bucket, vá em **Storage → Policies** e crie uma nova policy:

1. Clique em **New policy** no bucket `arquivos`
2. Escolha **For full customization**
3. Preencha:
   - **Policy name:** `allow all`
   - Marque todas as operações: **SELECT, INSERT, UPDATE, DELETE**
   - **USING expression:** `true`
   - **WITH CHECK expression:** `true`
4. Clique em **Save**

> ⚠️ Sem essa policy, o upload retorna erro `new row violates row-level security policy` mesmo com o bucket público.

### 4.3 Criar `src/lib/services/supabase.js`

```js
import { createClient } from '@supabase/supabase-js';
import 'dotenv/config';

const supabase = createClient(
    process.env.SUPABASE_URL,
    process.env.SUPABASE_SERVICE_ROLE_KEY
);

export default supabase;
```

### 4.4 Pegar credenciais do Supabase

1. Na **Project Overview (home)** do projeto, copie a `SUPABASE_URL` com `Project URL`
2. Em **Project Settings → Configuration → API Keys**:
   - Em **Legacy anon, service_role API keys**, copie a `service_role secret` (clique em **Reveal**)

> ⚠️ Não confunda com a chave `anon` — você precisa da `service_role`. A diferença: no JWT da `anon` aparece `"role":"anon"`, na `service_role` aparece `"role":"service_role"`.

### 4.5 Adicionar no `.env`

```env
# Supabase Storage
SUPABASE_URL="https://xxxx.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJ..."
```

> ⚠️ Nunca suba o `.env` para o GitHub. Use `service_role` apenas no backend — ela tem permissão total.

### 4.6 Criar `src/lib/helpers/arquivoHelper.js`

```js
import sharp from 'sharp';
import supabase from '../services/supabase.js';

const BUCKET = 'arquivos';

const prepararFoto = async (buffer) =>
    sharp(buffer)
        .resize({ width: 800, withoutEnlargement: true })
        .webp({ quality: 80 })
        .toBuffer();

export const upload = async (id, file) => {
    const ehFoto = file.mimetype.startsWith('image/');

    const buffer = ehFoto ? await prepararFoto(file.buffer) : file.buffer;
    const path = ehFoto
        ? `${id}/foto.webp`
        : `${id}/documento.${file.originalname.split('.').pop()}`;
    const contentType = ehFoto ? 'image/webp' : file.mimetype;

    const { error } = await supabase.storage
        .from(BUCKET)
        .upload(path, buffer, { contentType, upsert: true });

    if (error) throw new Error(error.message);

    return supabase.storage.from(BUCKET).getPublicUrl(path).data.publicUrl;
};

export const deletar = async (url) => {
    const path = url.split(`${BUCKET}/`)[1];
    const { error } = await supabase.storage.from(BUCKET).remove([path]);
    if (error) throw new Error(error.message);
};
```

> O Sharp converte jpeg, png, webp e gif para `.webp`, padronizando o formato e reduzindo o tamanho.

### 4.7 Criar `src/controllers/arquivoController.js`

```js
import ExemploModel from '../models/ExemploModel.js';
import {
    upload as uploadStorage,
    deletar as deletarStorage,
} from '../lib/helpers/arquivoHelper.js';

const uploadArquivo = (tipo) => async (req, res) => {
    try {
        const { id } = req.params;
        if (isNaN(id)) return res.status(400).json({ error: 'ID inválido.' });

        const exemplo = await ExemploModel.buscarPorId(parseInt(id));
        if (!exemplo) return res.status(404).json({ error: 'Registro não encontrado.' });
        if (!req.file) return res.status(400).json({ error: 'Nenhum arquivo enviado.' });

        if (exemplo[tipo]) await deletarStorage(exemplo[tipo]);
        exemplo[tipo] = await uploadStorage(id, req.file);
        const data = await exemplo.atualizar();

        return res.status(200).json({ message: `${tipo} enviado com sucesso!`, url: data[tipo] });
    } catch (error) {
        console.error(`Erro ao fazer upload do ${tipo}:`, error);
        return res.status(500).json({ error: `Erro ao fazer upload do ${tipo}.` });
    }
};

const buscarArquivo = (tipo) => async (req, res) => {
    try {
        const { id } = req.params;
        if (isNaN(id)) return res.status(400).json({ error: 'ID inválido.' });

        const exemplo = await ExemploModel.buscarPorId(parseInt(id));
        if (!exemplo) return res.status(404).json({ error: 'Registro não encontrado.' });
        if (!exemplo[tipo]) return res.status(404).json({ error: `Nenhum ${tipo} cadastrado.` });

        return res.status(200).json({ url: exemplo[tipo] });
    } catch (error) {
        console.error(`Erro ao buscar ${tipo}:`, error);
        return res.status(500).json({ error: `Erro ao buscar ${tipo}.` });
    }
};

const deletarArquivo = (tipo) => async (req, res) => {
    try {
        const { id } = req.params;
        if (isNaN(id)) return res.status(400).json({ error: 'ID inválido.' });

        const exemplo = await ExemploModel.buscarPorId(parseInt(id));
        if (!exemplo) return res.status(404).json({ error: 'Registro não encontrado.' });
        if (!exemplo[tipo]) return res.status(404).json({ error: `Nenhum ${tipo} para remover.` });

        await deletarStorage(exemplo[tipo]);
        exemplo[tipo] = null;
        await exemplo.atualizar();

        return res.status(200).json({ message: `${tipo} removido com sucesso!` });
    } catch (error) {
        console.error(`Erro ao remover ${tipo}:`, error);
        return res.status(500).json({ error: `Erro ao remover ${tipo}.` });
    }
};

export const uploadFoto = uploadArquivo('foto');
export const buscarFoto = buscarArquivo('foto');
export const deletarFoto = deletarArquivo('foto');

export const uploadDocumento = uploadArquivo('documento');
export const buscarDocumento = buscarArquivo('documento');
export const deletarDocumento = deletarArquivo('documento');
```

### 4.8 Criar `src/routes/arquivoRoute.js`

```js
import express from 'express';
import * as controller from '../controllers/arquivoController.js';
import { upload } from '../lib/middlewares/fileGate.js';

const router = express.Router();

router.post('/:id/foto', upload.single('foto'), controller.uploadFoto);
router.get('/:id/foto', controller.buscarFoto);
router.delete('/:id/foto', controller.deletarFoto);

router.post('/:id/documento', upload.single('documento'), controller.uploadDocumento);
router.get('/:id/documento', controller.buscarDocumento);
router.delete('/:id/documento', controller.deletarDocumento);

export default router;
```

### 4.9 Registrar em src/server.js

```js
import arquivoRoutes from './routes/arquivoRoute.js';

app.use('/api/exemplos', apiKey, arquivoRoutes);
```

### 4.10 Testar no Postman

**Upload de foto:**
```
POST http://localhost:3000/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
Body → form-data
  key: foto  |  type: File  |  value: [selecione uma imagem]
```

> ⚠️ No campo `key`, clique no dropdown ao lado e troque de **Text** para **File** antes de selecionar o arquivo.

**Buscar foto:**
```
GET http://localhost:3000/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
```

**Deletar foto:**
```
DELETE http://localhost:3000/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
```

**Upload de documento:**
```
POST http://localhost:3000/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
Body → form-data
  key: documento  |  type: File  |  value: [selecione um PDF, Word ou Excel]
```

**Buscar documento:**
```
GET http://localhost:3000/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
```

**Deletar documento:**
```
DELETE http://localhost:3000/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
```

> Confirme no Supabase: **Storage → arquivos → pasta `1/`** que os arquivos aparecem após o upload e somem após o delete.

> ✅ Storage funcionando? Avance para a Etapa 5 — Deploy no Render.

---

## Etapa 5 — Deploy no Render

### 5.1 Criar conta e conectar GitHub no Render

1. Acesse [render.com](https://render.com) e crie uma conta associando sua conta **GitHub** e autorizando o acesso.

### 5.2 Criar o Web Service

1. No dashboard, clique em **New → Web Service**
2. Selecione o repositório da API
3. Preencha:
   - **Name:** `Backend-Supabase-Render`
   - **Language:** `Node`
   - **Branch:** `main`
   - **Region:** `Oregon (US West) - melhor latência para o Brasil`
   - **Root Directory:** deixe em branco (a menos que package.json esteja em uma subpasta)
   - **Build Command:** `npm install && npx prisma generate`
   - **Start Command:** `node src/server.js`
   - **Instance Type:** `Free`

### 5.3 Configurar variáveis de ambiente

Em **Environment → Add Environment Variable**, adicione todas as variáveis do `.env`:

| Key | Value |
|-----|-------|
| `PORT` | `3000` |
| `API_KEY` | `exemplo-backend-2026` |
| `DATABASE_URL` | `postgresql://...` (pooler) |
| `DIRECT_URL` | `postgresql://...` (direct) |
| `SUPABASE_URL` | `https://xxxx.supabase.co` |
| `SUPABASE_SERVICE_ROLE_KEY` | `eyJ...` |

### 5.4 Fazer o deploy

Clique em **Deploy Web Service** e acompanhe os logs. O deploy foi bem sucedido quando aparecer:

```
==> Build successful 🎉
🚀 Servidor rodando em http://localhost:3000
==> Your service is live 🎉
==> Available at your primary URL https://[nome-do-seu-projeto].onrender.com
```

> ⚠️ No plano free o Render hiberna após 15 minutos de inatividade. A primeira requisição pode levar ~30 segundos.

### 5.5 Problemas comuns no Render

**❌ Erro: `Cannot find module`**

O `node_modules` não foi instalado. Verifique se o **Build Command** está como:
```
npm install && npx prisma generate
```

**❌ Erro: `prisma generate` falhou**

O Prisma Client não foi gerado. Confirme que o Build Command inclui `npx prisma generate`.

**❌ Erro: `supabaseUrl is required`**

As variáveis de ambiente não foram configuradas. Verifique em **Environment** se `SUPABASE_URL` e `SUPABASE_SERVICE_ROLE_KEY` estão preenchidas corretamente (sem aspas extras).

**❌ Erro de conexão com o banco**

Verifique se `DATABASE_URL` no Render está usando a URL do **pooler** (porta 6543), não a direta. A URL direta (porta 5432) é só para migrations locais via `DIRECT_URL`.

**❌ API retorna 401 em todas as rotas**

A variável `API_KEY` não está configurada no Render. Adicione `API_KEY` com o valor `exemplo-backend-2026` nas variáveis de ambiente.

**❌ Deploy não atualiza após push**

Por padrão o Render faz deploy automático a cada push na branch `main`. Se não estiver acontecendo, vá em **Settings → Auto-Deploy** e confirme que está ativado.

### 5.6 Testar no Postman com a URL do Render

Anote a URL fornecida pelo Render e configure uma variável de ambiente no Postman:

1. No Postman, clique em **Environments** (ícone de olho no canto superior direito)
2. Clique em **Add**
3. Adicione:
   - **Variable:** `base_url`
   - **Initial Value:** `https://[nome-do-seu-projeto].onrender.com`
4. Clique em **Save** e selecione o environment criado

Agora use `{{base_url}}` nas requisições:

**Status da API:**
```
GET {{base_url}}/
```

**Criar exemplo:**
```
POST {{base_url}}/api/exemplos
Header: X-API-Key: exemplo-backend-2026
Content-Type: application/json

{
    "nome": "Deploy Omega"
}
```

**Listar exemplos:**
```
GET {{base_url}}/api/exemplos
Header: X-API-Key: exemplo-backend-2026
```

**Buscar por ID:**
```
GET {{base_url}}/api/exemplos/1
Header: X-API-Key: exemplo-backend-2026
```

**Atualizar exemplo:**
```
PUT {{base_url}}/api/exemplos/1
Header: X-API-Key: exemplo-backend-2026
Content-Type: application/json

{
    "nome": "Exemplo Omega Atualizado"
}
```

**Deletar exemplo:**
```
DELETE {{base_url}}/api/exemplos/1
Header: X-API-Key: exemplo-backend-2026
```

**Upload de foto:**
```
POST {{base_url}}/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
Body → form-data
  key: foto  |  type: File  |  value: [selecione uma imagem]
```

**Buscar foto:**
```
GET {{base_url}}/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
```

**Deletar foto:**
```
DELETE {{base_url}}/api/exemplos/1/foto
Header: X-API-Key: exemplo-backend-2026
```

**Upload de documento:**
```
POST {{base_url}}/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
Body → form-data
  key: documento  |  type: File  |  value: [selecione um PDF, Word ou Excel]
```

**Buscar documento:**
```
GET {{base_url}}/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
```

**Deletar documento:**
```
DELETE {{base_url}}/api/exemplos/1/documento
Header: X-API-Key: exemplo-backend-2026
```

> ✅ API respondendo pela URL do Render? Deploy concluído!

---

## Etapa 6 — Documentação no Postman (link público)

O Postman permite gerar uma documentação pública com um link para compartilhar a API.

### 6.1 Organizar as requisições em uma Collection

1. No Postman, clique em **Collections** no menu lateral
2. Clique em **+** para criar uma nova collection
3. Nomeie como `API Exemplo`
4. Arraste todas as suas requisições para dentro da collection
5. Organize em pastas se quiser (ex: `Exemplos`, `Fotos`, `Documentos`)

### 6.2 Adicionar descrições

1. Clique na collection e depois em **Edit**
2. Adicione uma descrição geral da API
3. Para cada requisição, clique nela e adicione uma descrição no campo **Description**

### 6.3 Publicar a documentação

1. Clique nos **três pontos (...)** ao lado da collection
2. Clique em **View Documentation**
3. No canto superior direito, clique em **Publish**
4. Configure:
   - **Environment:** selecione o environment com `base_url` apontando para o Render
   - Deixe as demais opções padrão
5. Clique em **Publish**
6. Copie o link gerado — ele é público e pode ser compartilhado com qualquer pessoa

> O link fica no formato: `https://documenter.getpostman.com/view/XXXXXXX/XXXXXXX`

---

## Casos de Erros Comuns

### Após alterar schema.prisma

```bash
npx prisma migrate dev --name nome_da_alteracao
npx prisma generate
```

### Banco inconsistente (⚠️ APAGA TUDO)

```bash
npx prisma migrate reset
```

### Apenas popular dados

```bash
npx prisma db seed
```

### package.json — script dev no Mac/Linux

O comando `cls` não existe no Mac/Linux. Use:

```json
"dev": "nodemon src/server.js"
```

### Dicas

| Comando | Apaga dados? | Cria migrations? | Uso |
| --- | --- | --- | --- |
| `migrate dev` | ❌ Não | ✅ Sim | Alterar schema |
| `db seed` | ❌ Não | ❌ Não | Popular dados |
| `migrate reset` | ✅ SIM | ❌ Não | DEV only |

> ⚠️ No plano free o Render hiberna após 15 minutos de inatividade. A primeira requisição pode levar ~30 segundos.

> ✅ API respondendo pela URL do Render? Deploy concluído!
