# Capítulo 12 - Normalização

**Adrienne Watt**
https://opentextbc.ca/dbdesign01/chapter/chapter-12-normalization/

A normalização deve fazer parte do processo de design de banco de dados. No entanto, é difícil separar o processo de normalização do processo de modelagem ER, então as duas técnicas devem ser usadas simultaneamente.

Use um diagrama de entidade-relacionamento (DER) para fornecer uma visão geral, ou visão macro, dos requisitos de dados e operações de uma organização. Isso é criado através de um processo iterativo que envolve identificar entidades relevantes, seus atributos e relacionamentos.

O procedimento de normalização foca nas características de entidades específicas e representa a visão micro das entidades dentro do DER.

## O Que É Normalização?

A normalização é o ramo da teoria relacional que fornece insights de design. É o processo de determinar quanta redundância existe em uma tabela. Os objetivos da normalização são:

- Ser capaz de caracterizar o nível de redundância em um esquema relacional
- Fornecer mecanismos para transformar esquemas a fim de remover redundância

A teoria de normalização baseia-se fortemente na teoria de dependências funcionais. A teoria de normalização define seis formas normais (FN). Cada forma normal envolve um conjunto de propriedades de dependência que um esquema deve satisfazer e cada forma normal fornece garantias sobre a presença e/ou ausência de anomalias de atualização. Isso significa que formas normais mais altas têm menos redundância e, como resultado, menos problemas de atualização.

## Formas Normais

Todas as tabelas em qualquer banco de dados podem estar em uma das formas normais que discutiremos a seguir. Idealmente, queremos apenas redundância mínima para CP (Chave Primária) para CE (Chave Estrangeira). Tudo o mais deve ser derivado de outras tabelas. Existem seis formas normais, mas vamos examinar apenas as primeiras quatro, que são:

- Primeira forma normal (1FN)
- Segunda forma normal (2FN)
- Terceira forma normal (3FN)
- Forma normal de Boyce-Codd (FNBC)

FNBC é raramente usada.

## Primeira Forma Normal (1FN)

Na primeira forma normal, apenas valores únicos são permitidos na interseção de cada linha e coluna; portanto, não há grupos repetitivos.

Para normalizar uma relação que contém um grupo repetitivo, remova o grupo repetitivo e forme duas novas relações.

A CP da nova relação é uma combinação da CP da relação original mais um atributo da relação recém-criada para identificação única.

### Processo para 1FN

Usaremos a tabela Student_Grade_Report abaixo, de um banco de dados Escolar, como nosso exemplo para explicar o processo para 1FN.
````
Student_Grade_Report (StudentNo, StudentName, Major, CourseNo, CourseName, InstructorNo, InstructorName, InstructorLocation, Grade)
````
- Na tabela Relatório de Notas do Estudante, o grupo repetitivo é a informação do curso. Um estudante pode fazer muitos cursos.
- Remova o grupo repetitivo. Neste caso, são as informações do curso para cada estudante.
- Identifique a CP para sua nova tabela.
- A CP deve identificar unicamente o valor do atributo (StudentNo e CourseNo).
- Após remover todos os atributos relacionados ao curso e estudante, você fica com a tabela curso do estudante (StudentCourse).
- A tabela Student (Estudante) agora está na primeira forma normal com o grupo repetitivo removido.
- As duas novas tabelas são mostradas abaixo.
````
Student (StudentNo, StudentName, Major)
````

````
StudentCourse (StudentNo, CourseNo, CourseName, InstructorNo, InstructorName, InstructorLocation, Grade)
````
### Como atualizar anomalias de 1FN

StudentCourse (StudentNo, CourseNo, CourseName, InstructorNo, InstructorName, InstructorLocation, Grade)

- Para adicionar um novo curso, precisamos de um estudante.
- Quando informações do curso precisam ser atualizadas, podemos ter inconsistências.
- Para deletar um estudante, podemos também deletar informações críticas sobre um curso.

## Segunda Forma Normal (2FN)

Para a segunda forma normal, a relação deve primeiro estar em 1FN. A relação está automaticamente em 2FN se, e somente se, a CP compreende um único atributo.

Se a relação tem uma CP composta, então cada atributo não-chave deve ser totalmente dependente de toda a CP e não de um subconjunto da CP (ou seja, não deve haver dependência parcial ou aumentativa).

### Processo para 2FN

Para mover para 2FN, uma tabela deve primeiro estar em 1FN.

- A tabela Student já está em 2FN porque tem uma CP de coluna única.
- Ao examinar a tabela Student Course, vemos que nem todos os atributos são totalmente dependentes da CP; especificamente, todas as informações do curso. O único atributo que é totalmente dependente é nota.
- Identifique a nova tabela que contém as informações do curso.
- Identifique a CP para a nova tabela.
- As três novas tabelas são mostradas abaixo.

### Como atualizar anomalias de 2FN

- Ao adicionar um novo instrutor, precisamos de um curso.
- Atualizar informações do curso pode levar a inconsistências para informações do instrutor.
- Deletar um curso pode também deletar informações do instrutor.

## Terceira Forma Normal (3FN)

Para estar na terceira forma normal, a relação deve estar na segunda forma normal. Também todas as dependências transitivas devem ser removidas; um atributo não-chave não pode ser funcionalmente dependente de outro atributo não-chave.

### Processo para 3FN

- Elimine todos os atributos dependentes em relacionamento(s) transitivo(s) de cada uma das tabelas que têm um relacionamento transitivo.
- Crie nova(s) tabela(s) com dependência removida.
- Verifique nova(s) tabela(s) bem como tabela(s) modificada(s) para garantir que cada tabela tenha um determinante e que nenhuma tabela contenha dependências inadequadas.
- Veja as quatro novas tabelas abaixo.

Neste estágio, não deve haver anomalias na terceira forma normal. Vamos olhar o diagrama de dependência (Figura 12.1) para este exemplo. O primeiro passo é remover grupos repetitivos, como discutido acima.

Student (StudentNo, StudentName, Major) StudentCourse (StudentNo, CourseNo, CourseName, InstructorNo, InstructorName, InstructorLocation, Grade)

Para recapitular o processo de normalização para o banco de dados Escolar, revise as dependências mostradas na Figura 12.1.

As abreviações usadas na Figura 12.1 são as seguintes:

- PD: dependência parcial
- TD: dependência transitiva
- FD: dependência completa (Nota: FD tipicamente significa dependência funcional. Usar FD como abreviação para dependência completa é usado apenas na Figura 12.1.)

## Forma Normal de Boyce-Codd (FNBC)

Quando uma tabela tem mais de uma chave candidata, anomalias podem resultar mesmo que a relação esteja em 3FN. A forma normal de Boyce-Codd é um caso especial de 3FN. Uma relação está em FNBC se, e somente se, todo determinante é uma chave candidata.

### Exemplo FNBC 1

Considere a seguinte tabela (St_Maj_Adv).

|Student_id|Major|Advisor|
|---|---|---|
|111|Physics|Smith|
|111|Music|Chan|
|320|Math|Dobbs|
|671|Physics|White|
|803|Physics|Smith|

As regras semânticas (regras de negócio aplicadas ao banco de dados) para esta tabela são:

- Cada Estudante pode se especializar em várias matérias.
- Para cada Especialização, um dado Estudante tem apenas um Orientador.
- Cada Especialização tem vários Orientadores.
- Cada Orientador orienta apenas uma Especialização.
- Cada Orientador orienta vários Estudantes em uma Especialização.

As dependências funcionais para esta tabela são listadas abaixo. A primeira é uma chave candidata; a segunda não é.

- Student_id, Major ——> Advisor
- Advisor ——> Major

Anomalias para esta tabela incluem:

- Delete – estudante deleta informação do orientador
- Insert – um novo orientador precisa de um estudante
- Update – inconsistências

Nota: Nenhum atributo único é uma chave candidata. CP pode ser Student_id, Major ou Student_id, Advisor.

Para reduzir a relação St_Maj_Adv para FNBC, você cria duas novas tabelas:

- St_Adv (Student_id, Advisor)
- Adv_Maj (Advisor, Major)

**Tabela St_Adv**

|Student_id|Advisor|
|---|---|
|111|Smith|
|111|Chan|
|320|Dobbs|
|671|White|
|803|Smith|

**Tabela Adv_Maj**

|Advisor|Major|
|---|---|
|Smith|Physics|
|Chan|Music|
|Dobbs|Math|
|White|Physics|

### Exemplo FNBC 2

Considere a seguinte tabela (Client_Interview).

|ClientNo|InterviewDate|InterviewTime|StaffNo|RoomNo|
|---|---|---|---|---|
|CR76|13-May-02|10.30|SG5|G101|
|CR56|13-May-02|12.00|SG5|G101|
|CR74|13-May-02|12.00|SG37|G102|
|CR56|1-July-02|10.30|SG5|G102|

FD1 – ClientNo, InterviewDate –> InterviewTime, StaffNo, RoomNo (CP) FD2 – staffNo, interviewDate, interviewTime –> clientNO (chave candidata: CC) FD3 – roomNo, interviewDate, interviewTime –> staffNo, clientNo (CC) FD4 – staffNo, interviewDate –> roomNo

Uma relação está em FNBC se, e somente se, todo determinante é uma chave candidata. Precisamos criar uma tabela que incorpore as três primeiras FDs (tabela Client_Interview2) e outra tabela (tabela StaffRoom) para a quarta FD.

**Tabela Client_Interview2**

|ClientNo|InterviewDate|InterViewTime|StaffNo|
|---|---|---|---|
|CR76|13-May-02|10.30|SG5|
|CR56|13-May-02|12.00|SG5|
|CR74|13-May-02|12.00|SG37|
|CR56|1-July-02|10.30|SG5|

**Tabela StaffRoom**

|StaffNo|InterviewDate|RoomNo|
|---|---|---|
|SG5|13-May-02|G101|
|SG37|13-May-02|G102|
|SG5|1-July-02|G102|

## Normalização e Design de Banco de Dados

Durante o processo de normalização do design de banco de dados, certifique-se de que as entidades propostas atendam à forma normal exigida antes que as estruturas de tabela sejam criadas. Muitos bancos de dados do mundo real foram inadequadamente projetados ou sobrecarregados com anomalias se inadequadamente modificados ao longo do tempo. Você pode ser solicitado a reprojetar e modificar bancos de dados existentes. Isso pode ser um grande empreendimento se as tabelas não estiverem adequadamente normalizadas.

## Termos Chave e Abreviações

**primeira forma normal (1FN)**: apenas valores únicos são permitidos na interseção de cada linha e coluna, então não há grupos repetitivos

**normalização**: o processo de determinar quanta redundância existe em uma tabela

**segunda forma normal (2FN)**: a relação deve estar em 1FN e a CP compreende um único atributo

**regras semânticas**: regras de negócio aplicadas ao banco de dados

**terceira forma normal (3FN)**: a relação deve estar em 2FN e todas as dependências transitivas devem ser removidas; um atributo não-chave não pode ser funcionalmente dependente de outro atributo não-chave

## Exercícios

Complete os capítulos 11 e 12 antes de fazer estes exercícios.

1. O que é normalização?
2. Quando uma tabela está em 1FN?
3. Quando uma tabela está em 2FN?
4. Quando uma tabela está em 3FN?
5. Identifique e discuta cada uma das dependências indicadas no diagrama de dependência mostrado na Figura 12.2.
6. Para acompanhar estudantes e cursos, uma nova faculdade usa a estrutura de tabela na Figura 12.3. Desenhe o diagrama de dependência para esta tabela.
7. Usando o diagrama de dependência que você acabou de desenhar, mostre as tabelas (em sua terceira forma normal) que você criaria para corrigir os problemas que encontrou. Desenhe o diagrama de dependência para a tabela corrigida.
8. Uma agência chamada Instant Cover fornece pessoal de meio período/temporário para hotéis na Escócia. A Figura 12.4 lista o tempo gasto pelo pessoal da agência trabalhando em vários hotéis. O número de seguro nacional (NIN) é único para cada membro do pessoal. Use a Figura 12.4 para responder às questões (a) e (b). a. Esta tabela é suscetível a anomalias de atualização. Forneça exemplos de anomalias de inserção, deleção e atualização. b. Normalize esta tabela para a terceira forma normal. Declare quaisquer suposições.
9. Preencha os espaços: a. ____________________ produz uma forma normal mais baixa. b. Qualquer atributo cujo valor determina outros valores dentro de uma linha é chamado de ____________________. c. Um atributo que não pode ser mais dividido é dito exibir ____________________. d. ____________________ refere-se ao nível de detalhe representado pelos valores armazenados na linha de uma tabela. e. Uma tabela relacional não deve conter grupos ____________________.

Veja também Apêndice B: Exercícios de DER de Exemplo

## Bibliografia

Nguyen Kim Anh, Relational Design Theory. OpenStax CNX. 8 Jul 2009 Recuperado em julho de 2014 de http://cnx.org/contents/606cc532-0b1d-419d-a0ec-ac4e2e2d533b@1@1

Russell, Gordon. Chapter 4 – Normalisation. Database eLearning. N.d. Recuperado em julho de 2014 de db.grussell.org/ch4.html