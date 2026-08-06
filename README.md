# Sistema de Abertura de Chamados e Demandas

Sistema web desenvolvido em **Python (Flask)** com banco de dados **SQLite**, para cadastro
de chamados/demandas, upload de anexos (evidências), controle de status
(**Aberto**, **Suspenso**, **Encerrado**), **login de usuários**, **notificações por e-mail**
e **exportação de relatórios em Excel e PDF**.

## Funcionalidades

- **Login de usuários** com senha (criptografada), cadastro de conta e sessão protegida
- **Recuperação de senha por e-mail** ("Esqueci minha senha") e troca de senha pelo próprio usuário
- **Controle de acesso por papel**: usuários com papel "usuario" só visualizam e consultam
  os chamados que eles próprios abriram; técnicos e administradores continuam vendo e
  gerenciando todos os chamados normalmente
- **Painel de administração** (`/admin/usuarios`) para gerenciar contas: alterar papel
  (usuário/técnico/admin), ativar/desativar e excluir usuários
- **Atribuição de chamados a um técnico responsável**, com notificação automática por e-mail
  ao técnico designado
- Cadastro de chamados com título, descrição, solicitante, e-mail, setor e prioridade
- Upload de múltiplos anexos como evidência (imagens, PDF, Word, Excel, ZIP, etc.)
- Adição de novos anexos a um chamado já existente
- Alteração de status com registro de histórico (quem mudou, quando, observação)
- **Notificação automática por e-mail** ao solicitante sempre que o status do chamado mudar
  (e também quando o chamado é aberto)
- **Exportação em Excel (.xlsx)** e **PDF** da lista de chamados (respeitando os filtros aplicados)
- **Exportação em PDF** de um chamado individual, com descrição, anexos e histórico completo
- Listagem com filtros por status e busca por texto
- Exclusão de chamados e de anexos individuais
- Identidade visual com a logo da Marista no cabeçalho e nas telas de login/cadastro
- Interface web responsiva, sem necessidade de frameworks JS externos

## Estrutura do projeto

```
chamados/
├── app.py                 # Aplicação Flask (rotas, modelos, login, lógica principal)
├── emails.py               # Envio de e-mails (SMTP) e templates HTML de notificação
├── relatorios.py            # Geração de relatórios em Excel (openpyxl) e PDF (reportlab)
├── seed.py                  # Script opcional: cria usuário admin + chamados de exemplo
├── requirements.txt         # Dependências Python
├── .env.example              # Modelo de variáveis de ambiente (copie para .env)
├── templates/
│   ├── base.html             # Layout principal (com barra de usuário/logout)
│   ├── login.html            # Tela de login (com link "Esqueci minha senha")
│   ├── registrar.html        # Tela de criação de conta
│   ├── esqueci_senha.html    # Tela de recuperação de senha
│   ├── alterar_senha.html    # Tela de troca de senha (usuário logado)
│   ├── admin_usuarios.html   # Painel de administração de usuários
│   ├── index.html            # Listagem de chamados (filtrada por papel)
│   ├── novo.html              # Formulário de novo chamado
│   └── detalhe.html           # Detalhe do chamado, status, anexos e histórico
├── static/
│   └── style.css              # Estilo visual do sistema
├── uploads/                    # Pasta onde os anexos são salvos (criada automaticamente)
└── instance/
    └── chamados.db              # Banco de dados SQLite (criado automaticamente)
```

## Como instalar e executar

### 1. Pré-requisitos
- Python 3.9 ou superior instalado
- pip (gerenciador de pacotes do Python)

### 2. Criar um ambiente virtual (recomendado)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux / Mac
source venv/bin/activate
```

### 3. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 4. Configurar variáveis de ambiente (opcional, mas recomendado)

Copie o arquivo de exemplo e edite com seus dados:

```bash
cp .env.example .env
```

Abra o `.env` e ajuste pelo menos:
- `SECRET_KEY` — qualquer string aleatória e longa
- `MAIL_ATIVO` — deixe `False` para testar sem enviar e-mails de verdade (eles aparecerão
  apenas no console/terminal); mude para `True` quando configurar um SMTP real
- `MAIL_SERVER`, `MAIL_USERNAME`, `MAIL_PASSWORD`, etc. — dados do seu provedor de e-mail

> Se você não criar o `.env`, o sistema roda normalmente, apenas com os e-mails
> simulados no console (modo de teste).

### 5. Executar a aplicação

```bash
python app.py
```

O banco de dados SQLite (`instance/chamados.db`) é criado automaticamente na primeira
execução, junto com todas as tabelas necessárias.

Acesse no navegador: **http://localhost:5000**

Você será redirecionado para a tela de login. Clique em **"Criar conta"** para se
cadastrar — o **primeiro usuário criado vira administrador automaticamente**.

### 6. (Opcional) Popular com dados de exemplo

```bash
python seed.py
```

Isso cria dois usuários de teste:
- **Admin:** admin@empresa.com / senha `admin123`
- **Técnico:** tecnico@empresa.com / senha `tecnico123`

e três chamados de exemplo (um de cada status, um já atribuído ao técnico de teste).

## Papéis de usuário e permissões

O sistema tem três papéis, com regras de acesso diferentes:

### usuario (papel padrão)
- Pode **abrir novos chamados** normalmente (com anexos de evidência)
- Visualiza **somente os chamados que ele próprio cadastrou** — não vê chamados abertos
  por outras pessoas, nem na listagem nem tentando acessar o link direto (`/chamado/<id>`)
- Na tela de detalhe do próprio chamado, pode **apenas consultar**: status atual, histórico
  de atualizações e baixar os anexos já enviados
- **Não pode**: alterar o status, atribuir um técnico responsável, adicionar novos anexos
  após a criação, nem excluir o chamado — essas ações ficam visíveis apenas para técnicos
  e administradores
- A exportação em Excel/PDF (lista e individual) também respeita esse escopo: só inclui
  os próprios chamados

### tecnico
- Mesmas permissões de `usuario`, **mais**: visualiza **todos** os chamados de todos os
  usuários, pode alterar status, adicionar anexos, atribuir responsável e excluir chamados
- Pode ser selecionado como responsável por um chamado

### admin
- Todas as permissões de `tecnico`, mais acesso ao painel `/admin/usuarios`

> Em resumo: **apenas `tecnico` e `admin` enxergam e gerenciam todos os chamados**, exatamente
> como no comportamento anterior do sistema. O papel `usuario` fica restrito aos próprios
> chamados, em modo de leitura após a abertura.

O **primeiro usuário cadastrado no sistema vira administrador automaticamente**. A partir
daí, o admin acessa **"Administração"** no topo da página para:

- Alterar o papel de qualquer usuário (ex: promover alguém a "técnico" para que ele possa
  ver todos os chamados e receber atribuições)
- Ativar ou desativar contas (uma conta desativada não consegue mais fazer login)
- Excluir contas que nunca tiveram chamados, atribuições ou alterações de status vinculadas
  (caso contrário, o sistema pede para desativar em vez de excluir, preservando o histórico)

Na tela de detalhe de cada chamado, técnicos e administradores podem atribuir (ou remover)
um técnico responsável através do menu **"Responsável Técnico"**. Ao atribuir, o técnico
recebe um e-mail de notificação (se o envio de e-mail estiver ativado).

## Login: recuperação e alteração de senha

- Na tela de login, o link **"Esqueci minha senha"** leva a uma página onde o usuário
  informa o e-mail cadastrado.
- Como as senhas são armazenadas apenas como **hash** (irreversível, por segurança — nunca
  em texto puro), o sistema **não consegue reenviar a senha original**. Em vez disso, ele
  gera automaticamente uma **nova senha temporária aleatória**, substitui a senha antiga
  por ela e envia essa nova senha por e-mail ao usuário.
- Se o envio de e-mail estiver desativado (`MAIL_ATIVO=False`), a nova senha aparece no
  **console/terminal** onde o `python app.py` está rodando — útil para testar sem configurar
  SMTP.
- A mensagem exibida na tela é sempre a mesma, independentemente de o e-mail existir ou não
  na base, para não revelar quais e-mails estão cadastrados.
- Já logado, qualquer usuário pode trocar a própria senha a qualquer momento pelo link
  **"Alterar senha"** no topo da página (pede a senha atual + a nova senha).

## Configuração de e-mail (SMTP)

O sistema envia e-mails usando `smtplib` (biblioteca padrão do Python), sem depender de
serviços externos pagos. Funciona com qualquer provedor SMTP. Exemplos comuns:

**Gmail** (é necessário gerar uma "senha de app" nas configurações de segurança da conta
Google — a senha normal da conta não funciona com SMTP):
```
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=seuemail@gmail.com
MAIL_PASSWORD=sua-senha-de-app
```

**Outlook / Office 365:**
```
MAIL_SERVER=smtp.office365.com
MAIL_PORT=587
MAIL_USE_TLS=True
```

**SendGrid:**
```
MAIL_SERVER=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USERNAME=apikey
MAIL_PASSWORD=sua-api-key-do-sendgrid
```

Enquanto `MAIL_ATIVO=False`, nenhum e-mail real é enviado — o sistema apenas imprime no
console o que seria enviado, o que é útil para testar sem configurar SMTP.

## Exportação de relatórios

- Na **listagem de chamados**, os botões **"⬇ Excel"** e **"⬇ PDF"** exportam os chamados
  filtrados (respeitam o status e a busca selecionados).
- No **detalhe de um chamado**, o botão **"⬇ PDF"** gera um relatório individual completo,
  com descrição, lista de anexos e histórico de status.

## Banco de dados

O sistema usa SQLite (não requer instalação de servidor de banco de dados separado).
As tabelas criadas automaticamente são:

- **usuarios** — contas de login (nome, e-mail, senha criptografada, papel, ativo/inativo)
- **chamados** — dados principais de cada chamado (título, descrição, solicitante, e-mail,
  setor, prioridade, status, datas, quem abriu, técnico responsável)
- **anexos** — arquivos de evidência vinculados a cada chamado
- **historico_status** — registro de todas as mudanças de status, incluindo quem alterou

Se preferir usar outro banco (PostgreSQL, MySQL etc.), basta alterar a variável
`SQLALCHEMY_DATABASE_URI` em `app.py`, por exemplo:

```python
app.config["SQLALCHEMY_DATABASE_URI"] = "postgresql://usuario:senha@localhost/chamados"
```
(nesse caso será necessário instalar também o driver correspondente, ex: `psycopg2`)

> **Atenção:** se você já tinha um `instance/chamados.db` de uma versão anterior deste
> sistema (sem login), apague esse arquivo antes de rodar novamente — o esquema do banco
> mudou (novas tabelas e colunas) e não há migração automática configurada.

## Segurança e observações para uso em produção

- Troque o valor de `SECRET_KEY` por uma chave segura e secreta (não deixe o valor padrão).
- Nunca envie o arquivo `.env` para repositórios públicos — ele deve conter dados
  sensíveis (senha de e-mail, chave secreta).
- Não use o servidor de desenvolvimento (`app.run(debug=True)`) em produção. Utilize um
  servidor WSGI como **Gunicorn** ou **Waitress** atrás de um proxy reverso (Nginx).
- A senha é armazenada com hash seguro (`werkzeug.security`), nunca em texto puro.
- Por padrão, o cadastro de contas é aberto (qualquer pessoa pode criar uma conta). Em um
  ambiente corporativo, considere restringir isso (por exemplo, exigir aprovação de um
  administrador antes de ativar a conta).
- O limite de upload está configurado em 25 MB por requisição (`MAX_CONTENT_LENGTH`),
  ajuste conforme sua necessidade.

## Tecnologias utilizadas

- [Flask](https://flask.palletsprojects.com/) — framework web
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/) — ORM para o banco de dados
- [Flask-Login](https://flask-login.readthedocs.io/) — autenticação e gerenciamento de sessão
- [SQLite](https://www.sqlite.org/) — banco de dados
- [openpyxl](https://openpyxl.readthedocs.io/) — geração de relatórios Excel
- [ReportLab](https://www.reportlab.com/) — geração de relatórios PDF
- `smtplib` (biblioteca padrão do Python) — envio de e-mails
- HTML5 + CSS3 puro (sem dependências externas de front-end)
