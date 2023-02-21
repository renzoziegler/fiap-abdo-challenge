# Challenge Final - 6ABDO

Você deve ter recebido em seu e-mail cadastrado na FIAP uma mensagem como esta abaixo, com seu acesso à AWS Academy:

![Email de boas-vindas da AWS Academy](/images/email_canvas.png "Email de boas-vindas da AWS Academy")

*(caso não tenha recebido, entre em contato com o professor ou mentor para reenviar o convite)*

Ao clicar no botão de **Get Started**, você será redirecionado para a cadastro e login na plataforma da AWS Academy, que terá disponível para nosso Challenge o **AWS Academy Learner Lab"":

![Login na AWS Academy](/images/academy_login.png "Login na AWS Academy")

O Learner Lab nos dá a oportunidade de utilizar a AWS para criar os recursos para nosso challenge. Ao clicar em **Modules** no menu à esquerda, e depois em **Learner Lab** na tela que surgir, seremos levados à tela abaixo:

![Área de Modulos](/images/academy_vocareum.png "Área de Módulos")

Um terminal para sua sessão na AWS surgirá, junto com um tutorial de como utilizar o Learner Lab. Na barra superior, há um botão para iniciar a sessão da AWS (*Start Lab*). Clique nele, e espere o semáforo da AWS (na barra superior à esquerda) sair de vermelho para amarelo, e depois para verde. Quando isso acontecer, clique no semáforo, e você será levado para a conta de sua sessão na AWS, como abaixo:

![Busca de RDS no AWS Console](/images/aws_console_rds.png "Busca de RDS no AWS Console")

Agora você tem acesso à AWS para criar os recursos necessários para seu Challenge. Como conversamos em nossas *lives*, uma das fontes de dados para ingerirmos em nossa arquitetura de dados deve ser um banco de dados Postgres, com dados referentes a casos de COVID, baseados nos arquivos disponíveis em [Coronavirus Records Dataset: 2021](https://www.kaggle.com/datasets/iamsouravbanerjee/covid19-dataset-world-and-continent-wise?select=All+the+Countries+%28Others%29.txt). *(Os arquivos também estão disponíveis neste repositório)*

Para isso, vamos criar uma base de dados em Postgres, carregar o dataset em uma tabela, e com isso os dados estarão disponíveis para fazer a ingestão de dados. Comece buscando por RDS nos recursos da AWS, e clicando na opção de RDS acima, você será levado ao Dashboard do RDS:

![RDS Dashboard no AWS Console](/images/aws_console_rds_dashboard.png "RDS Dashboard no AWS Console")

Clique em **Criar banco de dados**, ou em **Bancos de dados** no menu à esquerda para ir à tela abaixo, onde estão listados os bancos já criados (a lista, agora, deve estar vazia):

![Criar banco no RDS](/images/aws_console_rds_create.png "Criar banco no RDS")

Clique em **Criar banco de dados** para criar seu primeiro banco no serviço RDS, e siga a recomendação de parâmetros abaixo: *(a lista de parâmetros recomendada abaixo é apenas uma sugestão, caso tenha interesse, conhecimento e curiosidade, pode seguir sua própria configuração - teste outras possibilidades, caso tenha alguma dúvida chame o professor ou mentor no Teams ou em uma das sessões de mentoria)*

- Criação Padrão
- Selecione Tipo de Banco PostgreSQL
- Versão do Mecanismo: PostgreSQL 13.7-R1
- Modelos: nível gratuito (para não consumir nenhum recurso, mas lembre-se de que temos USD100 disponíveis para nosso challenge)
- Configurações
    - Identificador da instância de banco de dados: nome único para identificar o banco criado
- Configurações de credenciais
    - Nome do usuário principal: nome do usuário admin, pode manter postgres
    - Senha principal: defina uma senha FORTE e guarde-a para usar posteriormente
- Configuração da instância
    - Escolha uma das opções de hardware, como o volume de uso será baixo, é possível selecionar qualquer opção disponível
- Armazenamento
    - SSD de uso geral
    - 20GiB para nosso caso é mais que suficiente
- Recursos de computação
    - Não se conectar a um recurso de computação do EC2
- Tipo de rede
    - IPv4
- Nuvem privada virtual (VPC)
    - manter a opção default
- Grupo de sub-redes de banco de dados
    - manter a opção selecionada
- Acesso público
    - Sim (para conseguirmos acessar o banco remotamente, é importante para nosso case, porém em um banco de dados em ambiente produtivo esta opção não é recomendável)
- Grupo de segurança de VPC (firewall)
    - Selecionar existente (default) - mais à frente iremos configurá-lo
- Zona de disponibilidade
    - sem preferência
- Autenticação de banco de dados
    - Autenticação de senha e do Banco de Dados do IAM (para utilizar o IAM para conexão na fase de ingestão)
- Desativar o Performance Insights
- Configuração Adicional
    - Opções de banco de dados
        - Nome do banco de dados inicial: fiap
        - Grupo de parâmetros do banco de dados: default.postgres13
- Desabilitar backups automáticos e criptografia
- Manter as demais opções com a escolha padrão

Ao final, clique em **Criar banco de dados**.

Assim que o banco estiver criado, você deve visualizar uma tela parecida com esta:
![Banco no RDS criado](/images/aws_console_created.png "Banco no RDS criado")

Nesta tela, você consegue visualizar os parâmetros para criar uma conexão ao banco de dados, como por exemplo o **hostname** (no parâmetro *Endpoint*) e **Porta**. O banco de dados a conectar é o que você criou e definiu na configuração para criação do banco (aqui sugerido com o nome de **fiap**), assim como o nome de usuário e senha.

![Regras de segurança de conexão](/images/aws_console_seguranca.png "Regras de segurança e conexão")

Para permitir o acesso remoto, na área de **Segurança e conexão**, clique no link do **Grupo de segurança da VPC** listado, assim você será levado para a seguinte tela:

![Grupos de segurança](/images/aws_console_security_group.png "Grupos de segurança")

Na parte de baixo da tela, no Menu superior, clique em **Regras de Entrada**. As regras serão listadas - clique em **Editar regras de entrada**:

![Regras de entrada](/images/aws_console_inbound_rules.png "Regras de entrada")

Aqui você consegue definir as regras de rede, que permitem acessar o banco de dados. No nosso caso, vamos adicionar uma regra, selecionando o *Tipo* como PostgreSQL (para habilitar a porta padrão do banco Postgres: 5432), e na *Origem*, digite 0.0.0.0/0, para permitir o acesso originário de qualquer endereço de IP. Na sequência, clique em **Salvar Regras**.

Esta configuração permite que qualquer máquina, de qualquer lugar, acesse o nosso banco pela porta 5432. Esta regra é válida para nosso challenge, porém perigosa para bancos em uso no nosso dia-a-dia, por expor e permitir acessos indevidos. Na vida real, devemos definir estas regras de forma cuidadosa, para limitar os acessos, portas e IPs somente a acessos devidos, e dentro do nosso contexto de aplicação.

Com o banco criado, chegou a hora de acessá-lo, e de criar a tabela e popular com os dados que precisaremos para fazer a ingestão de dados de nosso challenge.

Usando a ferramenta cliente SQL de sua preferência (neste exemplo, utilizaremos o dBeaver), crie uma conexão com o banco criado, na mesma forma que criamos aqui, utilizando os parâmetros definidos na criação do banco:

![Conexão do dBeaver](/images/dbeaver_connection.png ",Conexão do dBeaver")

Com a conexão criada, use o [Script SQL](https://github.com/renzoziegler/fiap-abdo-challenge/blob/main/script_covid.sql) deste repositório para criar e popular a tabela.

Com isso, você tem seu banco criado e populado. Agora é com você! Defina sua arquitetura, e use sua conta na AWS Academy para criar os recursos necessários para seu pipeline de dados! Lembre-se: quaisquer dúvidas, entre em contato com o professor e o mentor pelo Teams, ou em uma das sessões de mentoria a serem agendadas. Boa sorte!!