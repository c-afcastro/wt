# wirteilen — guia de instalação

App web para dividir despesas entre amigos. Cada um usa no próprio iPhone (ou Android, ou computador), todos lendo e gravando na mesma base online.

**Como funciona por baixo**

| Peça | Serviço | Custo |
|---|---|---|
| Site do app | GitHub Pages | grátis |
| Login, senha e recuperação por e-mail | Firebase Authentication | grátis |
| Base de dados única do grupo | Firebase Firestore (servidor em São Paulo) | grátis até 50 mil leituras e 20 mil gravações por dia |
| Funcionar sem internet | cache do Firestore + `sw.js` | — |

Por que Firebase e não Google Drive ou SharePoint: planilhas e arquivos no Drive não têm login por usuário, não aguentam duas pessoas gravando ao mesmo tempo e exigem que cada amigo autorize o app na conta Google dele. O Firebase resolve login, recuperação de senha, sincronização em tempo real e modo offline de uma vez. O Drive continua útil para guardar os **backups** (ver item 8).

## Arquivos da pasta

```
index.html            o app inteiro (telas, cálculos, Pix, backup)
firebase-config.js    onde você cola a chave do seu projeto Firebase
firestore.rules       regras de segurança da base (colar no Firebase)
sw.js                 permite abrir o app sem internet
manifest.webmanifest  nome e ícone ao instalar no celular
icons/                ícones do app
```

Sem configurar nada, o app abre em **modo demonstração**: funciona, mas os dados ficam só naquele aparelho.

---

## 1. Criar o projeto no Firebase (10 min)

1. Acesse **console.firebase.google.com** com sua conta Google.
2. **Criar um projeto** → nome `wirteilen` (ou outro) → pode desativar o Google Analytics → Criar.

## 2. Ligar o login por e-mail e senha

1. Menu lateral **Security (Segurança) → Authentication** → **Vamos começar**.
2. Aba **Método de login** → **E-mail/senha** → ativar a primeira chave → Salvar.
3. Aba **Modelos** → ícone de lápis em qualquer modelo → no rodapé, **Idioma do modelo: Português (Brasil)**. Assim o e-mail de recuperação de senha chega em português.

## 3. Criar a base de dados

1. Menu lateral **Databases & Storage (Bancos de dados e armazenamento) → Firestore** → **Criar banco de dados**. Se perguntar a edição, escolha **Standard**.
2. Local: **southamerica-east1 (São Paulo)**. Modo: **produção**.
3. Depois de criado, aba **Regras** → apague tudo → cole o conteúdo do arquivo `firestore.rules` → **Publicar**.

## 4. Conectar o app ao projeto

1. Ícone de engrenagem (ao lado de "Visão geral do projeto") → **Configurações do projeto**.
2. Em **Seus apps**, clique no ícone **`</>`** (Web) → apelido `wirteilen` → Registrar app (não marque Hosting).
3. Aparece um bloco `const firebaseConfig = { apiKey: "...", ... }`.
4. Abra `firebase-config.js` num editor de texto e copie cada valor para o campo correspondente. Exemplo:

```js
window.FIREBASE_CONFIG = {
  apiKey: "AIzaSyA...",
  authDomain: "wirteilen-1234.firebaseapp.com",
  projectId: "wirteilen-1234",
  storageBucket: "wirteilen-1234.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
};
```

A `apiKey` do Firebase é pública por natureza (vai para o navegador de qualquer forma). Quem protege os dados são o login e as regras do item 3.

## 5. Publicar no GitHub Pages (10 min)

1. Crie uma conta em **github.com** (se ainda não tiver).
2. **New repository** → nome `wirteilen` → **Public** → Create repository.
3. Na página do repositório: **Add file → Upload files** → arraste **todo o conteúdo da pasta** (incluindo a pasta `icons`) → **Commit changes**.
4. **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main` / pasta `/ (root)` → Save.
5. Em 1 a 2 minutos o endereço aparece no topo da página: `https://SEU-USUARIO.github.io/wirteilen/`.

Por último, volte ao Firebase: **Security → Authentication → Configurações → Domínios autorizados → Adicionar domínio** → `SEU-USUARIO.github.io` (sem `https://` e sem `/wirteilen`).

## 6. Instalar no iPhone

1. Abra o endereço no **Safari** (precisa ser o Safari).
2. Botão **Compartilhar** → **Adicionar à Tela de Início** → Adicionar.
3. O ícone do wirteilen aparece como um app, em tela cheia.

## 7. Usar com os amigos

1. Crie sua conta no app e crie um grupo (ex.: "Viagem Ilhabela").
2. **Ajustes → Copiar convite** e cole no WhatsApp. O texto leva o link e o código de 6 letras.
3. Cada amigo abre o link, cria a conta dele e entra direto no grupo.
4. Cada um preenche **Ajustes → Meu perfil e Pix** com a própria chave.

No dia a dia:

- **Despesas**: botão **+** no topo, à direita do nome do grupo, para lançar. Divisão **Igual**, por **Valores** exatos ou por **Partes** (ex.: um casal conta 2 partes).
- **Saldos**: quanto cada um tem a receber ou pagar e os **acertos sugeridos** (o menor número de Pix para zerar o grupo). O botão de Pix mostra o **QR Code** e o **Pix copia e cola já com o valor**. Depois de pagar, toque em **Registrar pagamento**.
- **Resumo**: total por mês, por categoria e quanto cada um pagou e consumiu.
- **Manual**: passo a passo guiado de começar a usar, lançar e acertar com Pix, backup e recuperação de senha, mais uma consulta rápida com busca.

## 8. Backup

Em **Ajustes → Backup**:

- **Exportar backup (.json)**: cópia completa do grupo. No iPhone abre a folha de compartilhar; escolha **Salvar em Arquivos** ou o app do **Google Drive**.
- **Planilha (.csv)**: abre no Excel, com separador `;` e vírgula decimal, uma coluna com a parte de cada pessoa.
- **Copiar backup**: copia o conteúdo para colar num e-mail ou nota.
- **Restaurar**: escolha o arquivo `.json` (ou cole o conteúdo). Funciona com o app online.
  - **Mesclar** repõe o que veio no backup sem apagar o resto.
  - **Substituir tudo** (só quem criou o grupo) deixa o grupo idêntico ao backup.
  - Se o backup tiver pessoas que não estão no grupo atual, o app pergunta a quem cada uma corresponde. Isso permite levar um grupo para outro projeto ou recriá-lo do zero.
- **Cópias automáticas**: o app guarda no próprio aparelho até 8 versões do grupo, criadas sozinhas sempre que os dados mudam.

## 9. Senha

- **Esqueci minha senha** na tela de entrada envia um link para o e-mail. Pelo link a pessoa cria uma senha nova.
- Logado, **Ajustes → Trocar senha por e-mail** faz o mesmo.
- Confira a caixa de spam no primeiro envio.

## 10. Sem internet

O app abre e funciona offline depois do primeiro acesso. Despesas lançadas sem conexão ficam guardadas no aparelho e sobem sozinhas quando a internet volta (o indicador no topo mostra **Offline** ou **Enviando…**). Entrar num grupo por código e restaurar backup precisam de internet.

## 11. Atualizar o app depois

1. Suba o `index.html` novo no GitHub (Add file → Upload files, substitui o antigo).
2. Em `sw.js`, troque `wirteilen-v2` por `wirteilen-v3` (e assim por diante) e suba também.
3. Os celulares pegam a versão nova na próxima abertura com internet.

## Problemas comuns

| Sintoma | Causa provável |
|---|---|
| Tela de entrada diz que o app não está ligado a uma base | `firebase-config.js` vazio ou não enviado ao GitHub |
| "Sem permissão para essa ação" | Regras do item 3 não publicadas |
| E-mail de recuperação não chega | Spam, ou modelo de e-mail sem idioma configurado |
| Login não funciona só no site publicado | Domínio `github.io` não autorizado (fim do item 5) |
| App no iPhone mostra versão antiga | Feche e abra de novo com internet; confira a troca de versão em `sw.js` |
