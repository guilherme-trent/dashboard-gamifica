# Gamifica · Dashboard Educacional

> **Vitrine do projeto.** O código é privado porque o sistema guarda dados de alunos, muitos deles menores de idade. Aqui estão o problema, a solução, as decisões de arquitetura e os resultados.

*English summary: the official enrollment and attendance system of Gamifica, a public tech-education program in Brasília (DF). It has more than 22,000 recorded sessions, AI reading of handwritten attendance lists with CPF checksum validation, permissions enforced in the database (RLS), and 53 automated tests.*

## O problema
O Gamifica leva cursos de programação, jogos, design e marketing a escolas públicas do DF, com laboratórios móveis em 7 trilhas. A operação precisava de um registro único de inscrições e atendimentos, alimentado por listas de presença em papel e planilhas, e de números confiáveis para a prestação de contas. Também precisava proteger os dados dos alunos.

## A solução
Uma plataforma web que virou o **registro oficial** de inscrições e atendimentos do projeto:

- **Cadastro de alunos e presença** nas 7 trilhas, com busca e filtros que ficam na URL (o link já abre com o filtro).
- **Importação de planilhas**: só o nome é obrigatório; o resto é opcional.
- **Leitura de listas de presença por foto**: a equipe fotografa a lista em papel com o celular, e a IA devolve nome e CPF de cada linha.
- **Revisão e aprovação**: o que a equipe envia fica "em revisão" e só conta nos indicadores depois da aprovação de um administrador.
- **Prestação de contas**: exportação para planilhas e relatórios, e uma visão pública só com totais por região.


## Telas
> Telas reais do sistema com **dados fictícios**: nomes, locais e CPFs inventados.

### Visão geral
Os indicadores em tempo real: alunos, atendimentos e o avanço da meta do contrato, o volume por trilha e o mapa de atendimentos por região. O aviso no topo mostra o que a equipe enviou e ainda espera revisão.

![Visão geral com indicadores, volume por trilha e mapa de atendimentos por região](telas/01-visao-geral.webp)

### Alunos
Busca por nome ou CPF e as 7 trilhas de cada aluno num relance. Os filtros (região, local, turma e situação nas trilhas) ficam no link: dá para compartilhar uma lista já filtrada.

![Lista de alunos com as trilhas concluídas](telas/02-alunos.webp)
![Lista de alunos com os filtros abertos](telas/03-alunos-filtros.webp)

### Ficha do aluno
A presença nas trilhas é marcada com um toque. Os dados pessoais ficam ocultos e só aparecem quando um administrador pede para ver.

![Ficha do aluno com o registro de presença nas trilhas](telas/04-aluno-detalhe.webp)

### Revisão e aprovação
Tudo o que a equipe envia (por foto, planilha ou cadastro manual) espera aqui. O administrador confere, corrige e aprova; só então entra nos indicadores.

![Envio da equipe em revisão, com aprovar, mudar local e rejeitar](telas/05-revisao-envio.webp)

### Regiões e locais
As regiões administrativas e os locais de atendimento, com alunos e atendimentos de cada um.

![Regiões e locais de atendimento com os totais](telas/06-regioes-e-locais.webp)

### Importação por foto
A equipe escolhe para onde vão os alunos e fotografa a lista de presença em papel; a IA lê nome e CPF de cada linha.

![Importação por câmera: região, local, turma e trilhas](telas/07-importar-camera.webp)

### Equipe e acessos
Cada pessoa tem um papel (administrador, equipe ou sem acesso), e as permissões são aplicadas no próprio banco.

![Equipe e acessos com os papéis de cada pessoa](telas/08-equipe-e-acessos.webp)

### No celular
<p>
  <img src="telas/09-celular-visao-geral.webp" alt="Visão geral no celular" width="260">
  &nbsp;
  <img src="telas/10-celular-alunos.webp" alt="Lista de alunos no celular" width="260">
</p>

## Resultados
- **Mais de 22 mil atendimentos** registrados.
- Usada pela equipe no dia a dia para consultar e cadastrar inscrições.
- **53 testes automatizados**, incluindo os do próprio banco de dados.

## Arquitetura

```mermaid
flowchart LR
  E["Equipe no celular"] -->|"foto da lista"| F["Função serverless (Vercel)"]
  F -->|"leitura"| IA["Claude Haiku (Anthropic)"]
  IA --> V{"Validação dos dígitos do CPF"}
  V -->|"linhas confiáveis"| R["Fila de revisão"]
  V -->|"linhas suspeitas"| M["Marcadas para conferência"]
  R -->|"aprovação do admin"| DB[("PostgreSQL (Supabase)")]
  W["Painel web (React)"] --> DB
  DB --> P{"Permissões no banco (RLS)"}
```

## Decisões que valem destacar
- **Permissões aplicadas no próprio banco, não só na tela.** Três perfis: administrador, equipe e público. A visão pública mostra só totais, sem nenhum nome ou CPF. Como as regras vivem no PostgreSQL, valem mesmo para quem tentar acessar a API diretamente. Isso é essencial com dados de menores de idade, e é o que a LGPD pede.
- **IA com conferência, não com confiança cega.** A leitura por foto às vezes troca o CPF de uma linha pelo da linha de baixo. A validação dos dígitos verificadores pega a maioria dessas trocas, e nada entra no registro sem passar pela revisão.
- **Fluxo de aprovação.** A equipe cadastra e importa, mas o que ela envia só conta nos indicadores depois de aprovado. Erros não contaminam os números da prestação de contas.
- **A chave da IA nunca vai para o navegador.** A leitura das fotos roda numa função serverless.

## Stack
React · TypeScript · Vite · Tailwind CSS · TanStack Query · Zod · Supabase / PostgreSQL (RLS) · Vercel (serverless) · API da Anthropic (Claude Haiku) · Vitest

## Meu papel
Projeto de iniciativa própria: idealizei, arquitetei e desenvolvi a plataforma, e sou o administrador (reviso e aprovo o que a equipe envia).
