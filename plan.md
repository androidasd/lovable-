# Plano de implementação — SIGDE

## Escopo e decisões técnicas

Construir o SIGDE operacional em português brasileiro conforme a especificação e o PDF enviados. Por orientação do usuário, não usar Supabase nem login OAuth Manus: autenticação local por e-mail e senha sobre o MySQL gerenciado do projeto, com Drizzle e migrações versionadas. Senhas somente em hashes Scrypt com salt aleatório; limitação de tentativas persistida em chaves HMAC opacas; cookies próprios e sessões assinadas. Incrementar `sessionVersion` ao vincular uma conta legada ao login local, trocar senha ou redefini-la, invalidando sessões anteriores. A primeira conta Administradora é configurada uma vez em tela de bootstrap protegida por `SIGDE_BOOTSTRAP_TOKEN`; o usuário escolhe e digita suas próprias credenciais no Preview HTTPS. Não há cadastro público nem redefinição por e-mail.

Não inserir alunos ou declarações de demonstração como se fossem reais. A migration adiciona apenas o registro oficial da escola descrito no PDF, necessário para preservar a conta existente e impor a FK institucional; não sobrescreve configurações futuras. Usar a logomarca fornecida no PDF; não copiar nem inventar a assinatura exibida na referência. O bloco de assinatura fica sem imagem até a instituição fornecer arquivo autorizado.

## Implementação

- **Frontend:** React/TypeScript e componentes acessíveis do starter. Dashboard, cadastro, emissão, histórico, configurações, modelos, usuários/permissões e troca de senha. Login e logout locais; primeira configuração até concluir o bootstrap.
- **Backend:** Express/tRPC. `server/authRouter.ts` e `auth-credentials.ts` autenticam, criam a primeira conta e gerenciam senhas; cookie `sigde_session` assinado com `MANUS_JWT_SECRET`. Não confiar no cookie `webdev_app_session`, em OAuth ou no login automático do Preview. Administrador cria contas, redefine senhas e atribui papéis; cada usuário altera a própria senha. Leitura requer perfil atribuído; escrita de alunos/declarações requer Administrador ou Secretaria; configurações/modelos/permissões/auditoria são restritos ao Administrador. Mutações verificam origem e token CSRF de dupla submissão.
- **Bootstrap:** exigir `SIGDE_BOOTSTRAP_TOKEN` recebido pelo formulário seguro de segredo; comparação em tempo constante, limite persistente por IP e uso único. Criar a primeira conta e registrar o estado do bootstrap na mesma transação; criar modelo padrão somente se ainda não houver um modelo ativo. A escola é inserida pela migration com valores do PDF, e o bootstrap não sobrescreve configurações existentes.
- **Banco:** MySQL/Drizzle. `localCredentials` guarda usuário, e-mail normalizado único e hash; `authBootstrap` controla a primeira conta; `authLoginAttempts` mantém rate limits HMAC. Tabelas relacionam instituição, usuários/perfis, alunos, matrículas anuais, turmas, modelos versionados, declarações com snapshot, sequência anual e auditoria. FK e índice vinculam usuário à instituição; unicidade de turma por instituição/ano/série/turma/turno e índices próprios para as FKs.
- **Alunos:** CRUD, pesquisa por nome/matrícula, filtros por série/turma/turno/situação e importação XLSX/CSV com prévia, validação, alertas de duplicidade e inativação confirmada. Datas passam validação calendária.
- **Declarações:** selecionar aluno com matrícula ativa correspondente ao ano/série/turma/turno/modalidade; servidor rejeita divergências e exige nome de mãe ou responsável. Modelos inativos não podem ser usados pela API. Se os dados forem editados, desvincular explicitamente e emitir como manual. Número anual confirmado transacionalmente apenas para controle e histórico; não é impresso na declaração. Snapshot da instituição/aluno/modelo imutável. Histórico com filtros, segunda via e cancelamento justificado; cópia de declaração cancelada mantém a marca `CANCELADA — SEM VALIDADE` no preview e no PDF.
- **PDF:** `@react-pdf/renderer` gera PDF vetorial/selecionável em tamanho A4 (595,28 × 841,89 pt), conforme solicitado; fontes compatíveis com Times New Roman e Arial são incorporadas. PDF.js renderiza em canvas os mesmos bytes na pré-visualização, sem depender do visualizador PDF embutido do Chrome. Reproduzir somente o cabeçalho, os dados, a redação e o bloco de assinatura/identificação já presentes no PDF de referência; sem número de emissão, rodapé ou texto de pré-visualização no documento normal. O administrador pode substituir a assinatura do modelo por outra imagem institucional autorizada. Download, impressão, zoom, ajuste e tela cheia; snapshots e configurações persistem no MySQL e são acessados por APIs autenticadas.
- **Qualidade/entrega:** manter no Preview até pedido expresso para publicar. TypeScript, testes, build e inspeção de rotas; não escrever/apagar alunos ou declarações reais para testar.

## Estrutura do projeto

- `client/src/pages/Home.tsx` — seleção de telas e estado de login.
- `client/src/pages/sigde/` — `Shell.tsx`, `DashboardPage.tsx`, `StudentsPage.tsx`, `DeclarationPage.tsx`, `DeclarationsPage.tsx`, `SettingsPage.tsx`, `TemplatesPage.tsx`, `UsersPage.tsx`, `AccountPage.tsx` e helpers compartilhados.
- `client/src/lib/declaration-pdf.tsx` — modelo A4, fontes incorporadas, assinatura do modelo, preview em canvas, download e impressão.
- `client/public/manus-routes.json` e `client/public/assets/` — manifesto, logomarca e bloco de assinatura extraídos do PDF de referência.
- `client/public/assets/fonts/` — fontes abertas compatíveis com Arial e Times New Roman, com licença incluída.
- `server/authRouter.ts`, `server/auth-credentials.ts` — endpoints de autenticação, hash, rate limit e sessão.
- `server/_core/csrf.ts` — verificação de origem nas mutações.
- `server/sigdeRouter.ts` — APIs SIGDE, validação, autorização, matrículas, documentos e auditoria.
- `server/_core/` — cookies, assinatura e validação de sessão; callbacks OAuth não são registrados.
- `drizzle/schema.ts`, `drizzle/` — esquema e migrações MySQL.

## Design

- **Movimento:** administração pública contemporânea, sobriedade editorial e clareza de software institucional.
- **Princípios:** precisão documental; navegação previsível; hierarquia calma; acessibilidade e confiança.
- **Filosofia de cor:** azul-marinho transmite autoridade; branco preserva leitura; cinzas frios estruturam áreas administrativas; azul vivo destaca ações sem competir com o documento.
- **Layout:** shell de secretaria com navegação lateral no desktop e recolhida no celular; emissão com formulário e folha lado a lado no desktop e abas no celular.
- **Elementos de assinatura:** divisórias finas, etiquetas documentais discretas e cartões compactos.
- **Interação/animação:** feedback imediato, confirmação de ações destrutivas, movimento funcional breve e respeito a `prefers-reduced-motion`.
- **Tipografia:** sans-serif legível na interface e serifada tradicional no documento.
- **Essência da marca:** ferramenta segura para emitir documentos escolares corretos com menos retrabalho; precisa, sóbria e confiável.
- **Voz:** objetiva e respeitosa. Exemplos: “Confira os dados antes de emitir.” / “Nenhum aluno cadastrado. Comece pelo cadastro da secretaria.”
- **Wordmark/logo:** wordmark SIGDE com símbolo abstrato de folha/documento, sem imitar selo oficial; usar apenas o brasão fornecido para a instituição.
- **Cor de assinatura:** azul-marinho institucional.
