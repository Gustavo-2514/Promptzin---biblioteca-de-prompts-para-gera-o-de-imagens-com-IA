# Promptzin

Link do projeto: [Promptzin](https://promptzin.xyz).

![Hero do projeto](./hero.webp)

## Descrição do projeto

Promptzin é um projeto de portfólio focado em ser uma biblioteca de prompts para geração de imagens com IA, onde o usuário pode: Publicar um prompt + imagem, salvar prompts de outros usuários, e buscar por diversos prompts dentro da aplicação, prontos para copiar e colar.

> A aplicação lida com dados reais de usuários. Por respeito à
> privacidade e para evitar exposição de vulnerabilidades,
> o código é privado. Recrutadores podem solicitar acesso via email: gustavoalbuquerquedev@gmail.com


## Features

- Busca de prompts por (Tags, Modelo)
- Copiar prompt com um clique
- Publicação de prompt + imagem (com moderação)
- Salvar prompts de outros usuários
- Autenticação com login social
- Painel admin com moderação de usuários e prompts
- Compressão automática de imagens com Sharp
- Rate Limit por ação com Upstash

## Stack utilizada

- **Next.js** (Framework)
- **Better Auth** (Autenticação/Sessão)
- **Shadcn + Tailwind** (UI)
- **React Masonry CSS** (Layout masonry)
- **Zod + React Hook Form** (Validação de dados e formulários)
- **tanstack Query** (Busca e cache de dados)
- **Prisma** (ORM)
- **Supabase** (Banco de dados - PostgreSQL)
- **Sharp** (Compressão de imagens)
- **Upstash** (Redis/Rate Limit)
- **Cloudflare** (Proteção contra bots/DDoS)
- **Vercel** (Hospedagem)

### Arquitetura

Optei por usar **Next.js** pelo forte foco em **SEO** e **Cache** pelo lado do servidor (por onde boa parte das chamadas do APP passa), e também porque em breve adicionarei um Blog no projeto (onde o **Next.js** se encaixa muito bem), dando dicas de como tirar o melhor proveito dos modelos de geração de imagem.

E como optei pelo **Next.js** nessa aplicação, aproveitei e escolhi outras tecnologias que funcionam muito bem em conjunto com o **Next.js** e em ambiente serverless da **Vercel**, como o **Upstash**.

O uso do **React** também está bem aproveitado, criando hooks e memoizando tudo que é possível com o uso de **useCallback**, **React.memo** e **useMemo**.

---

### Segurança

Toda a segurança da aplicação está sendo mantida pelo **Better Auth**, é ele que valida todas as rotas da aplicação e da API, a **Role** do usuário, e também controla a sessão. Todas as páginas privadas são validadas no lado do servidor, antes de sequer chegarem ao cliente.

A aplicação possui uma página **admin** (que modera usuários e valida prompts antes de ficarem públicos). Optei por fazer cada ação passar por uma **Server Action**, dificultando qualquer possível tentativa de exploração em um endpoint de admin. Todas as rotas, e principalmente cada ação de admin, são validadas por um **Rate Limit**, com base no caso de uso de um usuário comum e na importância da ação, por exemplo, ações muito críticas e que não são frequentemente repetidas em um curto intervalo de tempo, como a exclusão de conta.

### Banco de dados

Mantive a mesma preocupação com o **Banco de Dados**: nenhuma chamada ao **Supabase** é feita pelo client, todas são validadas e passam pelo servidor. O uso da **Anon Key** não está presente em lugar nenhum, mas mesmo assim mantive um modelo **"Zero Trust"**, ativando RLS e bloqueando requisições feitas via API REST com:

```sql
REVOKE ALL ON "NOME_DA_TABELA" FROM anon;
```

O uso do **Prisma** facilita muito nas queries da aplicação, pela sua praticidade e fácil uso de um ORM bem completo.

## Decisões técnicas relevantes

### Compressão de imagens

Um dos principais pontos da aplicação consiste em:

_Usuário autenticado publica um prompt + imagem_ **>** _validação dos dados no servidor_ **>** _salva imagem no storage e dados de texto no banco._

Durante o desenvolvimento eu notei um problema grande:

- O Supabase disponibiliza apenas **50MB** no plano Free (o qual eu utilizo no projeto)
- Cada imagem gerada pelo ChatGPT tem em média **2MB** e o Gemini muitas vezes tem o triplo do tamanho

Se eu deixasse assim, o limite seria estourado rapidamente.

Resolvi esse problema utilizando a biblioteca **Sharp**, convertendo cada imagem para **WebP** (o melhor formato para web hoje, por manter boa qualidade mesmo com um tamanho de arquivo menor), e comprimindo a imagem ao reduzir sua qualidade em 15%.

Resultado:

```
~3/4MB → ~120KB
```

Dessa forma, aumentei exponencialmente a capacidade de armazenamento da aplicação, fazendo com que o navegador não precise lidar com um volume grande de dados.

### Cache no lado do cliente

Com um alto volume de requisições em toda aplicação, optei pelo uso do **React Query** para cachear os dados no lado do cliente, minimizando ao máximo as requisições feitas ao banco de dados.