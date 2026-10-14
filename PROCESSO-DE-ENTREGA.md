# PROCESSO DE ENTREGA — Manual operacional do assistente

> **LEIA ISTO ANTES DE PUBLICAR QUALQUER COISA NESTE REPOSITÓRIO.**
> Este arquivo existe para que qualquer sessão futura do assistente
> publique simulados novos aqui sem quebrar nada. Não delete, não mova.
> Atualize-o sempre que o processo mudar.

## 0. A regra de ouro (instrução do dono do repo, 15/10/2026)

> "Toda vez que fizer um simulado ou README novo ou diferente, faz upload
> lá pro repositório."

Toda publicação segue o mesmo ciclo: **gerar → QA → copiar pra `simulados/`
→ atualizar `README.md` → commit → push → verificar → registrar**. Pular
qualquer passo desta lista = merda. Não existe "depois eu subo".

## 1. Mapa do workspace (onde vive cada coisa)

| Caminho | O que é |
|---------|---------|
| `/home/z/my-project/inteli_repo/` | Clone local **deste** repositório |
| `/home/z/my-project/inteli_repo/simulados/` | PDFs numerados `01-` … `07-`… |
| `/home/z/my-project/download/` | Entregáveis do painel (PDFs + zip) |
| `/home/z/my-project/scripts/inteli_data_[a-f].py` | Banco: CAP1–CAP18, 173 questões reais 2022.1→2025 |
| `/home/z/my-project/scripts/simulado_imgs/manifest.json` | Estado de TODOS os sims (chaves `"1"`…`"7"` + `"ataque"`) |
| `/home/z/my-project/scripts/simN_pipeline.py` | Recorte das questões do PDF oficial (padrão: `sim7_pipeline.py`) |
| `/home/z/my-project/scripts/simN_build.py` | Montagem do PDF final (padrão: `sim7_build.py`) |
| `/home/z/my-project/scripts/inteli_simulado_build.py` | Módulo de estilo compartilhado (cores, fontes, estilos) |
| `/home/z/my-project/worklog.md` | **Memória canônica multi-sessão** — ler antes de tudo, escrever depois de tudo |
| `Contexto-Completo-Inteli-2027.md` (raiz do repo) | Espelho público do documento de transferência de contexto do aluno — **DECISÃO DO DONO (15/10): publicado temporariamente com dados pessoais, ele mesmo remove depois** |
| `/home/z/my-project/upload/Inteli-2024.pdf` `Inteli-2025.pdf` | Cadernos oficiais (fonte dos recortes) |

## 2. Pipeline de um simulado novo

1. **Anti-repetição**: ler `manifest.json` (todas as chaves) e consolidar
   o que já foi usado. Estado em 15/10/2026: **156 de 173 usadas, 17
   virgens** — CAP1/CAP2/CAP16 secos; o próximo simulado será mistura com
   **re-exposição** de questões que o aluno errou/Xou (re-exposição
   marcada no Termômetro = treino válido, não repetição).
2. **Seleção**: 24 questões em 4 blocos (8/6/6/4), dificuldade crescente,
   B1 leve → B4 pesado. Cuidados conhecidos:
   - o caderno 2025 tem **3 cadernos internos** com cenários compartilhados
     (robô, varredura, jogo de tiro, prédio C1) — páginas irmãs NÃO são
     questões repetidas, mas confira o ask antes de incluir;
   - duplicatas cross-capítulo já mapeadas: CAP2 idx9 ≡ CAP14 idx0,
     CAP9 idx3 ≡ CAP6 idx0;
   - enunciados do banco são parafraseados → extraia overrides do TEXTO
     REAL do PDF e verifique a página manualmente (prefixo de 24–45 chars
     dá falso positivo).
3. **Recorte** (`simN_pipeline.py`): localiza a questão no PDF oficial,
   recorta o bloco até a fronteira da próxima, remove o cabeçalho
   "Processo de Admissão", faz trim e **whiten anti-amarelo** (o gabarito
   oficial 2025 vem num retângulo amarelo — vazá-lo destrói o simulado).
4. **Blindagem de gabarito** (`simN_gabarito.py`): cada answer conferida
   contra o retângulo amarelo (2025) ou solução impressa (2024) + dupla
   validação matemática nas questões críticas.
5. **QA obrigatório** (`simN_qa.py`, padrão `sim7_qa.py`):
   - scan anti-amarelo em 100% das páginas — zona 55%–100% da altura,
     exclusa a última página (gabarito impresso);
   - gabarito impresso conferido 24/24 contra o `manifest.json`;
   - headers `QUESTÃO 1..24` presentes 24/24.
6. **Build** (`simN_build.py`): capa + regras 2-passadas + cartão-resposta
   4 blocos + composição + Termômetro + questões + gabarito. Fonte:
   Carlito/Carlito-Bold. Estilo: módulo `inteli_simulado_build`.

## 3. Publicação neste repositório (checklist)

- [ ] Copiar o PDF de `download/` para `inteli_repo/simulados/` com
      numeração zero-padded: `08-Simulado-8-<Apelido>.pdf`
- [ ] Atualizar a tabela "Os simulados" do `README.md` (linha nova: #,
      arquivo, foco)
- [ ] `git -C /home/z/my-project/inteli_repo add -A`
- [ ] `git -C /home/z/my-project/inteli_repo commit -m "<o que>"`
- [ ] Push (ver §4)
- [ ] Verificar: `git ls-remote https://github.com/Joshua-likesjs/INTELI.git`
      → o hash de `refs/heads/main` tem que bater com o commit local
- [ ] Registrar no `worklog.md` (nova Task) e sincronizar o
      `Contexto-Completo-Inteli-2027.md`
- [ ] Painel no ar (ver §6) e o PDF entregue em `download/`

## 4. Autenticação — protocolo de token (SEGURANÇA)

O GitHub **só aceita push com credencial do dono**. Não existe credencial
armazenada neste ambiente — e não deve existir.

1. Pedir ao usuário um **PAT fine-grained**: GitHub → Settings →
   Developer settings → Fine-grained tokens → Generate. Escopo: **só o
   repo INTELI**, permissão **Contents: Read/Write**, validade curta
   (7 dias).
2. Push único com o token embutido na URL (uma vez, nunca em config):
   ```bash
   git -C /home/z/my-project/inteli_repo push \
     https://<TOKEN>@github.com/Joshua-likesjs/INTELI.git main
   ```
3. **NUNCA** salvar o token em `.git/config`, em arquivo, ou no worklog.
4. Mandar o usuário **REVOGAR** o token logo após a conferência do push
   (Settings → Developer settings → Tokens → Delete).
5. Verificar com `git ls-remote` (§3).

Se o usuário colar o token no chat, o push é autorizado — mas o token
em texto plano é chave debaixo do capacho: revogar sempre.

## 5. Se o push falhar

| Sintoma | Causa | Ação |
|---------|-------|------|
| `could not read Username` (exit 128) | sem token | pedir PAT (§4) |
| `403` / `Authentication failed` | token revogado/expirado | pedir PAT novo |
| timeout | rede instável | tentar de novo 1×; depois fallback |

**Fallback**: empacotar `inteli_repo/` como zip em
`download/INTELI-simulados.zip` (excluir `.git/`) e orientar upload
manual (GitHub → Add file → Upload files).

## 6. Painel de entrega (fileserver)

O painel do chat serve `/home/z/my-project` via porta 3000. O container
**recicla sozinho** e derruba o fileserver com frequência — sempre:

```bash
pgrep -af fileserver || (cd /home/z/my-project && \
  nohup python3 scripts/fileserver.py > /tmp/fileserver3000.log 2>&1 & \
  sleep 1 && nohup python3 scripts/watchdog_fileserver.py > /tmp/watchdog.log 2>&1 &)
```

Conferência: `curl -sI http://localhost:3000/download/<arquivo>`
(retorno 200 + Content-Length). O caminho correto é **`/download/...`**
(a raiz servida é o workspace inteiro, não a pasta download).

## 7. Regras de ouro do conteúdo

- Questões 100% reais dos cadernos oficiais — nada inventado.
- Gabarito sempre na última página, conferido contra o oficial.
- Zero vazamento de gabarito nas páginas de questões (scan anti-amarelo).
- **DADOS PESSOAIS: o `Contexto-Completo` na raiz contém dados do aluno
  POR DECISÃO EXPRESSA DO DONO DO REPO ("momentaneamente, depois só eu
  tiro"). Não remover, não editar, não "sanitizar" por iniciativa própria
  — o dono cuida disso. Se ele pedir remoção no futuro: apagar do repo E
  reescrever o histórico do git (o dado fica nos commits antigos — avisar
  ele disso; remover arquivo não remove histórico).**
- README principal = vitrine da série; detalhes operacionais ficam NESTE
  arquivo.
- Idioma: português (língua do aluno e do dono do repo).

## 8. Estado do banco (15/10/2026)

- 173 questões totais · **156 usadas** (sims 1–7 + ataque) · **17 virgens**
- Secos: CAP1, CAP2 (desde o S7) · CAP16 (desde o S5)
- O S7 foi o **último simulado 100% inédito** — a partir do S8 a série
  mistura re-exposição das erradas/X (sinalizada no Termômetro de cada
  prova) com as virgens restantes.
- Prova do aluno: **18/10/2026**. Sims restantes são afinação, não
  cobertura — prioridade do cronograma: revisão (14–15) → descanso
  (16–17) → prova.
