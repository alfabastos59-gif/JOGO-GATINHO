# Missão Quitério – Desafio Literário

Jogo de perguntas da **Biblioteca Maria Quitéria** (Colégio Estadual do Campo Maria Quitéria).
O aluno entra com o **código de aluno**, guia o gatinho pela fazenda até a escola e, na biblioteca,
responde perguntas sobre os **livros que pegou emprestado e devolveu**.

## Como abrir
Dê dois cliques em `index.html` (funciona direto no navegador, sem instalar nada).
Todo o jogo está nesse único arquivo: código, imagens do gatinho, sons e dados.

## Controles
- Setas ou WASD: mover · Shift: correr · Espaço/↑: pular · Z: patada · ↓: rolar
- Mouse ou toque: o gatinho vai até o ponto clicado · duplo clique: patada
- Na biblioteca: leve o gatinho até o cartão e espere o anel dourado encher (ou aperte Espaço)

## Como funciona
1. **Login:** código de aluno (guardado só como hash SHA-256 no jogo).
2. **Fazenda:** corrida até a escola, coletando leite (+10) e ração (+5) e se defendendo dos cachorros.
3. **Biblioteca:** um cartão para cada livro devolvido (até 5) + "Desafio Surpresa".
   **Só joga quem pegou um livro emprestado e já devolveu:** no login, quem não devolveu nenhum livro
   recebe um aviso (com o título do livro pendente, se houver) e não entra no jogo.
4. **Perguntas** geradas a partir do cadastro: autor, título, categoria, ano, editora,
   trecho da sinopse e dados do autor (tabela `autores`).
5. **Pontos e desafios concluídos** ficam salvos no navegador do aluno
   (e no Supabase, se ligado – veja abaixo).

## Atualizar alunos e livros
Gere um novo backup no aplicativo da biblioteca e rode:

```bash
python tools/atualizar_dados.py backup_biblioteca_maria_quiteria_AAAA-MM-DD.json index.html
```

O script troca apenas o bloco `const DATA=` do `index.html`.
**Não entram** no jogo: e-mails, telefones, PINs de administradores e auditoria.
O **avatar** do aluno entra no campo `a` de cada item de `DATA.students` (URL da imagem, `data:image/...` ou emoji);
se faltar, o ranking mostra a inicial do nome em um círculo colorido.
> Atenção: o script `tools/atualizar_dados.py` original descarta os avatares; ele precisa ser ajustado para gravar o campo `a`.

## Ligar ao Supabase (pontos / ranking)
1. No Supabase, rode `supabase/missao_progresso.sql` (SQL Editor).
2. No `index.html`, procure `const CFG=` e preencha:
   `enabled:true`, `url:'https://SEU-PROJETO.supabase.co'`, `key:'SUA_CHAVE_ANON'`.
3. O jogo passa a ler e gravar `pontos` e `concluidos` por `codigo_aluno`
   (use a tabela no Ranking do site).

## Segurança e privacidade
- **Mantenha este repositório PRIVADO.** O `index.html` contém primeiro nome dos alunos,
  hash dos códigos e quais livros cada um devolveu.
- Nunca suba o arquivo de backup (`backup_*.json`): ele contém PIN de administrador em texto puro.
  O `.gitignore` já bloqueia esses arquivos.
- A chave `anon` do Supabase é pública por natureza. Para impedir que alguém grave pontos de
  outro aluno, troque as políticas abertas do SQL por uma função (RPC) que valide o código
  na tabela de alunos.

## Ranking
Botão **🏆 Ranking** na tela de escolha de desafios e no cartão de fim de desafio.
Mostra, em ordem de pontos, só os alunos que já devolveram livro e pontuaram: posição (🥇🥈🥉),
avatar do cadastro, primeiro nome, desafios concluídos, estrelas e pontos. O aluno logado aparece destacado.
- **Sem Supabase** (padrão): o ranking reúne os alunos que jogaram **naquele aparelho** (progresso guardado no navegador).
- **Com Supabase ligado:** o ranking lê a tabela `missao_progresso` e reúne os alunos de todos os aparelhos.
