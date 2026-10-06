# Git

## 1. Definição

O **Git** é um sistema de controle de versão distribuído criado para rastrear alterações em arquivos ao longo do tempo. Seu uso é predominante no desenvolvimento de software, mas não se limita a esse domínio.

Diferentemente de sistemas centralizados, o Git adota um modelo distribuído: cada cópia do repositório contém o histórico completo do projeto. Isso significa que operações como commit, comparação de versões e criação de branches são executadas localmente, sem necessidade imediata de um servidor remoto.

Internamente, o Git trabalha com snapshots do estado do projeto, armazenados como objetos imutáveis identificados por hash criptográfico. Essa abordagem garante integridade e rastreabilidade do histórico.

---
## 2. Problemas que o Git resolve

O Git foi projetado para solucionar limitações encontradas em sistemas anteriores de versionamento.

### Controle de histórico

Permite manter registro preciso de todas as modificações realizadas em um conjunto de arquivos, incluindo autoria, data e descrição da mudança. Isso resolve problemas como perda de versões anteriores e ausência de auditoria.

### Colaboração paralela

Diversos desenvolvedores podem trabalhar simultaneamente no mesmo projeto, criando branches independentes e integrando alterações posteriormente. Isso reduz conflitos e elimina a necessidade de bloquear arquivos.

### Experimentação segura

A criação de branches é extremamente leve e rápida. Isso permite testar funcionalidades isoladamente sem comprometer a base principal do código.

### Integridade de dados

Cada objeto no Git é identificado por um hash (SHA-1, com evolução para SHA-256). Caso qualquer dado seja alterado, o hash correspondente muda, permitindo detectar corrupção ou adulteração.

### Trabalho offline

Como o histórico completo está presente localmente, é possível trabalhar, versionar e inspecionar alterações mesmo sem conexão com a internet.

---

## 3. Contexto histórico

O Git foi criado em 2005 por **Linus Torvalds**, criador do **Linux**.

Na época, o desenvolvimento do kernel Linux utilizava o sistema proprietário **BitKeeper**. Após divergências relacionadas à licença de uso, o acesso gratuito ao BitKeeper foi encerrado para a comunidade Linux.

Diante da necessidade de um sistema que suportasse:

- Alto volume de contribuidores
    
- Grande quantidade de arquivos
    
- Performance elevada
    
- Integridade garantida
    
- Modelo distribuído
    

Torvalds desenvolveu o Git em poucas semanas, com foco primário em desempenho, simplicidade estrutural e robustez.

Desde então, o Git tornou-se o padrão dominante da indústria.

---
## 4. Fluxo interno do Git

O funcionamento do **Git** pode ser entendido a partir de três áreas principais pelas quais as alterações passam durante o processo de versionamento.

Essas áreas representam diferentes estados dos arquivos dentro do repositório.

```
Working Directory  →  Staging Area  →  Local Repository  →  Remote Repository
```

---

# 1. Working Directory

O **Working Directory** é o diretório do projeto no sistema de arquivos.

Ele contém:

- arquivos que estão sendo editados
    
- arquivos novos
    
- arquivos modificados
    
- arquivos ainda não versionados
    

Qualquer alteração feita no código ocorre inicialmente nessa área.

Exemplo de situação:

```
arquivo.py modificado
novo_script.py criado
config.yaml removido
```

Essas alterações ainda **não fazem parte do histórico do Git**.

Comando usado para visualizar o estado dessa área:

```bash
git status
```

---

# 2. Staging Area (Index)

A **Staging Area** é uma área intermediária onde são preparados os arquivos que irão compor o próximo commit.

Ela permite selecionar **exatamente quais mudanças farão parte do commit**.

Fluxo:

```
Working Directory → git add → Staging Area
```

Exemplo:

```bash
git add arquivo.py
```

Nesse momento o Git registra que o estado atual de `arquivo.py` deve ser incluído no próximo commit.

Essa separação permite:

- organizar commits
    
- evitar commits acidentais
    
- agrupar alterações logicamente
    

---

# 3. Local Repository

O **Local Repository** contém o histórico completo do projeto.

Ele está armazenado dentro da pasta:

```
.git
```

Quando um commit é executado, o Git cria um snapshot do estado dos arquivos que estavam no staging.

Fluxo:

```
Staging Area → git commit → Local Repository
```

Exemplo:

```bash
git commit -m "implementa função de busca"
```

Após o commit:

- o histórico é atualizado
    
- um identificador hash é gerado para o commit
    
- os arquivos passam a fazer parte da história do projeto
    

---

# 4. Remote Repository

O **Remote Repository** é uma cópia do repositório hospedada em um servidor.

Plataformas comuns incluem:

- **GitHub**
    
- **GitLab**
    
- **Bitbucket**
    

Ele permite colaboração entre múltiplos desenvolvedores.

Fluxos principais:

Enviar alterações:

```
Local Repository → git push → Remote Repository
```

Receber alterações:

```
Remote Repository → git fetch / git pull → Local Repository
```

---

# Diagrama completo do fluxo

```
                git add
Working Directory ---------> Staging Area
       |                          |
       |                          |
       |                      git commit
       |                          |
       |                          v
       |                    Local Repository
       |                          |
       |                          |
       |                      git push
       |                          |
       v                          v
   arquivos                 Remote Repository
  modificados                 (GitHub, etc)
```

---

# Estados possíveis de um arquivo no Git

Um arquivo pode estar em um dos seguintes estados:

|Estado|Descrição|
|---|---|
|**Untracked**|arquivo ainda não monitorado pelo Git|
|**Modified**|arquivo alterado no working directory|
|**Staged**|arquivo preparado para commit|
|**Committed**|arquivo registrado no histórico|

Fluxo típico:

```
Untracked → git add → Staged → git commit → Committed
```
---
## 3. Principais comandos

A seguir, os comandos mais utilizados no dia a dia, conforme documentados no [Git Reference (git-scm.com)](https://git-scm.com/docs) e no [GitHub Git Cheat Sheet (docs.github.com)](https://docs.github.com/en/get-started/git-basics/git-cheatsheet):

**Configuração inicial**

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
```

**Iniciando e clonando repositórios**

```bash
git init              # Cria um novo repositório local
git clone <url>       # Clona um repositório remoto
```

**Acompanhando mudanças**

```bash
git status            # Exibe o estado dos arquivos no repositório
git add <arquivo>     # Adiciona arquivo(s) à staging area
git add .             # Adiciona todos os arquivos modificados
git diff              # Mostra diferenças ainda não adicionadas ao stage
git diff --staged     # Mostra diferenças dos arquivos já no stage
```

**Salvando mudanças**

```bash
git commit -m "mensagem"   # Cria um commit com os arquivos no stage
git commit -a -m "msg"     # Adiciona e commita todos os arquivos rastreados
```

**Branching e merging**

```bash
git branch                  # Lista branches locais
git branch <nome>           # Cria um novo branch
git checkout <branch>       # Muda para outro branch
git checkout -b <branch>    # Cria e já muda para o novo branch
git merge <branch>          # Faz merge do branch indicado no branch atual
```

**Sincronizando com repositório remoto**

```bash
git remote add origin <url>  # Adiciona um repositório remoto
git fetch                    # Baixa mudanças do remoto sem aplicar
git pull                     # Baixa e aplica mudanças (fetch + merge)
git push origin <branch>     # Envia commits locais para o remoto
```

**Inspecionando e desfazendo**

```bash
git log                     # Exibe histórico de commits
git log --oneline           # Histórico resumido
git revert <hash>           # Cria um novo commit que desfaz um commit anterior
git reset HEAD~             # Desfaz o último commit, mantendo as alterações
git stash                   # Guarda mudanças temporariamente sem commitar
git stash pop               # Recupera mudanças guardadas com stash
```

> **Fontes:** [Git Reference — git-scm.com/docs](https://git-scm.com/docs) | [Git Cheatsheet — docs.github.com](https://docs.github.com/en/get-started/git-basics/git-cheatsheet) | [Everyday Git — git-scm.com](https://git-scm.com/docs/giteveryday)

---

## 4. Fluxos de trabalho práticos

### Fluxo 1: Projeto novo, iniciado do zero

Este fluxo cobre desde a criação do repositório local até a publicação no GitHub.

```bash
# 1. Crie e entre na pasta do projeto
mkdir meu-projeto && cd meu-projeto

# 2. Inicializa o repositório Git
git init

# 3. Configure seus dados (se ainda não configurou globalmente)
git config user.name "Seu Nome"
git config user.email "seu@email.com"

# 4. Crie seus arquivos iniciais (ex: README e código)
echo "# Meu Projeto" > README.md
touch main.py

# 5. Adicione todos os arquivos ao stage
git add .

# 6. Faça o primeiro commit
git commit -m "feat: commit inicial do projeto"

# 7. Crie o branch principal com o nome 'main'
git branch -M main

# 8. Conecte ao repositório remoto (criado previamente no GitHub)
git remote add origin https://github.com/usuario/meu-projeto.git

# 9. Envie para o GitHub
git push -u origin main

# --- Ciclo de desenvolvimento contínuo ---

# 10. Crie um branch para a nova funcionalidade
git checkout -b feature/nova-funcionalidade

# 11. Desenvolva, edite arquivos, depois adicione e commite
git add .
git commit -m "feat: adiciona nova funcionalidade X"

# 12. Envie as mudanças para o GitHub
git push origin feature/nova-funcionalidade
```

> **Referência:** [gittutorial — git-scm.com](https://git-scm.com/docs/gittutorial) | [Everyday Git — git-scm.com](https://git-scm.com/docs/giteveryday)

---


# Exemplo completo de fluxo

1. editar arquivos
    

```
arquivo.py modificado
```

2. verificar estado
    

```bash
git status
```

3. adicionar ao staging
    

```bash
git add arquivo.py
```

4. registrar commit
    

```bash
git commit -m "implementa nova função"
```

5. enviar para servidor
    

```bash
git push origin main
```

---

# Fontes

Documentação oficial do Git  
[https://git-scm.com/docs](https://git-scm.com/docs)

Livro oficial **Pro Git**  
[https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)

Git Internals (capítulos técnicos)  
[https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

Atlassian Git Tutorials  
[https://www.atlassian.com/git/tutorials](https://www.atlassian.com/git/tutorials)

---

# Fluxo completo típico de desenvolvimento

Sequência comum de trabalho:

1. atualizar repositório
    

```bash
git pull
```

2. criar branch de trabalho
    

```bash
git switch -c nova-feature
```

3. editar arquivos
    
4. verificar alterações
    

```bash
git status
```

5. adicionar alterações
    

```bash
git add <nome_arquivo>
```

6. registrar commit
    

```bash
git commit -m "implementa nova feature"
```

7. enviar branch
    

```bash
git push origin nova-feature
```

8. integrar alterações
    

```
merge ou pull request
```

---

## Documentação oficial

Documentação oficial do Git  
[https://git-scm.com/docs](https://git-scm.com/docs)

Livro oficial **Pro Git**  
[https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)

Git Reference Manual  
[https://git-scm.com/docs](https://git-scm.com/docs)

---
## Referências técnicas

Atlassian Git Tutorials  
[https://www.atlassian.com/git/tutorials](https://www.atlassian.com/git/tutorials)

Linux Foundation — história do Git  
[https://www.linuxfoundation.org](https://www.linuxfoundation.org/)

---