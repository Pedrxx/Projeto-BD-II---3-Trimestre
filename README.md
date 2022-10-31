<h1 align="center">
mcXAMPPinho
</h1>

[Gustavo](#) | [Renam](#) | [Pablo](https://github.com/PabloDomiciano) | [Pedro](https://github.com/Pedrxx) | [Márcio](https://github.com/MarcioJCarvalho)


### TRIGGERS
01- Escreva quatro triggers de sintaxe - a trigger não precisa ter funcionalidade, basta não dar erro de sintaxe. Use variável global para testar.
- Faça uma declarando variáveis e com select into; 
- Faça a segunda com uma estrutura de decisão; 
- Faça a terceira que gere erro, impedindo a ação;
- Faça a quarta que utilize a variável new e old - tente diferenciar. 
```mysql
DELIMITER // 
CREATE TRIGGER sintaxe_trigger 
	AFTER 
	INSERT
	ON tabela
	FOR EACH ROW 
	BEGIN
		SET @meu_var = CURRENT_TIMESTAMP;
	END;
// DELIMITER ;
```

02- Uma trigger que tem a função adicionar a entrada de produtos no estoque deve ser associado para qual:
- Tabela?
- Tempo?
- Evento?
- Precisa de variáveis? Quais?
- Implemente a trigger. 

A) Item_venda.
B) Pode ser associado a qualquer tempo, before ou after.
C) UPDATE
D) Sim, estoque.
C) ...

```mysql
DELIMITER // 
CREATE TRIGGER atualiza_estoque
AFTER INSERT ON item_venda FOR EACH ROW
BEGIN 
	UPDATE produto SET estoque = estoque - new.quantidade WHERE produto.id = new.produto_id;
END;
//
DELIMITER ;
```

```mysql
DELIMITER // 
CREATE TRIGGER retorna_erro
BEFORE INSERT ON produto 
FOR EACH ROW
BEGIN 
	IF produto.quantidade < 0 THEN
		signal sqlstate '45000' set message_text = 'quantidade do produto não pode ser negativa';
	END IF;	
END;
//
DELIMITER ;
```

03- Uma trigger que tem a função criar um registro de auditoria quando um pagamento e recebimento for alterada deve ser associado para qual(is):
- Tabela(s)?
- Tempo?
- Evento?
- Implemente a trigger (pode criar a tabela de auditoria)
```mysql
DELIMITER // 
CREATE TRIGGER select_into 
BEFORE INSERT ON venda FOR EACH ROW
-- PRECEDES venda_horario_comercial
BEGIN 
	DECLARE cliente_ativo CHAR(1); 
    SELECT ativo INTO cliente_ativo FROM cliente WHERE id = new.cliente_id;
    -- funconanlidade 
END;
//
DELIMITER ;
```

04- Uma trigger que tem a função impedir a venda de um produto inferior a 50% do preço de venda deve ser associado para qual:
- Tabela?
- Tempo?
- Evento?
- Implemente a trigger
```mysql
DELIMITER // 
CREATE TRIGGER impedir_desconto_maior_que_metade_do_preco
BEFORE INSERT ON produto FOR EACH ROW
-- FOLLOWS impedir_desconto_maior_que_metade_do_preco
BEGIN 
    IF NOT TIME(current_timestamp) between TIME('08:00:00') and TIME('18:00:00') THEN
		signal sqlstate '45000' set message_text = 'desconto não permitido';
	END IF;
END;
//
DELIMITER ;
```

### PROCEDURES
01- Escreva quarto procedures de sintaxe - não precisa ter funcionalidade, basta não dar erro de sintaxe. Use variável global para testar.
- Faça uma declarando variáveis e com select into; 
- Faça a segunda com uma estrutura de decisão; 
- Faça a terceira que gere erro, impedindo a ação;
- Faça a quarta com if e else. 
```mysql
DELIMITER //
CREATE PROCEDURE atualiza_estoque(id_produto INT, qtde_comprada INT, valor_unit DECIMAL(9,2))
BEGIN
    SELECT count(*) INTO @contador FROM estoque WHERE id_produto = id_prod;
    IF contador > 0 THEN
        UPDATE estoque SET qtde=qtde + qtde_comprada, valor_unitario= valor_unit
        WHERE id_produto = id_prod;
    ELSE
        INSERT INTO estoque (id_produto, qtde, valor_unitario) values (id_prod, qtde_comprada, valor_unit);
    END IF;
END //
DELIMITER ;
```

02 - Escreva uma procedure que registre a baixa de um produto e já atualize devidamente o estoque do produto. Antes das ações, verifique se o produto é ativo.
```mysql
DELIMITER //
CREATE PROCEDURE inserir_venda_ativo(ID_PRODUTO INT, QUANTIDADE INT)
BEGIN
	DECLARE ATIVO CHAR;
    DECLARE QUANT INT;
    SELECT P.ATIVO INTO ATIVO FROM PRODUTO P;
    IF ATIVO = 'I' THEN
		BEGIN 
			SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'PRODUTO INATIVO';
        END;
	ELSE 
		BEGIN 
			INSERT  INTO VENDA VALUES (ID_PRODUTO, QUANTIDADE);
            UPDATE PRODUTO SET SALDO = QUANTIDADE WHERE ID_PRODUTO = ID_PRODUTO;
        END;
    END IF;
END;
// 
DELIMITER ;
```

03 - Escreva uma procedure que altere o preço de um produto vendido (venda já realizada - necessário verificar a existência da venda). Não permita altearções abusivas - preço de venda abaixo do preço de custo. É possível implementar esta funcionalidade sem a procedure? Se sim, indique como, bem como as vantagens e desvantagens.
```mysql
```

04 - Escreva uma procedure que registre vendas de produtos e já defina o total da venda. É possível implementar a mesma funcionalidade por meio da trigger? Qual seria a diferença?
```mysql
DELIMITER //
	CREATE PROCEDURE registrar_venda(id_produto INT, id_venda INT, quatidade INT, preco_unidade DECIMAL(8,2))
	BEGIN
		DECLARE soma_total DECIMAL (8,2);
        DECLARE preco DECIMAL (8,2);
        DECLARE quantidade FLOAT;
        SELECT iv.preco_unidade INTO preco FROM item_venda iv;
        SELECT iv.quantidade INTO quantidade FROM item_venda iv;
        SET soma_total = preco * quantidade;
    END;
//
DELIMITER ;
```

5- Para o controle de salário de funcionários de uma empresa e os respectivos adiantamentos (vales):
 - quais tabelas são necessárias?
 
R: tabela pagamento e funcionario
```mysql
DELIMITER //
CREATE PROCEDURE table_pagamento
BEGIN
    drop table pagamento;
    CREATE TABLE pagamento(
        id INT NOT NULL AUTO_INCREMENT
        ,salario DECIMAL (8,3 )
        ,vale DECIMAL(8,3)
        ,func_id int NOT NULL
        ,CONSTRAINT id_pagamento PRIMARY KEY(id)
        ,CONSTRAINT func_fk FOREIGN KEY (func_id) REFERENCES funcionario (ID) 
    );
END;
//
DELIMITER ;
```

06- De acordo com o seu projeto de banco de dados, pense em pelo menos 3 procedures úteis. Discuta com os seus colegas em relação a relevância e implemente-as.
```mysql
```

07- Explique as diferenças entre trigger, função e procedure. Indique as vantagens e desvantagens em utilizar a procedure.
```text
Uma função precisa ser chamada manualmente e deve obrigatoriamente ter retorno, diferente de uma procedure que como o próprio nome diz, trata-se de um procedimento, logo não é necessário retorno. Já a trigger, "gatilho" em português, tem esse nome porque é "disparada" sempre que determinado evento ocorre no banco de dados, não precisando ser chamada diretamente como as outras citadas.
A grande vantagem de se utilizar Stored Procedures é a produtividade, que leva diretamente à facilidade de manutenção, facilidade de uso e escalabilidade, pois imagine que ao invés de criar códigos no backend da aplicação e\ou realizar vários comandos para manipular os dados de um banco você pudesse construir uma rotina dinâmica que pode ou não receber parâmetros para ser executada sempre que quiser, podendo ser reutilizada infinitamente: isso são Procedures. Porém, isso traz também a pequena desvantagem que é a dificuldade na escrita (pois a sintaxe não é tão conhecida) 
```







