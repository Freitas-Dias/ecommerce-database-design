# 🛒 E-Commerce & Marketplace Database Design (MySQL)

## 📌 Descrição do Projeto

Este projeto consiste na modelagem conceitual, lógica e física de um banco de dados relacional para um sistema de **E-commerce com suporte a Marketplace**, desenvolvido em **MySQL**. 

A arquitetura foi desenvolvida aplicando boas práticas de modelagem de dados e regras rigorosas de normalização até a **Terceira Forma Normal (3FN)**. O sistema contempla a gestão completa de produtos, vendedores terceiros (marketplace), estoques, fornecedores, controle de clientes com especialização para Pessoa Física (PF) e Pessoa Jurídica (PJ), múltiplos meios de pagamento e rastreabilidade logística de entregas.

# 🛒 Projeto de Modelagem de Banco de Dados para E-commerce (MySQL)

[![MySQL](https://img.shields.io/badge/MySQL-00000F?style=for-the-badge&logo=mysql&logoColor=white)](#)
[![Database Design](https://img.shields.io/badge/Modelagem-Relacional_3FN-blue?style=for-the-badge)](#)
[![GitHub Portfolio](https://img.shields.io/badge/Portfolio-Data_Science-green?style=for-the-badge)](#)

Este repositório contém o projeto conceitual, lógico e físico do banco de dados relacional para um sistema de **E-commerce e Marketplace**, desenvolvido em **MySQL**. 

O projeto foi construído do zero, evoluindo através de revisões iterativas de arquitetura para garantir a integridade dos dados, escalabilidade, conformidade com as Formas Normais (3FN) e suporte eficiente tanto para operações transacionais (OLTP) quanto para futuras análises de dados (OLAP/Data Science).

---

## 📐 1. Contextualização e Regras de Negócio

O objetivo principal foi modelar um ecossistema completo de e-commerce que suportasse vendas próprias e vendas por terceiros (Marketplace), além de fluxos de pagamentos e logística.

### Regras de Negócio Solicitadas:

#### 📦 Produto, Fornecedores e Marketplace
* Os produtos são comercializados por uma plataforma única, mas podem ter **vendedores terceiros (Marketplace)** distintos.
* Cada produto possui um **fornecedor associado**.
* Um ou mais produtos podem compor um único **pedido**.
* O estoque precisa de ser rastreado por local de armazenamento físico.

#### 👥 Cliente e Especialização (PF / PJ)
* **Exclusividade de Perfil:** Uma conta de cliente pode ser Pessoa Física (PF) ou Pessoa Jurídica (PJ), mas **nunca ambas simultaneamente**.
* **Frete:** O endereço do cliente é a base para o cálculo dinâmico do frete.
* **Histórico:** Um cliente pode realizar múltiplos pedidos.

#### 💳 Formas de Pagamento
* Um cliente pode ter **mais de uma forma de pagamento (cartões/métodos) cadastrada** na sua conta.
* Um pedido pode ser pago utilizando uma ou mais formas de pagamento associadas.
* Em caso de cancelamento do pedido, o histórico da transação é preservado para fins financeiros e de auditoria (sem eliminação direta de registos).

#### 🚚 Pedido e Logística de Entrega
* Os pedidos contêm status da compra, endereço e informações financeiras.
* Um pedido pode ser cancelado sem perda de rastreabilidade.
* **Logística Desacoplada:** A entrega é modelada como uma **entidade separada do Pedido**, contendo status de envio e **código de rastreio único**.

---

## 🛠️ 2. Explicação Didática da Solução e Decisões de Arquitetura

A modelagem passou por um processo contínuo de refinamento para corrigir gargalos clássicos de bancos de dados relacionais. Abaixo estão as principais decisões técnicas aplicadas:

### A) Especialização 1:1 para Pessoa Física e Jurídica (Herança de Tabela)
Para resolver a regra de que um cliente só pode ser PF ou PJ sem gerar colunas nulas (`NULL`) nem duplicar dados de contacto, aplicou-se o padrão de **Tabela Base com Especialização**:
* **`Cliente` (Tabela Pai):** Armazena dados comuns como `idCliente`, `Nome`, `Endereco`, `Email` e `Tipo_de_Cliente ('PF', 'PJ')`.
* **`Cliente_PF` e `Cliente_PJ` (Tabelas Filhas):** Têm uma relação $1:1$ com a tabela pai. A chave `Cliente_idCliente` atua simultaneamente como **Chave Primária (PK) e Chave Estrangeira (FK)**.
  * `Cliente_PF` guarda exclusivamente `CPF`, `RG` e `Data_de_Nascimento`.
  * `Cliente_PJ` guarda exclusivamente `CNPJ`, `Razao_Social`, `Nome_Fantasia` e `Inscricao_Estadual`.

### B) Preservação de Histórico Financeiro
Em e-commerces, o preço de um produto varia ao longo do tempo. Se a consulta do valor de uma venda passada dependesse apenas da tabela `Produto`, os relatórios financeiros do passado mudariam sempre que o preço do produto fosse atualizado.
* **Solução:** O valor unitário no momento exato da compra foi fixado na tabela associativa `Relacao_de_produto_por_pedido` (`Valor_Unitario`).
* **Precisão Decimal:** Todos os campos monetários (`Preco`, `Valor_Unitario`, `Frete`, `Valor`) foram configurados como `DECIMAL(10,2)` para evitar erros de ponto flutuante comuns em tipos `FLOAT` ou `INT`.

### C) Desacoplamento da Entrega e Pagamentos
* **`Entrega`:** Criada como uma entidade $1:N$ ligada ao `Pedido`. Isso permite que o pedido seja criado antes do envio e suporta atualizações contínuas de rastreamento (`Codigo_Rastreio`, `Status_Entrega`, `Datas_de_controle`).
* **`Forma_de_Pagamento`:** O cliente pode registar $N$ cartões/meios. A tabela relacional `Forma_de_Pagamento_do_Pedido` vincula o pagamento transacional ao pedido efetuado.

---

## 📊 3. Resultado Final: Diagrama de Entidade-Relacionamento (DER)

Abaixo está a representação visual final da arquitetura do banco de dados, totalmente validada até à **Terceira Forma Normal (3FN)**:

![Diagrama do Banco de Dados de E-commerce](./Projeto%20de%20E-commerce.png)

---

## 💻 4. Script de Criação do Banco de Dados (DDL)

Caso queiras replicar a estrutura no teu ambiente local MySQL, executa o script SQL abaixo:

```sql
CREATE DATABASE IF NOT EXISTS ecommerce;
USE ecommerce;

-- 1. Tabela Pai: Cliente
CREATE TABLE Cliente (
    idCliente INT AUTO_INCREMENT PRIMARY KEY,
    Nome VARCHAR(45) NOT NULL,
    Tipo_de_Cliente ENUM('PF', 'PJ') NOT NULL,
    Endereco VARCHAR(45) NOT NULL,
    Email VARCHAR(45) NOT NULL UNIQUE
);

-- 1.1 Especialização: Cliente PF (Relacionamento 1:1)
CREATE TABLE Cliente_PF (
    Cliente_idCliente INT PRIMARY KEY,
    CPF VARCHAR(45) NOT NULL UNIQUE,
    RG VARCHAR(45),
    Data_de_Nascimento DATE,
    CONSTRAINT fk_cliente_pf FOREIGN KEY (Cliente_idCliente) 
        REFERENCES Cliente(idCliente) ON DELETE CASCADE
);

-- 1.2 Especialização: Cliente PJ (Relacionamento 1:1)
CREATE TABLE Cliente_PJ (
    Cliente_idCliente INT PRIMARY KEY,
    CNPJ VARCHAR(45) NOT NULL UNIQUE,
    Razao_Social VARCHAR(45) NOT NULL,
    Nome_Fantasia VARCHAR(45),
    Inscricao_Estadual VARCHAR(45),
    CONSTRAINT fk_cliente_pj FOREIGN KEY (Cliente_idCliente) 
        REFERENCES Cliente(idCliente) ON DELETE CASCADE
);

-- 2. Forma de Pagamento
CREATE TABLE Forma_de_Pagamento (
    idForma_de_Pagamento INT AUTO_INCREMENT PRIMARY KEY,
    Cliente_idCliente INT NOT NULL,
    Tipo_de_pagamento VARCHAR(45) NOT NULL,
    CONSTRAINT fk_forma_pagamento_cliente FOREIGN KEY (Cliente_idCliente) 
        REFERENCES Cliente(idCliente) ON DELETE CASCADE
);

-- 3. Produto
CREATE TABLE Produto (
    idProduto INT AUTO_INCREMENT PRIMARY KEY,
    Categoria VARCHAR(45) NOT NULL,
    Descricao VARCHAR(45) NOT NULL,
    Valor DECIMAL(10,2) NOT NULL
);

-- 4. Fornecedor
CREATE TABLE Fornecedor (
    idFornecedor INT AUTO_INCREMENT PRIMARY KEY,
    Razao_Social VARCHAR(45) NOT NULL,
    Nome_Fantasia VARCHAR(45),
    CNPJ VARCHAR(45) NOT NULL UNIQUE,
    Inscricao_Estadual VARCHAR(45)
);

-- 5. Disponibilizando produto (Fornecedor x Produto)
CREATE TABLE Disponibilizando_produto (
    Fornecedor_idFornecedor INT NOT NULL,
    Produto_idProduto INT NOT NULL,
    Preco DECIMAL(10,2) NOT NULL,
    Quantidade_de_venda_por_vendedor INT DEFAULT 0,
    PRIMARY KEY (Fornecedor_idFornecedor, Produto_idProduto),
    CONSTRAINT fk_disp_prod_fornecedor FOREIGN KEY (Fornecedor_idFornecedor) 
        REFERENCES Fornecedor(idFornecedor),
    CONSTRAINT fk_disp_prod_produto FOREIGN KEY (Produto_idProduto) 
        REFERENCES Produto(idProduto)
);

-- 6. Terceiro Vendedor (Marketplace)
CREATE TABLE Terceiro_Vendedor (
    idTerceiro_Vendedor INT AUTO_INCREMENT PRIMARY KEY,
    Razao_Social VARCHAR(45) NOT NULL,
    Local VARCHAR(45)
);

-- 7. Relação de produtos por vendedor
CREATE TABLE Relacao_de_produtos_por_vendedor (
    Terceiro_Vendedor_idTerceiro_Vendedor INT NOT NULL,
    Produto_idProduto INT NOT NULL,
    Quantidade INT DEFAULT 0,
    PRIMARY KEY (Terceiro_Vendedor_idTerceiro_Vendedor, Produto_idProduto),
    CONSTRAINT fk_rel_prod_vendedor FOREIGN KEY (Terceiro_Vendedor_idTerceiro_Vendedor) 
        REFERENCES Terceiro_Vendedor(idTerceiro_Vendedor),
    CONSTRAINT fk_rel_prod_produto FOREIGN KEY (Produto_idProduto) 
        REFERENCES Produto(idProduto)
);

-- 8. Estoque
CREATE TABLE Estoque (
    idEstoque INT AUTO_INCREMENT PRIMARY KEY,
    Local VARCHAR(45) NOT NULL
);

CREATE TABLE Produto_em_Estoque (
    Produto_idProduto INT NOT NULL,
    Estoque_idEstoque INT NOT NULL,
    Quantidade INT DEFAULT 0,
    PRIMARY KEY (Produto_idProduto, Estoque_idEstoque),
    CONSTRAINT fk_prod_estoque_produto FOREIGN KEY (Produto_idProduto) 
        REFERENCES Produto(idProduto),
    CONSTRAINT fk_prod_estoque_estoque FOREIGN KEY (Estoque_idEstoque) 
        REFERENCES Estoque(idEstoque)
);

-- 9. Pedido
CREATE TABLE Pedido (
    idPedido INT AUTO_INCREMENT PRIMARY KEY,
    Status_do_pedido VARCHAR(45) NOT NULL DEFAULT 'Em Andamento',
    Descricao VARCHAR(45),
    Cliente_idCliente INT NOT NULL,
    Frete DECIMAL(10,2) DEFAULT 0.00,
    CONSTRAINT fk_pedido_cliente FOREIGN KEY (Cliente_idCliente) 
        REFERENCES Cliente(idCliente)
);

-- 10. Forma de Pagamento do Pedido
CREATE TABLE Forma_de_Pagamento_do_Pedido (
    Pedido_idPedido INT NOT NULL,
    Forma_de_Pagamento_idForma_de_Pagamento INT NOT NULL,
    PRIMARY KEY (Pedido_idPedido, Forma_de_Pagamento_idForma_de_Pagamento),
    CONSTRAINT fk_pagto_pedido_pedido FOREIGN KEY (Pedido_idPedido) 
        REFERENCES Pedido(idPedido),
    CONSTRAINT fk_pagto_pedido_forma FOREIGN KEY (Forma_de_Pagamento_idForma_de_Pagamento) 
        REFERENCES Forma_de_Pagamento(idForma_de_Pagamento)
);

-- 11. Itens do Pedido
CREATE TABLE Relacao_de_produto_por_pedido (
    Produto_idProduto INT NOT NULL,
    Pedido_idPedido INT NOT NULL,
    Quantidade INT NOT NULL DEFAULT 1,
    Valor_Unitario DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (Produto_idProduto, Pedido_idPedido),
    CONSTRAINT fk_rel_pedido_produto FOREIGN KEY (Produto_idProduto) 
        REFERENCES Produto(idProduto),
    CONSTRAINT fk_rel_pedido_pedido FOREIGN KEY (Pedido_idPedido) 
        REFERENCES Pedido(idPedido)
);

-- 12. Entrega
CREATE TABLE Entrega (
    idEntrega INT AUTO_INCREMENT PRIMARY KEY,
    Pedido_idPedido INT NOT NULL,
    Codigo_Rastreio VARCHAR(50) UNIQUE,
    Status_Entrega VARCHAR(45) DEFAULT 'Aguardando Envio',
    Datas_de_controle VARCHAR(45),
    CONSTRAINT fk_entrega_pedido FOREIGN KEY (Pedido_idPedido) 
        REFERENCES Pedido(idPedido)
);

```
---
## ✍️ Autor

Desenvolvido por **Ricardo Freitas**  
*Estudante de Ciência de Dados & Entusiasta em Arquitetura de Dados.*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/seu-perfil)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/seu-usuario)
