```sql

--PREPARE ROLES
CREATE ROLE ERP_ADMIN;
CREATE ROLE ERP_READ;

--PREPARE USERS
CREATE USER ERP IDENTIFIED BY ERP;
CREATE USER ERP_APP IDENTIFIED BY ERP_APP;
CREATE USER ERP_REPORTS IDENTIFIED BY ERP_READ;

--GRANT CREATE SESSION
grant connect,resource TO ERP_ADMIN;
GRANT CREATE SESSION TO ERP;
GRANT CREATE SESSION TO ERP_APP;
GRANT CREATE SESSION TO ERP_REPORTS;

--GRANT ROLES
GRANT CREATE TABLE TO ERP_ADMIN;
GRANT ERP_ADMIN TO ERP;
GRANT ERP_ADMIN TO ERP_APP;
GRANT ERP_READ TO ERP_REPORTS;

--SEQUENCES
CREATE SEQUENCE SEQ_CONTAS MINVALUE 999 MAXVALUE 9999999999999999 START WITH 999 INCREMENT BY 1 NOCACHE NOCYCLE;
CREATE SEQUENCE SEQ_CATEGORIAS_PRODUTO MINVALUE 999 MAXVALUE 9999999999999999 START WITH 999 INCREMENT BY 1 NOCACHE NOCYCLE;
CREATE SEQUENCE SEQ_PRODUTOS MINVALUE 999 MAXVALUE 9999999999999999 START WITH 999 INCREMENT BY 1 NOCACHE NOCYCLE;
CREATE SEQUENCE SEQ_UNIDADES_MEDIDA MINVALUE 999 MAXVALUE 9999999999999999 START WITH 999 INCREMENT BY 1 NOCACHE NOCYCLE;
CREATE SEQUENCE SEQ_TABELAS_PRECO MINVALUE 999 MAXVALUE 9999999999999999 START WITH 999 INCREMENT BY 1 NOCACHE NOCYCLE;


--PERMITIR INSERTS
ALTER USER ERP_APP QUOTA 100M ON USERS;
GRANT UNLIMITED TABLESPACE TO ERP_APP;

--TABELAS
CREATE TABLE USUARIOS(
   ID                  VARCHAR2(20 BYTE) NOT NULL,
   NOME                VARCHAR2(50 BYTE) NOT NULL,
   EMAIL               VARCHAR2(50 BYTE) NOT NULL,
   TELEFONE            VARCHAR2(11 BYTE),
   SEXO                CHAR(3 BYTE) ,
   DT_NASCIMENTO       DATE,
   SITUACAO            CHAR(1 BYTE) DEFAULT 'A' NOT NULL ,
   
   CONSTRAINT PK_USUARIOS PRIMARY KEY(ID),
   CONSTRAINT CKV_USUARIOS_SITUACAO CHECK (SITUACAO IN ('A', 'I')),
   CONSTRAINT CKV_USUARIOS_SEXO CHECK (SEXO IN ('MAS', 'FEM'))
   
);

CREATE TABLE CONTAS(
   ID                    NUMBER(19) NOT NULL,
   CNPJ                  VARCHAR2(14 BYTE),
   RAZAO_SOCIAL          VARCHAR2(125 BYTE),
   NOME_FANTASIA         VARCHAR2(50 BYTE),
   SITUACAO              CHAR(1 BYTE) DEFAULT 'A' NOT NULL,
   
   AUD_DH_CRIACAO        DATE DEFAULT SYSDATE,
   AUD_DH_ATUALIZACAO    DATE,
   AUD_CRIADO_POR        VARCHAR2(20) NOT NULL,
   AUD_VERSAO            NUMBER(11) DEFAULT 0 NOT NULL,
   
   CONSTRAINT PK_CONTAS PRIMARY KEY(ID),
   CONSTRAINT CKV_CONTAS_SITUACAO CHECK (SITUACAO IN ('A', 'I')),
   CONSTRAINT CKU_CONTAS_CNPJ UNIQUE (CNPJ)
   
);

CREATE TABLE CATEGORIAS_PRODUTO(
   ID                   NUMBER(19) NOT NULL,
   NOME                 VARCHAR2(50 BYTE),
   DESCRICAO            CLOB,
   SITUACAO             CHAR(1 BYTE) DEFAULT 'A' NOT NULL,
   
   AUD_DH_CRIACAO       DATE DEFAULT SYSDATE,
   AUD_DH_ATUALIZACAO   DATE,
   AUD_CRIADO_POR       VARCHAR2(20),
   AUD_VERSAO           NUMBER(11) DEFAULT 0,
   
   CONSTRAINT PK_CATEGORIAS_PRODUTO PRIMARY KEY(ID),
   CONSTRAINT CKV_CATEGORIAS_PRODUTO_SIT CHECK (SITUACAO IN ('A', 'I')),
   CONSTRAINT CKU_CATEGORIAS_PRODUTO_NOME UNIQUE (NOME)
   
);

CREATE TABLE PRODUTOS(
   ID                      NUMBER(19) NOT NULL,
   NOME                    VARCHAR2(50 BYTE) NOT NULL,
   DESCRICAO               CLOB,
   SITUACAO                CHAR(1 BYTE) DEFAULT 'A' NOT NULL,
   I_CATEGORIAS_PRODUTO    NUMBER(19) NOT NULL,

   AUD_DH_CRIACAO          DATE DEFAULT SYSDATE,
   AUD_DH_ATUALIZACAO      DATE,
   AUD_CRIADO_POR          VARCHAR2(20),
   AUD_VERSAO              NUMBER(11) DEFAULT 0 NOT NULL,
   
   CONSTRAINT PK_PRODUTOS PRIMARY KEY(ID),
   CONSTRAINT FK_PRODUTOS_I_CATEGORIAS_PRO FOREIGN KEY(I_CATEGORIAS_PRODUTO) REFERENCES CATEGORIAS_PRODUTO(ID),
   CONSTRAINT CKV_PRODUTOS_SITUACAO CHECK (SITUACAO IN ('A', 'I')),
   CONSTRAINT CKU_PRODUTOS_NOME UNIQUE (NOME)
  
);

CREATE TABLE UNIDADES_MEDIDA(
   ID                    NUMBER(19) NOT NULL,
   SIGLA                 VARCHAR2(14 BYTE),
   DESCRICAO             VARCHAR2(50 BYTE),
   SITUACAO              CHAR(1 BYTE) DEFAULT 'A' NOT NULL,
   
   AUD_DH_CRIACAO        DATE DEFAULT SYSDATE,
   AUD_DH_ATUALIZACAO    DATE,
   AUD_CRIADO_POR        VARCHAR2(20),
   AUD_VERSAO            NUMBER(11) DEFAULT 0,
   
   CONSTRAINT PK_UNIDADES_MEDIDA PRIMARY KEY(ID),
   CONSTRAINT CKV_UNIDADES_MEDIDA_SITUACAO CHECK (SITUACAO IN ('A', 'I'))
   
);

CREATE TABLE TABELAS_PRECO(
   ID                    NUMBER(19) NOT NULL,
   NOME                  VARCHAR2(50 BYTE) NOT NULL,
   DESCRICAO             CLOB,
   SITUACAO              CHAR(1 BYTE) DEFAULT 'A' NOT NULL,
   
   AUD_DH_CRIACAO        DATE DEFAULT SYSDATE,
   AUD_DH_ATUALIZACAO    DATE,
   AUD_CRIADO_POR        VARCHAR2(20),
   AUD_VERSAO            NUMBER(11) DEFAULT 0 NOT NULL,
   
   CONSTRAINT PK_TABELAS_PRECO PRIMARY KEY(ID),
   CONSTRAINT CKV_TABELAS_PRECO_NOME UNIQUE(NOME),
   CONSTRAINT CKV_TABELAS_PRECO_SITUACAO CHECK (SITUACAO IN ('A', 'I'))
);

CREATE TABLE TABELAS_PRECO_PRODUTOS(
   I_TABELAS_PRECO       NUMBER(19) NOT NULL,
   I_PRODUTOS            NUMBER(19) NOT NULL,
   I_UNIDADES_MEDIDA     NUMBER(19) NOT NULL,
   VLR_UNITARIO          NUMBER(16, 2) NOT NULL,
   QTD_DISPONIVEL        NUMBER(16, 2) DEFAULT 0,
   SITUACAO              CHAR(1 BYTE) DEFAULT 'A' NOT NULL,

   CONSTRAINT PK_TABELAS_PRECO_PRODUTOS PRIMARY KEY(I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA),
   CONSTRAINT FK_TAB_PRE_PROD_I_TAB_PREC FOREIGN KEY(I_TABELAS_PRECO) REFERENCES PRODUTOS(ID),
   CONSTRAINT FK_TAB_PRE_PROD_I_PRODUTOS FOREIGN KEY(I_PRODUTOS) REFERENCES PRODUTOS(ID),
   CONSTRAINT FK_TAB_PRE_PROD_I_UNID_MED FOREIGN KEY(I_UNIDADES_MEDIDA) REFERENCES UNIDADES_MEDIDA(ID),
   CONSTRAINT CKV_TAB_PRE_PROD_SITUACAO CHECK (SITUACAO IN ('A', 'I'))
);

--DOCUMENTACAO

COMMENT ON COLUMN USUARIOS.ID IS 'Identificador do usuário';
COMMENT ON COLUMN USUARIOS.NOME IS 'Nome do usuário';
COMMENT ON COLUMN USUARIOS.EMAIL IS 'E-mail do usuário';
COMMENT ON COLUMN USUARIOS.SEXO IS 'Sexo do usuário';
COMMENT ON COLUMN USUARIOS.DT_NASCIMENTO IS 'Data de nascimento do usuário';
COMMENT ON COLUMN USUARIOS.SITUACAO IS 'Situação do usuário';

COMMENT ON COLUMN CONTAS.ID IS 'Identificador do conta';
COMMENT ON COLUMN CONTAS.CNPJ IS 'CNPJ da conta';
COMMENT ON COLUMN CONTAS.RAZAO_SOCIAL IS 'Razão social da conta';
COMMENT ON COLUMN CONTAS.NOME_FANTASIA IS 'Nome fantasia da conta';
COMMENT ON COLUMN CONTAS.SITUACAO IS 'Situação do usuário';
COMMENT ON COLUMN CONTAS.AUD_DH_CRIACAO IS 'Data e hora de criação do registro';
COMMENT ON COLUMN CONTAS.AUD_DH_ATUALIZACAO IS 'Data e hora de atualização do registro';
COMMENT ON COLUMN CONTAS.AUD_CRIADO_POR IS 'Usuário que criou o registro';
COMMENT ON COLUMN CONTAS.AUD_VERSAO IS 'Versão do registro';

COMMENT ON COLUMN CATEGORIAS_PRODUTO.ID IS 'Identificador da categoria';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.NOME IS 'Nome da categoria';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.DESCRICAO IS 'Descrição da categoria';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.SITUACAO IS 'Situação da categoria';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.AUD_DH_CRIACAO IS 'Data e hora de criação do registro';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.AUD_DH_ATUALIZACAO IS 'Data e hora de atualização do registro';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.AUD_CRIADO_POR IS 'Usuário que criou o registro';
COMMENT ON COLUMN CATEGORIAS_PRODUTO.AUD_VERSAO IS 'Versão do registro';

COMMENT ON COLUMN PRODUTOS.ID IS 'Identificador do produto';
COMMENT ON COLUMN PRODUTOS.NOME IS 'Nome do produto';
COMMENT ON COLUMN PRODUTOS.DESCRICAO IS 'Descrição do produto';
COMMENT ON COLUMN PRODUTOS.SITUACAO IS 'Situação do produto';
COMMENT ON COLUMN PRODUTOS.I_CATEGORIAS_PRODUTO IS 'Código da categoria do produto';
COMMENT ON COLUMN PRODUTOS.AUD_DH_CRIACAO IS 'Data e hora de criação do registro';
COMMENT ON COLUMN PRODUTOS.AUD_DH_ATUALIZACAO IS 'Data e hora de atualização do registro';
COMMENT ON COLUMN PRODUTOS.AUD_CRIADO_POR IS 'Usuário que criou o registro';
COMMENT ON COLUMN PRODUTOS.AUD_VERSAO IS 'Versão do registro';

COMMENT ON COLUMN UNIDADES_MEDIDA.ID IS 'Identificador da unidade de medida';
COMMENT ON COLUMN UNIDADES_MEDIDA.SIGLA IS 'Sigla da unidade de medida';
COMMENT ON COLUMN UNIDADES_MEDIDA.DESCRICAO IS 'Descrição da unidade de medida';
COMMENT ON COLUMN UNIDADES_MEDIDA.SITUACAO IS 'Situação da unidade de medida';
COMMENT ON COLUMN UNIDADES_MEDIDA.AUD_DH_CRIACAO IS 'Data e hora de criação do registro';
COMMENT ON COLUMN UNIDADES_MEDIDA.AUD_DH_ATUALIZACAO IS 'Data e hora de atualização do registro';
COMMENT ON COLUMN UNIDADES_MEDIDA.AUD_CRIADO_POR IS 'Usuário que criou o registro';
COMMENT ON COLUMN UNIDADES_MEDIDA.AUD_VERSAO IS 'Versão do registro';

COMMENT ON COLUMN TABELAS_PRECO.ID IS 'Identificador da tabela de preço';
COMMENT ON COLUMN TABELAS_PRECO.NOME IS 'Nome da tabela de preço';
COMMENT ON COLUMN TABELAS_PRECO.DESCRICAO IS 'Descrição da tabela de preço';
COMMENT ON COLUMN TABELAS_PRECO.SITUACAO IS 'Situação da tabela de preço';
COMMENT ON COLUMN TABELAS_PRECO.AUD_DH_CRIACAO IS 'Data e hora de criação do registro';
COMMENT ON COLUMN TABELAS_PRECO.AUD_DH_ATUALIZACAO IS 'Data e hora de atualização do registro';
COMMENT ON COLUMN TABELAS_PRECO.AUD_CRIADO_POR IS 'Usuário que criou o registro';
COMMENT ON COLUMN TABELAS_PRECO.AUD_VERSAO IS 'Versão do registro';



--RODAR COM SYS OU OUTRO DBA
declare
cursor c1 is select table_name from DBA_TABLES TABLES WHERE TABLES.owner='ERP';
cmd varchar2(200);
begin
for c in c1 loop
cmd := 'GRANT SELECT ON ERP.'||c.table_name|| ' TO ERP_READ';
execute immediate cmd;
end loop;
end;

declare
cursor c1 is select table_name from DBA_TABLES TABLES WHERE TABLES.owner='ERP';
cmd varchar2(200);
begin
for c in c1 loop
cmd := 'GRANT SELECT,INSERT ON ERP.'||c.table_name|| ' TO ERP_ADMIN';
execute immediate cmd;
end loop;
end;


--CARGA DE DADOS
--USUARIOS
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO) VALUES ('bruno.luiz', 'Bruno Bitencourt', 'bruno.luiz@bth.com.br', '48991851245', 'MAS', '21/02/1996');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('fernando.boaglio', 'Fernando Boaglio', 'fernando.boaglio@fasatc.com.br', '47999951442', 'MAS', '04/04/1987');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO, SITUACAO)  VALUES ('elder.moraes', 'Elder Moraes', 'elder.moraes@fasatc.com.br', '48991471236', 'MAS', '15/04/1960', 'I');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('edson.yanaga', 'Edson Yanaga', 'edson.yanaga@fasatc.com.br', '47999951442', 'MAS', '04/04/1987');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('reza_rahman', 'Reza Rahman', 'reza_rahman@bol.com.br', '47999851411', 'MAS', '04/11/1976');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('thjanssen123', 'Thorben Janssen', 'thjanssen123@gujava.com', '7498541247', 'MAS', '04/04/1999');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('delabassee', 'David Delabassée', 'delabassee@gujava.com', '4891968512', 'MAS', '11/02/1954');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('otaviojava', 'Otávio Santana', 'otaviojava@gujava.com', '47999951442', 'MAS', '04/03/1988');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('gegastaldi', 'George Gastaldi', 'gegastaldi@gujava.com', '4834628732', 'MAS', '30/01/1998');
INSERT INTO USUARIOS (ID, NOME, EMAIL, TELEFONE, SEXO, DT_NASCIMENTO)  VALUES ('rcandidosilva', 'Rodrigo Silva', 'rcandidosilva@gujava.com', '4791995437', 'MAS', '01/12/1987');

--CONTAS
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '02916265010060' , 'Amazon Web Services Inc' , 'Amazon Web Services', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '64192669010005' , 'Gartner Group' , 'Gartner', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '10310730010042' , 'Oracle Corporation' , 'Oracle', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '10570115010075' , 'SAP SE' , 'SAP', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '64270952010008' , 'Sun Microsystems' , 'Sun Microsystems', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '24161853100145' , 'Salesforce.com' , 'Salesforce', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '24161853100579' , 'International Business Machines' , 'IBM', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '71260723010078' , 'Lenovo' , 'Lenovo', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '07545982010046' , 'LinkedIn' , 'LinkedIn', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '02006487000226' , 'Dell' , 'Dell', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '08228749010000' , 'Hewlett-Packard' , 'HP', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '05004855000538' , 'Acer' , 'Acer', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '09583779010090' , 'ASUS' , 'ASUS', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '07986737010074' , 'Seiko Epson Corporation' , 'Epson', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '86574365001226' , 'Facebook Inc' , 'Facebook', 'bruno.luiz' );
INSERT INTO CONTAS (ID, CNPJ, RAZAO_SOCIAL, NOME_FANTASIA, AUD_CRIADO_POR) VALUES (SEQ_CONTAS.NEXTVAL, '10209882010053' , 'WhatsApp Inc (Facebook Inc)' , 'WhatsApp', 'bruno.luiz' );

--CATEGORIAS_PRODUTO
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'LINUX PLATFORMS' , 'A stable, proven foundation that’s versatile enough for rolling out new applications, virtualizing environments, and creating a secure hybrid cloud' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'RED HAT JBOSS MIDDLEWARE' , 'A fully certified Java™ EE 7 container that includes everything needed to build, run, and manage Java-based services.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'VIRTUALIZATION PLATFORM' , 'Complete enterprise virtualization management for servers and desktops on the same infrastructure.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'CLOUD COMPUTING' , 'A single-subscription offering that lets you build and manage an open, private Infrastructure-as-a-Service (IaaS) cloud and ease your way into a highly scalable, public-cloud-like infrastructure based on OpenStack®' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'STORAGE' , 'Open, software-defined file storage that combines reliable Red Hat software with x86 commodity hardware, eliminating the need for high-cost proprietary storage systems.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'MOBILE PLATFORM' , 'A platform that offers centralized control of security and back-end integration, collaborative app development, and a range of deployments that increase the speed of app integration with enterprise systems and delivery.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'MANAGEMENT' , 'The easiest way to manage your Red Hat infrastructure for efficient and compliant IT operations. Lets you establish trusted content repos and processes that help you build a standards-based, secure Red Hat environment.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'Red Hat Open Innovation Labs' , 'Speed up your next application development project. We’ll help your team make use of innovative open source technologies, rapidly build prototypes, do DevOps, and adopt agile. Get started immediately with hands-on guidance from Red Hat’s subject matter experts.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'Training' , 'Improve your in-demand technology skills.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'Certification' , 'Make yourself more marketable by certifying your skill level.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'Consulting' , 'Team up with us to collaboratively tackle your technology challenges.' , 'bruno.luiz' );
INSERT INTO CATEGORIAS_PRODUTO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_CATEGORIAS_PRODUTO.NEXTVAL, 'Support' , 'Get proactive, focused help from the best engineers in the industry.' , 'bruno.luiz' );

--PRODUTOS
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 1, 'Red Hat Enterprise Linux' , 'Red Hat® Enterprise Linux® delivers military-grade security, 99.999% uptime, support for business-critical workloads, and so much more. Ultimately, the platform helps you reallocate resources from maintaining the status quo to tackling new challenges. It\'s just 1 reason why more than 90% of Fortune Global 500 companies use Red Hat products and solutions.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 1, 'Red Hat Satellite' , 'As your Red Hat® environment continues to grow, so does the need to manage it to a high standard of quality. Red Hat Satellite is an infrastructure management product specifically designed to keep Red Hat Enterprise Linux® environments and other Red Hat infrastructure running efficiently, properly secured, and compliant with various standards.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 1, 'Red Hat OpenStack Platform' , 'Your IT department is being challenged to react faster to growing customer demands. If you’re like many IT organizations, you’re exploring Infrastructure-as-a-Service (IaaS) private clouds, like OpenStack®, to swiftly deploy and scale IT infrastructure. But not just any OpenStack cloud can deliver the demands of a production-scale environment and meet your performance, scalability, and security standards. Fortunately, Red Hat® OpenStack Platform does all this and more.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 2, 'Red Hat JBoss Enterprise Application Platform' , 'Red Hat® JBoss® Enterprise Application Platform (JBoss EAP) delivers enterprise-grade security, performance, and scalability in any environment. Whether on-premise; virtual; or in private, public, or hybrid clouds, JBoss EAP can help you deliver apps faster, everywhere.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 2, 'Red Hat JBoss Web Server' , 'Your development team loves the flexibility of open source web servers and components, like Apache and Tomcat. But as your company grows, it demands that you move to a more secure, more stable environment with the enterprise-level features you need to deliver large-scale websites and lightweight web apps.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 2, 'Red Hat JBoss Data Grid' , 'Great user experience is ever-more dependent on application performance and quality. Even a few seconds delay can mean the difference between success and failure for a new business initiative. To capitalize on customer engagement, you need to know your customers and provide targeted offers that prompt them to interact in real time. Data bottlenecks are becoming more common as organizations need to process larger volumes, greater varieties, and a higher velocity of data to meet customer expectations and deliver personalized data-driven engagement.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 2, 'Red Hat JBoss Developer Studio' , 'Red Hat® JBoss® Developer Studio provides superior support for your entire development life cycle in one tool. It\'s a certified Eclipse-based integrated development environment (IDE) for developing, testing, and deploying rich web apps, mobile web apps, transactional enterprise apps, and microservices.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 3, 'Red Hat Virtualization' , 'Virtualization can vastly improve efficiency, free up resources, and cut costs. But you can\'t afford to sacrifice performance, security, and existing investments. And as you plan for future technologies like cloud and containers, it\'s important to build common services that use your virtualization investment while avoiding vendor lock-in. So what if you could virtualize both your servers and workstations, manage them from 1 simple interface, and build a foundation for future technologies on your terms—without compromise?' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 4, 'Red Hat Cloud Infrastructure' , 'Red Hat® Cloud Infrastructure is a combination of tightly integrated Red Hat technologies that lets you build and manage an open, private Infrastructure-as-a-Service (IaaS) cloud—at a much lower cost than alternative solutions. You can deploy any combination of these components in whatever way you need.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 4, 'Red Hat CloudForms' , 'Managing a complex, hybrid IT environment can require multiple management tools, redundant policy implementations, and extra staff to handle the operations. Red Hat® CloudForms simplifies IT, providing unified management and operations in a hybrid environment.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 4, 'Red Hat OpenShift' , 'Red Hat® OpenShift is a container application platform that brings docker and Kubernetes to the enterprise. Regardless of your applications architecture, OpenShift lets you easily and quickly build, develop, and deploy in nearly any infrastructure, public or private. Whether it’s on-premise, in a public cloud, or hosted, you have an award-winning platform to get your next big idea to market ahead of your competition.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 5, 'Red Hat Gluster Storage' , 'Red Hat® Gluster Storage is a software-defined storage (SDS) platform designed to handle the requirements of traditional file storage—high-capacity tasks like backup and archival as well as high-performance tasks of analytics and virtualization.But unlike traditional storage systems, Red Hat Gluster Storage isn’t rigid and expensive. It easily scales across bare metal, virtual, container, and cloud deployments' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 5, 'Red Hat Ceph Storage' , 'Red Hat® Ceph Storage is a massively scalable, programmable storage platform that supports cloud infrastructure, media repositories, backup and restore systems, and data lakes. It can free you from the expensive lock of proprietary, hardware-based storage solutions; consolidate labor and storage costs into 1 versatile solution; and introduce cost-effective scalability that modern workloads require.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 6, 'Red Hat Mobile Application Platform' , 'Red Hat Mobile Application Platform supports an agile approach to developing, integrating, and deploying enterprise mobile applications—whether native, hybrid, or on the web. The platform supports collaborative development across multiple teams and projects with a wide variety of leading tool kits and frameworks. You gain central control over security and policy management, the ease of Mobile Backend-as-a-Service (MBaaS) integration with enterprise systems, and a choice of cloud deployment options.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 7, 'Red Hat Satellite' , 'As your Red Hat® environment continues to grow, so does the need to manage it to a high standard of quality.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 7, 'Red Hat Insights' , 'Predictive analytics helps you see what’s happening in your IT environment. It also allows staff to fix technical issues before they impact your environment—avoiding costly downtime.' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 8, 'Red Hat Red Hat Open Innovation Labs' , '' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 9, 'Red Hat Training' , '' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 10, 'Red Hat Certification' , '' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 11, 'Red Hat Consulting' , '' , 'bruno.luiz' );
INSERT INTO PRODUTOS (ID, I_CATEGORIAS_PRODUTO, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_PRODUTOS.NEXTVAL, 12, 'Red Hat Support' , '' , 'bruno.luiz' );

--UNIDADES_MEDIDA
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'UN' , 'Unity', 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'SU' , 'Concurrent user', 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'GR' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'L', 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'KG' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'RL' , 'bruno.luiz' );             
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'AP' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'CP' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'SE' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'EM' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'TL' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'TP' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'RF' , 'bruno.luiz' );
INSERT INTO UNIDADES_MEDIDA (ID, SIGLA, AUD_CRIADO_POR) VALUES (SEQ_UNIDADES_MEDIDA.NEXTVAL, 'CJ' , 'bruno.luiz' );

--TABELAS_PRECO
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Hosting - JEE - Oracle RAC - Storage' , 'n database computing, Oracle Real Application Clusters (RAC) — an option[1] for the Oracle Database software produced by Oracle Corporation and introduced in 2100 with Oracle9i — provides software for clustering and high availability in Oracle database environments. Oracle Corporation includes RAC with the Standard Edition, provided the nodes are clustered using Oracle Clusterware.', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Oracle - DBaaS' , 'A cloud database is a database that typically runs on a cloud computing platform, access to it is provided as a service.', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'MongoDB - DBaaS' , 'Designed to support every stage of your application, MongoDB Atlas is the easiest way to take advantage of the fastest-growing database. Try it now for free — no credit card required. Or easily import existing workloads with a live migration.', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'WildFly as a Service on Linux' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Amazon S3' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Amazon RDS' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'AWS Lambda' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'WORDPRESS + S3' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'DevOps' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Archiving' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Backup and Recovery' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Websites' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'E-Commerce' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Business Applications' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Internet of Things' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Federal Government' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Healthcare' , 'Whether you are part of a community hospital or global pharmaceutical company, AWS helps you add agility, improve collaboration, and makes it easier to experiment and incorporate new technological innovation. Organizations across the healthcare and life sciences industry are using AWS for everything from basic storage to clinical information systems.', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Nonprofit' , 'Tens of thousands of nonprofits and NGOs worldwide use AWS to build highly available, scalable websites, host core business and employee-facing systems, and manage donor outreach and fundraising efforts – keeping them mission centric. This helps nonprofit organizations pave the way for innovation and, ultimately, make the world a better place through technology.', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'Financial Service' , '', 'usuario.padrao');
INSERT INTO TABELAS_PRECO (ID, NOME, DESCRICAO, AUD_CRIADO_POR) VALUES (SEQ_TABELAS_PRECO.NEXTVAL, 'High Performance Computing' , 'High Performance Computing (HPC) allows scientists and engineers to solve complex, compute-intensive and data-intensive problems. HPC applications often require high network performance, fast storage, large amounts of memory, very high compute capabilities, or all of these. AWS allows you to increase the speed of research and reduce time-to-results by running high performance computing in the cloud and scaling to larger numbers of parallel HPC tasks than would be practical in most on-premise HPC environments. AWS helps to reduce costs by providing CPU, GPU, and FPGA servers on-demand, optimized for specific applications, and without the need for large capital investments. You have access to a full-bisection, high bandwidth network for tightly-coupled, IO-intensive and storage-intensive workloads, which enables you to scale out across thousands of cores for faster results.', 'usuario.padrao');

--TABELAS_PRECO_PRODUTOS
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 1, 1, 510.87, 500);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 2, 1, 871.91, 300);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 3, 1, 250.17, 120);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 4, 1, 900.99, 500);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 5, 1, 1500.45, 1000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 6, 1, 510.33, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 7, 1, 545.14, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 8, 1, 800.00, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 9, 1, 199.00, 255);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 10, 1, 255.97, 100);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (1, 11, 1, 79.99, 10);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (2, 1, 1, 51897.87, 50);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (2, 2, 1, 3269.87, 75);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (2, 3, 1, 487.87, 1500);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (3, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (7, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (4, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (5, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (6, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (8, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (9, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (10, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 8, 1, 487.87, 60000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 7, 1, 154.87, 15000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (11, 9, 1, 999.99, 50000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 1, 1, 8000.87, 5000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 5, 1, 8478.87, 2540);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 6, 1, 5115.87, 3000);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 8, 1, 487.87, 423423);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 7, 1, 154.87, 6456);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (12, 9, 1, 999.99, 645645);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 1, 1, 8000.87, 64564);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 5, 1, 8478.87, 234543);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 6, 1, 5115.87, 432423);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 8, 1, 487.87, 5435);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 7, 1, 154.87, 6456);
INSERT INTO TABELAS_PRECO_PRODUTOS (I_TABELAS_PRECO, I_PRODUTOS, I_UNIDADES_MEDIDA, VLR_UNITARIO, QTD_DISPONIVEL) VALUES (13, 9, 1, 999.99, 848);



--UPDATES

--1
UPDATE USUARIOS SET SITUACAO = 'I' WHERE ID = 'fernando.boaglio';

--2
UPDATE USUARIOS SET EMAIL = CONCAT(ID, '@gujava.com') WHERE EMAIL LIKE '%interjug.com';

--3
UPDATE USUARIOS SET SITUACAO = 'A' WHERE ID = 'elder.moraes';

--4
UPDATE TABELAS_PRECO SET SITUACAO = 'I' WHERE NOME IN ('WORDPRESS + S3', 'AWS Lambda', 'Amazon RDS');

--5
UPDATE TABELAS_PRECO_PRODUTOS SET VLR_UNITARIO = (VLR_UNITARIO * 1.3) WHERE QTD_DISPONIVEL < 500;
UPDATE TABELAS_PRECO_PRODUTOS SET VLR_UNITARIO = (VLR_UNITARIO * 1.15) WHERE QTD_DISPONIVEL >= 500 AND QTD_DISPONIVEL < 1000;
UPDATE TABELAS_PRECO_PRODUTOS SET VLR_UNITARIO = (VLR_UNITARIO * 1.075) WHERE QTD_DISPONIVEL >= 1000;


--CONSULTAS

--1
SELECT P.NOME NOME_PRODUTO, TO_CHAR(SUM(QTD_DISPONIVEL),'FM999G999G999D90', 'nls_numeric_characters='',.''') QTD_TOTAL, TO_CHAR(SUM(QTD_DISPONIVEL * VLR_UNITARIO),'FM999G999G999D90', 'nls_numeric_characters='',.''') VALOR_TOTAL FROM TABELAS_PRECO_PRODUTOS T INNER JOIN PRODUTOS P ON P.ID = T.I_PRODUTOS GROUP BY P.NOME ORDER BY SUM(QTD_DISPONIVEL) DESC; 

--2
SELECT P.NOME NOME_PRODUTO, COUNT(P.ID) FROM TABELAS_PRECO_PRODUTOS T INNER JOIN PRODUTOS P ON P.ID = T.I_PRODUTOS GROUP BY P.NOME ORDER BY COUNT(P.ID) DESC; 

--3
SELECT ID CODIGO, NOME FROM CATEGORIAS_PRODUTO  WHERE NOME != UPPER(NOME);
```
