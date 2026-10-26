# Projeto A3: UC - Modelagem de Software

# Banco de Dados para Controle de Estacionamento

Prof. Marcelo Amorim

## Contexto e objetivos

Cada grupo vai projetar e implementar, em SQLite, o banco de dados de um sistema de controle de estacionamento e publicar os modelos conceitual, lógico e físico em um repositório do GitHub.

O estacionamento recebe carros avulsos (rotativos), cobrados por hora, e carros de mensalistas, que pagam uma mensalidade. O sistema registra entradas, saídas, pagamentos, lojas conveniadas e ocorrências.

Ao final do projeto, o grupo deve ser capaz de:

- modelar um minimundo a partir de regras de negócio escritas em linguagem natural;
- produzir os três níveis de modelagem e mantê-los coerentes entre si;
- implementar o modelo no SQLite, com restrições de integridade;
- provar que o modelo funciona, respondendo a consultas de negócio;
- publicar o resultado de modo que outra pessoa clone o repositório e reproduza o banco.

### Fora do escopo

Não há interface nem programa: o projeto é só o banco de dados. Também não fazem parte do sistema vagas individuais, reservas, várias unidades, recarga de veículos elétricos, registro de cancelas, avarias, turnos de funcionários, tolerância de tempo, teto diário nem descontos.

## Organização e marcos

O trabalho é feito em grupos, com três entregas parciais para o professor corrigir o rumo e uma entrega final, que é a avaliada.

- Tamanho dos grupos: Máximo de 6 alunos

| Marco | O que entregar | Data |
| --- | --- | --- |
| Divulgação do enunciado | - | 26/10 |
| Grupo formado | Preencher o formulário [https://forms.gle/Zm3gKhJ9PPQZqzeG6](https://forms.gle/Zm3gKhJ9PPQZqzeG6) com a lista de integrantes e link do repositório no GitHub | 26/10 |
| Entrega parcial 1 | Modelo conceitual | 09/11 |
| Entrega parcial 2 | Modelo lógico | 16/11 |
| Entrega parcial 3 | Script de criação das tabelas (físico) e dados de teste | 23/11 |
| Entrega final | Repositório completo, com as 17 consultas e toda a documentação | 30/11 |

As entregas parciais não têm nota própria. Elas existem para o professor apontar problemas antes da avaliação final.

## Base fornecida

O arquivo [base_carros.sql](./base_carros.sql) cria e popula as tabelas `marca` (17 linhas) e `modelo` (77 linhas), que o grupo recebe prontas e não pode alterar.

| Tabela | Colunas | Observações |
| --- | --- | --- |
| `marca` | `cod_marca` (chave primária), `nome` | `nome` é único |
| `modelo` | `cod_modelo` (chave primária), `nome`, `carroceria`, `cod_marca` | `cod_marca` referencia `marca`; `carroceria` é Hatch, Sedã, SUV, Picape ou Minivan; o par marca e nome é único |

Como usar:

1. Copie o arquivo, sem alterações, para `sql/00-base-fornecida.sql` do seu repositório.
2. Execute-o antes de qualquer script do grupo.
3. As tabelas do grupo que tratam de veículos devem referenciar `modelo (cod_modelo)` por chave estrangeira.

Os códigos são fixos, então os seus dados de teste podem citar `cod_modelo` diretamente. Os modelos existem no mercado brasileiro, mas a lista é ilustrativa. Todo o restante do banco é modelado pelo grupo.

## Regras de negócio

As 15 regras abaixo definem o sistema: modele exatamente o que elas dizem e, onde forem omissas, decida e registre a decisão em `docs/decisoes.md`.

São de dois tipos. Umas o banco deve garantir, com restrições (chaves, `NOT NULL`, `UNIQUE`, `CHECK`). Outras o banco não bloqueia, mas o modelo precisa permitir verificá-las por consulta (RN05 e RN14, consultas 16 e 17).

### Veículos, clientes e capacidade

- **RN01.** Todo veículo é identificado pela placa e tem um modelo da base fornecida. Só há carros.
- **RN02.** Um veículo pode ou não estar associado a um cliente. Quando uma placa desconhecida entra, registra-se uma entrada avulsa: o veículo é cadastrado (placa e modelo) sem cliente. Se o motorista é mensalista, ele se identifica e o veículo passa a ser associado ao cliente.
- **RN03.** O modelo deve armazenar os dados do estacionamento (nome e endereço) e a sua capacidade máxima de veículos.
- **RN04.** O modelo deve permitir calcular quantas vagas estão livres: a capacidade menos os veículos que estão dentro (estadias sem saída). Rotativos e mensalistas dividem a mesma capacidade.
- **RN05.** Um veículo não deve ter duas estadias em aberto ao mesmo tempo.

### Estadias, tarifa e pagamento

- **RN06.** Cada entrada gera uma estadia, com o veículo, o funcionário que a registrou e a data e hora de entrada. A saída é registrada depois.
- **RN07.** A estadia é de mensalista quando o veículo tem contrato vigente na data da entrada. Esse tipo de estadia não gera tarifa nem pagamento.
- **RN08.** Nas demais estadias (rotativas), o valor é a tarifa vigente na data da entrada multiplicada pelas horas de permanência, e a fração de hora conta como hora cheia (2h30 equivalem a 3 horas). A tarifa é única, em valor por hora.
- **RN09.** As tarifas mudam com o tempo, e é preciso saber qual valia em cada data.
- **RN10.** O pagamento registra a forma (dinheiro, débito, crédito ou PIX), o valor e o funcionário que o recebeu. Uma estadia rotativa tem um único pagamento, com uma única forma.
- **RN11.** Uma estadia pode estar vinculada a, no máximo, uma loja conveniada. É só um registro, sem desconto. As lojas conveniadas são cadastradas.

### Mensalistas

- **RN12.** O contrato registra o cliente (titular), um único veículo, a vigência (início e fim) e o valor mensal. Não há vaga fixa.
- **RN13.** A mensalidade é cobrada por competência (mês e ano), com data de vencimento e data de pagamento. Sem data de pagamento, a mensalidade está pendente.
- **RN14.** Um veículo não deve ter dois contratos com períodos sobrepostos.

### Ocorrências

- **RN15.** Toda ocorrência tem data e hora, tipo, descrição, a estadia envolvida e o funcionário que a registrou.

Os funcionários têm cadastro simples, sem controle de turnos.

## Os três modelos

O grupo entrega o modelo conceitual, o lógico e o físico, e os três precisam contar a mesma história, cada um no seu nível de abstração.

### O que muda de um nível para outro

|  | Conceitual | Lógico | Físico |
| --- | --- | --- | --- |
| Pergunta que responde | O que o negócio precisa guardar? | Como um banco relacional guarda isso? | Como isso fica no SQLite? |
| Diagrama | Diagrama entidade-relacionamento (DER) | Diagrama de tabelas, com chaves | Script SQL |
| Elementos | Entidades, atributos, relacionamentos | Tabelas, colunas, chaves | Tabelas, tipos, restrições, índices |
| Relacionamentos | Losango com cardinalidade (mínima, máxima); o N:N é permitido | Não existem: viram chave estrangeira ou tabela associativa | Chaves estrangeiras (`FOREIGN KEY`) |
| Atributos especiais | Compostos e multivalorados são permitidos | Só valores simples: o composto vira colunas, o multivalorado vira outra tabela | Colunas com tipo |
| Identificação | Atributo identificador | Chave primária, simples ou composta | `PRIMARY KEY` |
| Normalização | Não se aplica | Até a 3FN | Herdada do lógico |
| Tipos de dados | Não aparecem | Genéricos ou ausentes | `INTEGER`, `TEXT`, `REAL`... |

Em resumo: o conceitual descreve o negócio, sem pensar em tabelas. O lógico traduz esse mesmo conteúdo para o vocabulário do modelo relacional. O físico o implementa em um SGBD concreto.

### Exemplo de tradução do conceitual para o lógico

O exemplo usa o catálogo de filmes visto em aula, não este projeto.

| No conceitual | No lógico |
| --- | --- |
| Estúdio produz Filme (1:N) | `Filme (CodFilme, ..., CodEstudio)`, com `CodEstudio` referenciando `Estudio` |
| Filme pertence a Gênero (N:N) | Tabela associativa `FilmeGenero (CodFilme, CodGenero)` |
| Usuário favorita Filme (N:N, com data de adição) | `Favorito (CodUsuario, CodFilme, DataAdicao)`, com chave primária composta |
| Atributo composto Endereço | Colunas `Logradouro`, `Cidade`, `UF` e `CEP` |
| Atributo multivalorado Telefones | Tabela `TelefoneUsuario (CodUsuario, Telefone)` |
| Dependente (entidade fraca) | `Dependente (CodEmpregado, Nome, ...)`, com chave primária composta |

### Ferramentas

Por padrão, os diagramas conceitual e lógico são feitos no **brModelo**. É permitido usar outra ferramenta, e também gerar o modelo físico a partir do lógico, desde que:

1. o conceitual seja um diagrama entidade-relacionamento com cardinalidades (mínima, máxima) e uma **legenda** que explique como lê-las;
2. o lógico mostre as tabelas, com chaves primárias e estrangeiras;
3. a diferença entre os dois níveis seja respeitada, como na tabela acima.

Entregue o arquivo do projeto da ferramenta, além da imagem exportada.

### Restrições mínimas no modelo físico

- `PRIMARY KEY` em todas as tabelas e `FOREIGN KEY` em todos os relacionamentos.
- `NOT NULL` em tudo o que for obrigatório pelas regras.
- Pelo menos 3 `CHECK` com sentido de negócio (por exemplo, forma de pagamento válida, saída posterior à entrada, valores positivos).
- Pelo menos 2 `UNIQUE` e pelo menos 1 `DEFAULT`.
- O script começa com `PRAGMA foreign_keys = ON;` e cria as tabelas na ordem das dependências.

### Coerência entre os modelos

O modelo físico pode ser gerado a partir do lógico, mas o critério é a coerência:

- cada entidade e cada relacionamento do conceitual tem correspondente no lógico, e as diferenças são explicadas;
- o físico implementa o lógico, com as mesmas tabelas e chaves;
- as 17 consultas rodam no banco entregue e dão resultados que fazem sentido com os dados de teste.

## Dados de teste

O grupo entrega um arquivo `sql/02-dados.sql`, com os `INSERT`s que populam o banco, e esses dados precisam cobrir os cenários mínimos da tabela abaixo.

| Dado | Mínimo exigido |
| --- | --- |
| Estacionamento | 1 registro, com capacidade de 80 veículos |
| Funcionários | 3 |
| Clientes | 5 |
| Veículos | 15, de pelo menos 5 modelos e 3 marcas da base fornecida; pelo menos 3 sem cliente associado |
| Tarifas | 2 ou mais, com vigências diferentes, uma delas vigente em 05/03/2024 |
| Contratos | 3, de clientes diferentes; pelo menos 1 já encerrado |
| Mensalidades | 6, com pelo menos 2 pagas e 2 pendentes |
| Lojas conveniadas | 3, sendo 1 sem nenhuma estadia vinculada |
| Estadias | 30, em pelo menos 3 dias diferentes; pelo menos 3 ainda em aberto; pelo menos 5 de mensalistas; pelo menos 3 com permanência fracionada (por exemplo, 2h30); pelo menos 6 vinculadas a lojas |
| Pagamentos | Um para cada estadia rotativa encerrada, usando as quatro formas de pagamento |
| Ocorrências | 5, de pelo menos 2 tipos |

Regras para os dados:

- Os dados devem obedecer às 15 regras. Em particular, as consultas 16 e 17 (auditoria) devem retornar vazio.
- Use apenas dados fictícios: nada de placas, nomes ou documentos de pessoas reais.
- O script deve rodar sem erro logo depois do script de criação, com as chaves estrangeiras ativas.

## Consultas obrigatórias

O grupo escreve as 17 consultas abaixo sobre o seu próprio modelo, e elas valem 25 dos 100 pontos do projeto. As perguntas estão em linguagem de negócio: os nomes das tabelas e colunas são os que o grupo escolheu.

| Nº | Nível | Pergunta | Regra | Pontos |
| --- | --- | --- | --- | --- |
| 1 | Básico | Quais veículos estão no estacionamento agora (entrada sem saída)? Mostre placa e horário de entrada, do mais antigo ao mais recente. | RN06 | 1 |
| 2 | Básico | Quais veículos cadastrados não têm cliente associado? | RN02 | 1 |
| 3 | Básico | Quais mensalidades ainda não foram pagas? Mostre contrato, competência e vencimento. | RN13 | 1 |
| 4 | Básico | Qual era a tarifa por hora vigente em 05/03/2024? | RN09 | 1 |
| 5 | Agregação | Quantas estadias começaram em cada dia? | RN06 | 1,5 |
| 6 | Agregação | Quanto foi recebido em cada forma de pagamento? | RN10 | 1,5 |
| 7 | Agregação | Quantas ocorrências houve de cada tipo? | RN15 | 1,5 |
| 8 | Agregação | Qual a permanência média, em minutos, das estadias encerradas? | RN06 | 1,5 |
| 9 | Junções | Para cada estadia, mostrar a placa, a marca e o modelo do veículo. | RN01 | 2 |
| 10 | Junções | Listar os contratos com o nome do cliente, a placa e o modelo do veículo. | RN12 | 2 |
| 11 | Junções | Quantas estadias cada loja conveniada vinculou, incluindo as lojas sem nenhuma? | RN11 | 2 |
| 12 | Junções | Listar as ocorrências com data e hora, tipo, placa do veículo e nome do funcionário que registrou. | RN15 | 2 |
| 13 | Avançado | Quantas vagas estão livres agora? | RN03, RN04 | 1,5 |
| 14 | Avançado | Qual o valor devido em cada estadia rotativa encerrada, pela tarifa vigente na data da entrada, com fração contando como hora cheia? | RN08, RN09 | 2 |
| 15 | Avançado | Classificar cada estadia como rotativa ou de mensalista (veículo com contrato vigente na data da entrada). | RN07 | 1,5 |
| 16 | Avançado | Auditoria: quais placas têm mais de uma estadia em aberto? (deve retornar vazio) | RN05 | 1 |
| 17 | Avançado | Auditoria: quais contratos do mesmo veículo têm períodos sobrepostos? (deve retornar vazio) | RN14 | 1 |

### Como entregar

- Um arquivo `sql/03-consultas.sql`, com as consultas numeradas e a pergunta de cada uma em comentário.
- O resultado que cada consulta devolve com os dados de teste do grupo, em comentário no próprio arquivo ou no README, para que o professor confira sem precisar executá-las.
- Cada consulta deve responder exatamente à pergunta, e não a uma versão parecida.

### Como são corrigidas

- **Pontuação cheia:** a consulta roda e responde corretamente à pergunta.
- **Metade dos pontos:** a consulta roda, mas o resultado está incompleto ou impreciso.
- **Zero:** a consulta não roda ou não responde à pergunta.

Uma consulta que não roda no banco entregue costuma indicar que o modelo físico não corresponde ao modelo lógico, e isso também é observado no critério de coerência.

## Repositório e entregas

O produto final é um repositório público no GitHub que qualquer pessoa consiga clonar e, executando os scripts em ordem, obter o banco criado e populado sem erros.

### Estrutura de pastas

```
README.md
docs/
  01-conceitual/      DER (imagem e arquivo do projeto) e descrição
  02-logico/          diagrama lógico, esquema relacional em texto, justificativas
  03-fisico/          decisões de tipos, restrições e índices
  decisoes.md         decisões tomadas onde as regras eram omissas
sql/
  00-base-fornecida.sql
  01-schema.sql
  02-dados.sql
  03-consultas.sql
```

### O que o README deve conter

- nome do projeto, **RA e nome completo** dos integrantes do grupo e o que o banco faz;
- como reproduzir o banco: a ordem dos scripts e como executá-los no DB Browser for SQLite;
- os diagramas, ou links para eles;
- as ferramentas usadas;
- a licença do código e dos dados.
- Em `docs/decisoes.md`, registre cada decisão onde as regras eram omissas, com a justificativa.

### Entregas

As entregas parciais e a final são feitas pelo link do repositório, e vale o que estiver no ramo principal na data de cada marco.

## Avaliação

O projeto vale 100 pontos, e o maior peso está nas consultas, porque elas mostram se o modelo e os dados realmente funcionam.

| Critério | Pontos | O que se avalia |
| --- | --- | --- |
| Modelo conceitual | 15 | Todas as entidades, atributos e relacionamentos necessários às regras; cardinalidades coerentes; notação correta, com legenda; entidade fraca e atributos especiais bem usados. |
| Modelo lógico | 15 | Tabelas, chaves primárias e estrangeiras; N:N resolvido; atributos especiais traduzidos; normalização até a 3FN; justificativa das decisões de tradução. |
| Modelo físico | 15 | Script que roda sem erro; tipos adequados; as restrições mínimas exigidas; ordem de criação correta. |
| Coerência entre os modelos | 10 | Correspondência entre conceitual, lógico e físico; diferenças explicadas; banco entregue igual ao modelo documentado. |
| Dados de teste | 10 | Cenários mínimos atendidos; dados que obedecem às regras; script que roda logo após a criação. |
| Consultas obrigatórias | 25 | As 17 consultas, conforme a pontuação de cada uma. |
| Repositório, documentação e colaboração | 10 | Estrutura de pastas, README, decisões registradas, licença, histórico de commits de todos os integrantes. |
| **Total** | **100** |  |

### Nota individual

A nota do grupo vale para todos os integrantes. O professor pode ajustar a nota de quem não tiver contribuição relevante no histórico do repositório e pode pedir uma breve explicação oral do projeto.

## Dúvidas frequentes

| Pergunta | Resposta |
| --- | --- |
| Posso criar tabelas além das necessárias? | Sim, desde que sirvam às regras e a decisão fique registrada em `docs/decisoes.md`. |
| Posso alterar as tabelas `marca` e `modelo`? | Não. Elas são fornecidas e fixas. |
| A regra é omissa. O que faço? | Decida, registre a decisão e a justificativa em `docs/decisoes.md` e siga em frente. |
| Preciso construir uma interface? | Não. O projeto é só o banco de dados. |
| Qual banco de dados usar? | SQLite. Use o DB Browser for SQLite para criar, popular e consultar. |
| Posso gerar o modelo físico a partir do lógico? | Sim, desde que o resultado seja coerente com os modelos e execute as consultas. |
| Posso usar dados de pessoas reais? | Não. Todos os dados de teste devem ser fictícios. |
| Posso consultar modelos de outros bancos como referência? | Sim, desde que citados no README. Copiar o modelo de outro grupo não é permitido. |