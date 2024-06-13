import mysql.connector 
from mysql.connector import Error

class Database:
    def __init__(self, host, user, password, database):
        self.connection = None
        try:
            self.connection = mysql.connector.connect(
                host=host,
                user=user,
                password=password,
                database=database,
                auth_plugin="mysql_native_password"
            )
            if self.connection.is_connected():
                db_info = self.connection.get_server_info()
                print("Connected to MySQL Server version", db_info)
        except Error as e:
            print("Error while connecting to MySQL", e)
            self.connection = None

    def inserir_registro(self, tabela, dados):
        if self.connection is None:
            raise Exception("Not connected to the database.")
        placeholders = ', '.join(['%s'] * len(dados))
        columns = ', '.join(dados.keys())
        sql = f"INSERT INTO {tabela} ({columns}) VALUES ({placeholders})"
        try:
            cursor = self.connection.cursor()
            cursor.execute(sql, list(dados.values()))
            self.connection.commit()
            return cursor.lastrowid
        except Error as e:
            print("Failed to insert record into MySQL table", e)
        finally:
            cursor.close()

    def consultar_registro(self, tabela, criterios):
        if self.connection is None:
            raise Exception("Not connected to the database.")
        try:
            cursor = self.connection.cursor(dictionary=True)
            conditions = 'ND '.join([f"{k} = %s" for k in criterios.keys()])
            sql = f"SELECT * FROM {tabela} WHERE {conditions}"
            cursor.execute(sql, list(criterios.values()))
            return cursor.fetchall()
        except Error as e:
            print("Failed to read record from MySQL table", e)
        finally:
            cursor.close()

    def atualizar_registro(self, tabela, id_coluna, id_valor, novos_dados):
        if self.connection is None:
            raise Exception("Not connected to the database.")
        sets = ', '.join([f"{k} = %s" for k in novos_dados.keys()])
        valores = list(novos_dados.values())
        valores.append(id_valor)
        sql = f"UPDATE {tabela} SET {sets} WHERE {id_coluna} = %s"
        try:
            cursor = self.connection.cursor()
            cursor.execute(sql, valores)
            self.connection.commit()
        except Error as e:
            print("Failed to update record in MySQL table", e)
        finally:
            cursor.close()

    def remover_registro(self, tabela, coluna, valor):
        if self.connection is None:
            raise Exception("Not connected to the database.")
        try:
            cursor = self.connection.cursor()

            cursor.execute(f"SHOW COLUMNS FROM {tabela} LIKE %s", (coluna,))
            result = cursor.fetchone()
            if not result:
                raise Exception(f"The column '{coluna}' does not exist in table '{tabela}'.")

            sql = f"DELETE FROM {tabela} WHERE {coluna} = %s"
            cursor.execute(sql, (valor,))
            self.connection.commit()
            return cursor.rowcount > 0
        except Error as e:
            print("Failed to delete record from MySQL table", e)
        finally:
            cursor.close()

    def listar_registros(self, tabela):
        if self.connection is None:
            raise Exception("Not connected to the database.")
        try:
            cursor = self.connection.cursor(dictionary=True)
            sql = f"SELECT * FROM {tabela}"
            cursor.execute(sql)
            return cursor.fetchall()
        except Error as e:
            print("Failed to fetch records from MySQL table", e)
        finally:
            cursor.close()

# Função para exibir o menu e obter a escolha do usuário
def exibir_menu():
    print("Menu Principal:")
    print("1. Clientes")
    print("2. Animais")
    print("3. Produtos")
    print("4. Agendamentos")
    print("5. Serviços")
    print("6. Funcionários")
    print("7. Compras")
    print("0. Sair")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        exibir_menu_clientes()
    elif escolha == "2":
        exibir_menu_animais()
    elif escolha == "3":
        exibir_menu_produtos()
    elif escolha == "4":
        exibir_menu_agendamentos()
    elif escolha == "5":
        exibir_menu_servicos()
    elif escolha == "6":
        exibir_menu_funcionarios()
    elif escolha == "7":
        exibir_menu_compras()
    elif escolha == "0":
        print("Saindo...")
        exit()
    else:
        print("Opção inválida. Tente novamente.")

#cliente
class Cliente:
    def __init__(self, nome, telefone, email, endereco):
        self.nome = nome
        self.telefone = telefone
        self.email = email
        self.endereco = endereco
    def __repr__(self):
        return f'Cliente({self.nome}, {self.telefone}, {self.email}, {self.endereco})'
class Registro:
    def __init__(self, base_de_dados):
        self.base_de_dados = base_de_dados
def exibir_menu_clientes():
    print("Menu de Clientes:")
    print("1. Inserir cliente")
    print("2. Consultar cliente")
    print("3. Atualizar cliente")
    print("4. Remover cliente")
    print("5. Listar todos os clientes")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        nome = input("Nome: ")
        telefone = input("Telefone: ")
        email = input("Email: ")
        endereco = input("Endereço: ")
        novo_cliente = {'nome': nome, 'telefone': telefone, 'email': email, 'endereco': endereco}
        cliente_id = db.inserir_registro('clientes', novo_cliente)
        print(f'Novo cliente inserido com ID: {cliente_id}')
    elif escolha == "2":
        nome = input("Nome do cliente a consultar: ")
        clientes = db.consultar_registro('clientes', {'nome': nome})
        print('Clientes encontrados:', clientes)
    elif escolha == "3":
        cliente_id = input("ID do cliente a atualizar: ")
        novo_endereco = input("Novo endereço: ")
        db.atualizar_registro('clientes', 'id', int(cliente_id), {'endereco': novo_endereco})
        print('Cliente atualizado')
    elif escolha == "4":
        cliente_id = input("ID do cliente a remover: ")
        if db.remover_registro('clientes', 'id', int(cliente_id)):
            print('Cliente removido')
        else:
            print('Cliente não encontrado')
    elif escolha == "5":
        todos_clientes = db.listar_registros('clientes')
        print('Todos os clientes:', todos_clientes)
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_clientes()


#animal
def exibir_menu_animais():
    print("Menu de Animais:")
    print("1. Inserir animal")
    print("2. Consultar animal")
    print("3. Atualizar animal")
    print("4. Remover animal")
    print("5. Listar todos os animais")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")
    
    if escolha == "1":
        nome = input("Nome do animal: ")
        especie = input("Espécie: ")
        raca = input("Raça: ")
        idade = input("Idade: ")
        novo_animal = {'nome': nome, 'especie': especie, 'raca': raca, 'idade': idade}
        animal_id = db.inserir_registro('animais', novo_animal)
        print(f'Novo animal inserido com ID: {animal_id}')
    elif escolha == "2":
        nome = input("Nome do animal a consultar: ")
        animais = db.consultar_registro('animais', {'nome': nome})
        print('Animais encontrados:', animais)
    elif escolha == "3":
        animal_id = input("ID do animal a atualizar: ")
        nova_raca = input("Nova raça: ")
        db.atualizar_registro('animais', 'id', int(animal_id), {'raca': nova_raca})
        print('Animal atualizado')
    elif escolha == "4":
        animal_id = input("ID do animal a remover: ")
        if db.remover_registro('animais', 'id', int(animal_id)):
            print('Animal removido')
        else:
            print('Animal não encontrado')
    elif escolha == "5":
        todos_animais = db.listar_registros('animais')
        print('Todos os animais:', todos_animais)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_animais()


#produtos
def exibir_menu_produtos():
    print("Menu de Produtos:")
    print("1. Inserir produto")
    print("2. Consultar produto")
    print("3. Atualizar produto")
    print("4. Remover produto")
    print("5. Listar todos os produtos")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        nome = input("Nome do produto: ")
        preco = input("Preço: ")
        descricao = input("Descrição: ")
        novo_produto = {'nome': nome, 'preco': preco, 'descricao': descricao}
        produto_id = db.inserir_registro('produtos', novo_produto)
        print(f'Novo produto inserido com ID: {produto_id}')
    elif escolha == "2":
        nome = input("Nome do produto a consultar: ")
        produtos = db.consultar_registro('produtos', {'nome': nome})
        print('Produtos encontrados:', produtos)
    elif escolha == "3":
        produto_id = input("ID do produto a atualizar: ")
        novo_preco = input("Novo preço: ")
        db.atualizar_registro('produtos', 'id', int(produto_id), {'preco': novo_preco})
        print('Produto atualizado')
    elif escolha == "4":
        produto_id = input("ID do produto a remover: ")
        if db.remover_registro('produtos', 'id', int(produto_id)):
            print('Produto removido')
        else:
            print('Produto não encontrado')
    elif escolha == "5":
        todos_produtos = db.listar_registros('produtos')
        print('Todos os produtos:', todos_produtos)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_produtos()


#agendamento
def exibir_menu_agendamentos():
    print("Menu de Agendamentos:")
    print("1. Inserir agendamento")
    print("2. Consultar agendamento")
    print("3. Atualizar agendamento")
    print("4. Remover agendamento")
    print("5. Listar todos os agendamentos")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        data = input("Data do agendamento (YYYY-MM-DD): ")
        hora = input("Hora do agendamento (HH:MM): ")
        animal_id = input("ID do animal: ")
        servico = input("Serviço: ")
        novo_agendamento = {'data': data, 'hora': hora, 'animal_id': animal_id, 'ervico': servico}
        agendamento_id = db.inserir_registro('agendamentos', novo_agendamento)
        print(f'Novo agendamento inserido com ID: {agendamento_id}')
    elif escolha == "2":
        agendamento_id = input("ID do agendamento a consultar: ")
        agendamentos = db.consultar_registro('agendamentos', {'id': agendamento_id})
        print('Agendamento encontrado:', agendamentos)
    elif escolha == "3":
        agendamento_id = input("ID do agendamento a atualizar: ")
        nova_data = input("Nova data do agendamento (YYYY-MM-DD): ")
        db.atualizar_registro('agendamentos', 'id', int(agendamento_id), {'data': nova_data})
        print('Agendamento atualizado')
    elif escolha == "4":
        agendamento_id = input("ID do agendamento a remover: ")
        if db.remover_registro('agendamentos', 'id', int(agendamento_id)):
            print('Agendamento removido')
        else:
            print('Agendamento não encontrado')
    elif escolha == "5":
        todos_agendamentos = db.listar_registros('agendamentos')
        print('Todos os agendamentos:', todos_agendamentos)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_agendamentos() 


#servico
def exibir_menu_servicos():

    print("Menu de Serviços:")
    print("1. Inserir serviço")
    print("2. Consultar serviço")
    print("3. Atualizar serviço")
    print("4. Remover serviço")
    print("5. Listar todos os serviços")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        nome = input("Nome do serviço: ")
        descricao = input("Descrição do serviço: ")
        preco = input("Preço do serviço: ")
        novo_servico = {'nome': nome, 'descricao': descricao, 'preco': preco}
        servico_id = db.inserir_registro('servicos', novo_servico)
        print(f'Novo serviço inserido com ID: {servico_id}')
    elif escolha == "2":
        nome = input("Nome do serviço a consultar: ")
        servicos = db.consultar_registro('servicos', {'nome': nome})
        print('Serviços encontrados:', servicos)
    elif escolha == "3":
        servico_id = input("ID do serviço a atualizar: ")
        nova_descricao = input("Nova descrição do serviço: ")
        db.atualizar_registro('servicos', 'id', int(servico_id), {'descricao': nova_descricao})
        print('Serviço atualizado')
    elif escolha == "4":
        servico_id = input("ID do serviço a remover: ")
        if db.remover_registro('servicos', 'id', int(servico_id)):
            print('Serviço removido')
        else:
            print('Serviço não encontrado')
    elif escolha == "5":
        todos_servicos = db.listar_registros('servicos')
        print('Todos os serviços:', todos_servicos)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_servicos()                       


#Funcionario
def exibir_menu_funcionarios():
    print("Menu de Funcionários:")
    print("1. Inserir funcionário")
    print("2. Consultar funcionário")
    print("3. Atualizar funcionário")
    print("4. Remover funcionário")
    print("5. Listar todos os funcionários")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        nome = input("Nome do funcionário: ")
        cpf = input("CPF do funcionário: ")
        telefone = input("Telefone do funcionário: ")
        endereco = input("Endereço do funcionário: ")
        novo_funcionario = {'nome': nome, 'cpf': cpf, 'telefone': telefone, 'endereco': endereco}
        funcionario_id = db.inserir_registro('funcionarios', novo_funcionario)
        print(f'Novo funcionário inserido com ID: {funcionario_id}')
    elif escolha == "2":
        cpf = input("CPF do funcionário a consultar: ")
        funcionarios = db.consultar_registro('funcionarios', {'cpf': cpf})
        print('Funcionário encontrado:', funcionarios)
    elif escolha == "3":
        funcionario_id = input("ID do funcionário a atualizar: ")
        novo_telefone = input("Novo telefone do funcionário: ")
        db.atualizar_registro('funcionarios', 'id', int(funcionario_id), {'telefone': novo_telefone})
        print('Funcionário atualizado')
    elif escolha == "4":
        funcionario_id = input("ID do funcionário a remover: ")
        if db.remover_registro('funcionarios', 'id', int(funcionario_id)):
            print('Funcionário removido')
        else:
            print('Funcionário não encontrado')
    elif escolha == "5":
        todos_funcionarios = db.listar_registros('funcionarios')
        print('Todos os funcionários:', todos_funcionarios)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_funcionarios()


#compras 
def exibir_menu_compras():
    print("Menu de Compras:")
    print("1. Inserir compra")
    print("2. Consultar compra")
    print("3. Atualizar compra")
    print("4. Remover compra")
    print("5. Listar todas as compras")
    print("0. Voltar ao menu principal")

    escolha = input("Escolha uma opção: ")

    if escolha == "1":
        cliente_id = input("ID do cliente: ")
        produto_id = input("ID do produto: ")
        quantidade = input("Quantidade comprada: ")
        data_compra = input("Data da compra (DD/MM/YYYY): ")
        nova_compra = {'cliente_id': int(cliente_id), 'produto_id': int(produto_id), 'quantidade': int(quantidade), 'data_compra': data_compra}
        compra_id = db.inserir_registro('compras', nova_compra)
        print(f'Nova compra inserida com ID: {compra_id}')
    elif escolha == "2":
        compra_id = input("ID da compra a consultar: ")
        compras = db.consultar_registro('compras', {'id': int(compra_id)})
        print('Compra encontrada:', compras)
    elif escolha == "3":
        compra_id = input("ID da compra a atualizar: ")
        nova_quantidade = input("Nova quantidade comprada: ")
        db.atualizar_registro('compras', 'id', int(compra_id), {'quantidade': int(nova_quantidade)})
        print('Compra atualizada')
    elif escolha == "4":
        compra_id = input("ID da compra a remover: ")
        if db.remover_registro('compras', 'id', int(compra_id)):
            print('Compra removida')
        else:
            print('Compra não encontrada')
    elif escolha == "5":
        todas_compras = db.listar_registros('compras')
        print('Todas as compras:', todas_compras)    
    elif escolha == "0":
        exibir_menu()
    else:
        print("Opção inválida. Tente novamente.")
        exibir_menu_compras()
