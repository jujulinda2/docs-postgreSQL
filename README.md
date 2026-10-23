# docs-postgreSQL



# Documentação Técnica de Banco de Dados

**Status da documentação:** Este arquivo consolida e acumula continuamente todas as camadas e códigos fornecidos da construção do banco de dados. A cada nova inclusão de código enviada, este documento é reeditado na íntegra para manter a documentação unificada.

---

## Identificação do nome do banco de dados, tabelas necessárias e atributos

Para identificar o nome do banco de dados é necessário entender qual sistema está sendo criado e para qual finalidade. Como por exemplo, a criação de um sistema para o colégio plural. Uma sugestão de nome para o banco é `colegio_plural`.

Para identificar quais tabelas são necessárias, uma dica é procurar palavras como cadastrar e registrar, além de pensar "O sistema precisa guardar vários registros desse tipo de coisa?" Se a resposta for sim, provavelmente é uma tabela. Por exemplo, no contexto do sistema para o colégio plural diz:

- "permitir organizar o cadastro dos responsáveis", "permitindo organizar os dados dos alunos e responsáveis”
- "Ao cadastrar um novo Aluno, o usuário deverá associá-lo a um Responsável."
- "registro e gerenciamento dos atendimentos realizados pelo setor.”
- "A funcionalidade de Atendimento deverá apresentar a listagem dos atendimentos cadastrados ordenada por data, trazendo os dados relacionados ao responsável e ao aluno."
- "importância da autenticação dos usuários com tempo de expiração"
- "Na funcionalidade principal do sistema deverá exibir o nome do usuário logado..."

Essas frases nos ajudam a identificar que deve existir 4 tabelas. Aluno, Responsável, Atendimento e Usuário.

Para identificar os atributos necessários em cada tabela é necessário pensar quais informações eu preciso guardar sobre cada tabela. Lembrando sempre que toda tabela deve ter uma identificação de chave primária (id). Por exemplo, no contexto do colégio plural:

### Aluno

Para o aluno, o sistema pede que ao ser cadastrado, seja associado a um responsável. Então nós já sabemos que a tabela aluno deve receber o id (chave primária) e uma chave estrangeira que o associe ao seu responsável (`responsavel_id`). Outras informações como nome, cpf e data\_nascimento podem ser adicionadas para completar.

### Responsável

Para o responsável, o sistema precisa ser capaz de cadastrar e buscar responsáveis. O id é necessário, assim como em todas as tabelas. Atributos como nome, cpf, telefone, e-mail e endereço podem ser adicionados para complementar a tabela e deixar o sistema mais conciso.

### Atendimento

Para agendamento, o sistema deve cadastrar, listar e atualizar atendimentos. O id é necessário. Assim como qualquer agendamento, é necessário que tenha uma data e opcionalmente uma descrição. Todo atendimento é associado a um aluno e a um usuário, devendo ter então as chaves estrangeiras (`aluno_id`) e (`usuario_id`).

### Usuário

Para o usuário, o sistema fala em autenticação, nome do usuário e tempo de expiração. O id é necessário. O nome o próprio sistema pede. Para fazer a autenticação, um campo como e-mail serve e a senha valida a autenticação. O atributo de data\_expiracao é pedido pelo sistema.

---

## Para a criação do banco de dados

Abra o dbeaver, clique na tomadinha no canto superior esquerdo para criar uma nova conexão, selecione PostgreSQL. Porta: 5432, em caso de não funcionamento, use a porta 5433. Senha: aluno. Teste sua conexão, faça o download e avance.

Para abrir o script e começar o código, selecione `CTRL F3`.

---

## Códigos necessários para o desenvolvimento do banco

### Criar o banco de dados

Para criar o banco:

```
create database nome_banco;
```

Exemplo:

```
create database colegio_plural;
```

Lembre-se de sempre usar `_` para nomes “compostos”.

Para executar o comando, clique em qualquer local dentro do comando e clique ctrl enter ou selecione todo o código e clique na seta de play no canto superior equerdo.

### Criar uma tabela

Para criar uma tabela:

```
create table nome_tabela(
    atributos
);
```

Exemplo:

```
create table responsavel(
    id serial primary key,
    nome varchar(30) not null,
    cpf char(11) not null,
    telefone char(11)not null,
    email varchar(30)
);
```

Exemplo 2:

```
create table usuario(
    id serial primary key,
    nome varchar(30) not null,
    email varchar(30) not null,
    senha varchar(35) not null,
    data_expiracao date not null
);
```

### Criar uma tabela que contenha chave estrangeira

Para criar uma tabela que contenha chave estrangeira:

```
create table nome_tabela(
    atributos
);
```

Lembre-se de sempre garantir que a tabela a ser referenciada já esteja criada, no exemplo a seguir, é necessário que a tabela responsavel já tenha sido criada.

Exemplo:

```
create table aluno(
    id serial primary key,
    nome varchar(30) not null,
    data_nascimento date not null,
    cpf char(11) not null,
    responsavel_id int not null,
    foreign key (responsavel_id) references responsavel(id) on delete
    cascade on update cascade
);
```

O uso do “on delete cascade on update cascade” é necessário nas tabelas que possuem chave estrangeira para que caso um responsável (exemplo) seja apagado, automaticamente o banco apaga os alunos dependentes desse responsável. Além disso, se o responsável for atualizado (como seu ID), automaticamente o banco já atualiza as informações do responsável dentro dos alunos.

Depois de realizar a criação de todas as tabelas, comece a realizar os inserts necessários.

---

## Criação de INSERT nas tabelas

```
insert into nome_tabela (atributos)
values (inserção dos atributos);
```

Exemplo:

```
insert into responsavel (nome, cpf, telefone, email)
values ('julia', '12345678999', '24988010244', 'julia123@gmail.com'),
('maria', '09865423809','24992345981','mariaclre@gmail.com'),
('lara', '16909353789', '24988224435', 'flordeliz@gmail.com');
```

### Exemplo de insert com atributo senha

```
insert into usuario (nome, email, senha, data_expiracao)
values ('Felippe', 'lipelindo@gmail.com', md5('bull'), '2026-10-10'),
('Marcelo', 'marcelo123@gmail.com', md5('postgresql'), '2026-11-22'),
('Gustavo', 'gustavo456@gmail.com', md5('computador'), '2026-12-13');
```

### Exemplo de insert com chave estrangeira

```
insert into aluno (nome, data_nascimento, cpf, responsavel_id)
values ('lucas', '2010-04-08', '12290145767', 1),
('andressa', '2012-08-15', '09678234390', 2),
('helena', '2015-11-23', '15345698099', 3);
```

### Outra forma de fazer insert sem precisar colocar os atributos é:

```
insert into usuario
Values (1,'Felippe', 'lipelindo@gmail.com', md5('bull'), '2026-10-10');
```

O uso do md5 é necessário quando um dos atributos é a senha. Ele serve para fazer uma codificação dentro do sistema, transformando a sua senha em um código hash(um código de 32 caracteres). Na hora do login normalmente o sistema pega a senha digitada, calcula o MD5 novamente e compara:

```
Usuário digita: bull
        ↓
      MD5
        ↓
     mesmo hash?
        ↓
    SIM → senha correta
    NÃO → senha incorreta
```

Isso impede que a pessoa que tenha acesso ao banco veja sua senha diretamente. Ao invés de ver “bull” ela vai ver algo como c8f3...

---

## Comandos que podem ser necessários

### Select (utilizado para consultar os dados armazenados em uma tabela.)

Selecionar todos os atributos:

```
select * from responsavel;
```

Selecionar atributos específicos:

```
SELECT nome, email FROM responsavel;
```

Selecionar com condição:

```
SELECT nome, email
FROM responsavel
WHERE nome = 'julia';
```

### Update (utilizado para alterar informações que já existem na tabela.)

Atualizar somente um atributo:

```
UPDATE responsavel
SET telefone = '24999999999'
WHERE nome = 'julia';
```

Atualizar mais de um atributo:

```
UPDATE responsavel
SET telefone = '24988342255',
email = 'juju22@gmail.com'
WHERE nome = 'julia';
```

O “where” é muito importante, sem ele todos os responsáveis teriam dados atualizados e não somente o que desejamos.

### Delete (utilizado para excluir REGISTROS de uma tabela.)

Apagar todos os registros da tabela:

```
delete from responsavel;
```

Para apagar somente um registro, é necessário usar o WHERE, seja para apagar por nome, id ou outro atributo:

```
delete from responsavel
where nome = 'julia';
```

```
delete from responsavel
where id = 2;
```

### Drop table (utilizado para excluir uma tabela inteira.)

Ao utilizar o drop table, a tabela e os registros serão removidos:

```
drop table responsavel;
```

### Alter table (utilizado para modificar a estrutura de uma tabela que já existe.)

Adicionar um atributo:

```
alter table responsavel
add idade int;
```

Remover um atributo:

```
alter table responsavel
drop column idade;
```

Usamos o drop column para que tanto o atributo, quanto os registros sejam apagados.
