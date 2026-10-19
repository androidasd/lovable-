# Entregas do SIGDE

## Interface institucional, navegação e dashboard
- [x] Interface profissional, moderna, minimalista, em português brasileiro; azul-marinho, branco, cinza-claro e detalhes azuis; tipografia legível, ícones profissionais, transições discretas, layout responsivo para computador/tablet/celular, navegação por teclado e acessibilidade.
- [x] Menu com Dashboard; Cadastro de Alunos; Emitir Declaração; Declarações Emitidas; Modelos de Documentos; Configurações da Escola; Usuários e Permissões.
- [x] Dashboard mostra total de alunos, total de declarações, declarações emitidas no mês, alunos por série, histórico das últimas declarações, ação rápida Nova Declaração e pesquisa rápida por nome ou matrícula.
- [x] Indicadores refletem registros reais; estado inicial vazio apresenta zeros, sem dados fictícios como se fossem reais.
- [x] Notificações claras de sucesso/erro, carregamento e confirmação de operações; controles decorativos sem ação foram removidos.

## Cadastro, pesquisa e importação de alunos
- [x] Cadastrar e editar nome completo, data de nascimento, nome da mãe, pai opcional, responsável legal opcional, matrícula, ano letivo, série/ano, turma, turno (manhã/tarde/noite/integral), modalidade de ensino e situação da matrícula.
- [x] Pesquisar por nome ou matrícula e filtrar por série, turma, turno e situação.
- [x] Importar alunos em lote por Excel ou CSV com pré-visualização e validação antes de confirmar; detectar possíveis cadastros duplicados e identificar erros por linha; não presumir turno inválido/ausente, validar situação informada e usar ATIVA somente quando a coluna de situação estiver em branco. A importação e seu evento de auditoria são atômicos.
- [x] Inativar cadastro mediante confirmação; persistir dados com controles de segurança e permissões.

## Emissão e pré-visualização do documento
- [x] Buscar/selecionar aluno cadastrado com preenchimento automático de nome, nascimento, filiação, ano letivo, série, turma e turno; permitir preencher manualmente todos os campos para aluno não cadastrado.
- [x] Formulário inclui dados do aluno (nome, nascimento, mãe ou responsável, série, turma, turno, ano letivo) e do documento (número, data de emissão, nome e cargo do signatário); o servidor também rejeita emissão sem nome de mãe ou responsável legal.
- [x] Data de emissão inicia na data atual e pode ser editada. Número da declaração é sequencial por ano sem duplicidade, confirmado em transação no MySQL. A tela mostra a previsão; impressão e download só são habilitados depois de salvar e confirmar o número definitivo.
- [x] Preview atualiza ao modificar campos; no desktop exibe formulário e documento lado a lado; no celular oferece abas adaptadas; inclui zoom, ajuste à página e tela cheia.
- [x] Se dados pessoais/escolares forem editados depois de selecionar um aluno, o vínculo é removido com aviso e a emissão passa a ser manual. No servidor, `studentId` exige cadastro ativo e matrícula ativa correspondente; divergência entre formulário, aluno e matrícula é rejeitada.
- [x] Documento deriva do PDF fornecido, recriado digitalmente (não usa fotografia como fundo): logomarca do Governo do Pará; cabeçalho GOVERNO DO ESTADO DO PARÁ, SECRETARIA EXECUTIVA DE EDUCAÇÃO, 10ª UNIDADE REGIONAL DE EDUCAÇÃO e COLÉGIO ESTADUAL DE ENSINO MÉDIO ROSA ALVAREZ REBELO; endereço Rua Sete de Setembro, nº 576 – bairro: Centro; Município Senador José Porfírio, Estado Pará; telefone (091) 3556–1397; processo de autorização Res. nº 267 de 26/05/2024–CEE/PA; Código INEP 15178137.
- [x] Título centralizado e sublinhado DECLARAÇÃO DE ESCOLARIDADE; texto formal segue a redação do modelo: “Declaramos para todos os fins de direito e a quem possa interessar que o (a) aluno (a) [NOME COMPLETO], nascido (a) em: [DATA DE NASCIMENTO], filho (a) de: [NOME DA MÃE OU RESPONSÁVEL], matriculou-se neste Estabelecimento de Ensino, no ano letivo de [ANO LETIVO], para cursar a [SÉRIE] do Ensino Médio (SOME REGULAR), na turma [TURMA], no turno da [TURNO], frequentando regularmente as aulas. Por ser verdade, firmo a presente declaração para que surta efeitos legais.”
- [x] Filiação permite compatibilizar o exemplo visual que contém mãe e pai com a regra de mãe ou responsável.
- [x] Modalidade da matrícula é substituída no texto-base; o modelo de referência mantém a redação original “Ensino Médio (SOME REGULAR)”.
- [x] Exibir local/data “Senador José Porfírio/PA, [DIA] de [MÊS] de [ANO]” e bloco de assinatura/identificação do responsável autorizado.
- [x] Não inventar assinaturas ou carimbos; reproduzir a assinatura com sua identificação impressa exatamente como aparece no PDF de referência. Administrador pode substituir o bloco por uma versão institucional autorizada.
- [x] Administrador pode editar o texto e salvar nova versão sem modificar declarações emitidas anteriormente; o servidor só aceita o modelo ativo escolhido para novas emissões.

## PDF, impressão e armazenamento de declarações
- [x] Gerar PDF vertical em tamanho A4 (595,28 × 841,89 pt), com texto selecionável/pesquisável, acentos corretos, margens institucionais, nitidez, fontes incorporadas compatíveis com Times New Roman e Arial e sem sobreposição.
- [x] O PDF normal contém somente os elementos e dados previstos no modelo; número de emissão fica no histórico interno e não aparece no papel, nem há rodapé ou texto “pré-visualização”. Preview e PDF final derivam do mesmo documento A4 gerado por `@react-pdf/renderer`; PDF.js rasteriza os mesmos bytes em canvas para evitar bloqueio do visualizador embutido do Chrome. Controles de impressão, download, salvar declaração e nova declaração são funcionais.
- [x] Nomear arquivo automaticamente, por exemplo DECLARACAO_ESCOLARIDADE_MARIA_SILVA_2026.pdf; controles da interface não aparecem no PDF.
- [x] Salvar declaração e snapshot dos dados usados na emissão, mesmo que aluno/modelo mude depois.
- [x] Conteúdo de declaração e ativos institucionais são acessados por procedimentos autenticados; segunda via preserva snapshot original.

## Histórico, auditoria e relatórios
- [x] Consultar declarações com número, aluno, série, data, usuário responsável e status; pesquisar por período, aluno ou número, inclusive dados do snapshot de uma emissão manual.
- [x] Visualizar, baixar novamente, imprimir novamente e cancelar com justificativa. Toda segunda via de documento cancelado mostra no preview e no PDF a marca “CANCELADA — SEM VALIDADE”.
- [x] Consultar histórico de alterações e registrar ações relevantes com ator e data; registrar eventos junto às alterações na mesma transação.
- [x] Exportar o histórico em CSV com os filtros selecionados.

## Configurações institucionais, modelos e permissões
- [x] Configurações administrativas: nome da escola, governo/secretaria, unidade regional, endereço, município/estado, telefone, código INEP, ato de autorização, logomarca, nome/cargo do responsável e assinatura digitalizada autorizada.
- [x] Dados institucionais alimentam automaticamente os documentos e são preservados no snapshot.
- [x] Administrador pode criar contas locais por nome/e-mail/senha e escolher papel; pode definir ou redefinir senhas sem expor hashes. Usuário pode alterar sua própria senha; não há envio de redefinição por e-mail.
- [x] Administrador gerencia usuários, configurações, modelos, histórico e auditoria; Secretaria cadastra/edita alunos, emite declarações e consulta histórico; Consulta acessa dados/documentos por rotas autenticadas.
- [x] Contas locais sem perfil ficam sem acesso a dados escolares e são identificadas como “Aguardando autorização”; somente Administrador lhes atribui Consulta, Secretaria ou Administrador.
- [x] Alterações de acesso e operações relevantes são registradas; permissões validadas no servidor.

## Autenticação, MySQL, segurança e LGPD
- [x] Login e logout do SIGDE usam e-mail e senha, não OAuth Manus. Hash de senha com salt (Scrypt), validação no servidor, mensagens genéricas, limites de tentativas persistidos em HMAC por IP/e-mail, sessão assinada com expiração e cookie próprio `sigde_session`; bootstrap de conta legada, troca e redefinição de senha incrementam `sessionVersion` para invalidar sessões anteriores; mutações tRPC verificam a origem contra CSRF. Não aceitar o auto-login OAuth do Preview nem implementar bypass de desenvolvimento.
- [x] Bootstrap de Administrador é de uso único e protegido por `SIGDE_BOOTSTRAP_TOKEN`; a tela principal de login solicita somente e-mail e senha, enquanto a configuração do primeiro Administrador fica em um fluxo separado que solicita token, nome, e-mail, senha e confirmação no Preview HTTPS. A rota se desativa após o bootstrap.
- [x] Usar MySQL gerenciado já provisionado no projeto, via Drizzle ORM e migrações versionadas; não usar nem solicitar Supabase/PostgreSQL externo.
- [x] Banco relacional acrescenta credenciais locais (hash + e-mail normalizado único), bloqueio de bootstrap de uso único e tabela de rate limit HMAC; `users.institutionId` possui FK e índice para `institutions.id`. Migration inclui somente o registro oficial da escola com valores fornecidos no PDF, preservando a conta legada sem apagar dados.
- [x] Papéis Administrador, Secretaria e Consulta; validação de permissões no servidor, controle de acesso aos dados documentais e trilha de auditoria.
- [x] Considerar LGPD e dados de estudantes possivelmente menores; minimizar dados e não gravar conteúdo pessoal em logs.
- [x] Fluxos de interface e API implementados para cadastro, pesquisa, preenchimento, pré-visualização, emissão, armazenamento e download de PDF; sem registros fictícios.

## Pronto para uso e qualidade
- [x] Erros em linguagem clara; formulários validados e estados de carregamento; sem dados fictícios apresentados como reais.
- [x] Aplicação permanece em Preview até solicitação expressa de publicação.
- [x] `pnpm check`, suíte de testes (21 testes), `pnpm build`, `git diff --check` e `drizzle-kit generate` passam; nenhum schema change está pendente. Migrações aditivas 0003–0006 aplicadas; FKs de matrículas preservadas e nenhuma conta, aluno ou declaração criada para testes, pois a base gerenciada é compartilhada.
- [ ] Provisionar a primeira conta Administradora: `SIGDE_BOOTSTRAP_TOKEN` está configurado no projeto; a consulta `auth.bootstrapStatus` confirma `needsSetup=true` e `tokenConfigured=true`. Ainda falta usar “Configurar primeiro Administrador” no Preview e criar a conta com nome, e-mail e senha; após o bootstrap, remover o segredo temporário das configurações.
