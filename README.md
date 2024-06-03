# Projeto de Banco de Dados com PostgreSQL

Este projeto contém consultas SQL para obter informações de um banco de dados PostgreSQL, incluindo consultas básicas e de junção.

## Estrutura do Banco de Dados

O banco de dados possui as seguintes tabelas:
- EMPRESA (id_empresa, razao_social, inativo)
- PRODUTOS (id_produto, descricao, inativo)
- CONFIG_PRECO_PRODUTO (id_config_preco_produto, id_vendedor, id_empresa, id_produto, preco_minimo, preco_maximo)
- VENDEDORES (id_vendedor, nome, cargo, salario, data_admissao, inativo)
- CLIENTES (id_cliente, razao_social, data_cadastro, id_vendedor, id_empresa, inativo)
- PEDIDO (id_pedido, id_empresa, id_cliente, valor_total, data_emissao, situacao)
- ITENS_PEDIDO (id_item_pedido, id_pedido, id_produto, preco_praticado, quantidade)

## Consultas SQL

### Consultas Básicas

1. **Lista de funcionários ordenando pelo salário decrescente:**
    ```sql
    SELECT nome, cargo, salario, data_admissao
    FROM VENDEDORES
    ORDER BY salario DESC;
    ```

2. **Lista de pedidos de vendas ordenado por data de emissão:**
    ```sql
    SELECT id_pedido, id_empresa, id_cliente, valor_total, data_emissao, situacao
    FROM PEDIDO
    ORDER BY data_emissao;
    ```

3. **Valor de faturamento por cliente:**
    ```sql
    SELECT c.id_cliente, c.razao_social, SUM(p.valor_total) AS faturamento_total
    FROM CLIENTES c
    JOIN PEDIDO p ON c.id_cliente = p.id_cliente
    GROUP BY c.id_cliente, c.razao_social
    ORDER BY faturamento_total DESC;
    ```

4. **Valor de faturamento por empresa:**
    ```sql
    SELECT e.id_empresa, e.razao_social, SUM(p.valor_total) AS faturamento_total
    FROM EMPRESA e
    JOIN PEDIDO p ON e.id_empresa = p.id_empresa
    GROUP BY e.id_empresa, e.razao_social
    ORDER BY faturamento_total DESC;
    ```

5. **Valor de faturamento por vendedor:**
    ```sql
    SELECT v.id_vendedor, v.nome, SUM(p.valor_total) AS faturamento_total
    FROM VENDEDORES v
    JOIN CLIENTES c ON v.id_vendedor = c.id_vendedor
    JOIN PEDIDO p ON c.id_cliente = p.id_cliente
    GROUP BY v.id_vendedor, v.nome
    ORDER BY faturamento_total DESC;
    ```

### Consultas de Junção

**Consulta de junção para obter o preço base do produto:**
```sql
SELECT 
    p.id_produto,
    p.descricao,
    c.id_cliente,
    c.razao_social AS cliente_razao_social,
    e.id_empresa,
    e.razao_social AS empresa_razao_social,
    v.id_vendedor,
    v.nome AS vendedor_nome,
    cpp.preco_minimo,
    cpp.preco_maximo,
    COALESCE(
        (SELECT ip.preco_praticado
         FROM ITENS_PEDIDO ip
         JOIN PEDIDO pe ON ip.id_pedido = pe.id_pedido
         WHERE ip.id_produto = p.id_produto AND pe.id_cliente = c.id_cliente
         ORDER BY pe.data_emissao DESC
         LIMIT 1),
        cpp.preco_minimo
    ) AS preco_base
FROM PRODUTOS p
JOIN CONFIG_PRECO_PRODUTO cpp ON p.id_produto = cpp.id_produto
JOIN CLIENTES c ON cpp.id_empresa = c.id_empresa
JOIN EMPRESA e ON c.id_empresa = e.id_empresa
JOIN VENDEDORES v ON c.id_vendedor = v.id_vendedor
ORDER BY p.id_produto, c.id_cliente;
