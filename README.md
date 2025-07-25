# trilha_python_desafio3
Transformação do script de sistema bancario utilizando classes, treino do bootcamp


from abc import ABC, abstractclassmethod, abstractproperty
from datetime import datetime

class Cliente:
    def __init__(self, endereco):
        self.endereco = endereco
        self.contas = []

    def realizar_transacao(self, conta, transacao):
        transacao.registrar(conta)

    def adicionar_conta(self, conta):
        self.contas.append(conta)

class PessoaFisica(Cliente):
    def __init__(self, nome, data_nascimento, cpf, endereco):
        super().__init__(endereco)
        self.nome = nome
        self.data_nascimento = data_nascimento
        self.cpf = cpf

class Conta:
    def __init__(self, numero, cliente):
        self.saldo = 0
        self.numero = numero
        self.agencia = "0001"
        self.cliente = cliente
        self.historico = Historico()

    @classmethod
    def nova_conta(cls, cliente, numero):
        return cls(numero, cliente)

    @property
    def saldo(self):
        return self.saldo

    @property
    def numero(self):
        return self.numero

    @property
    def agencia(self):
        return self.agencia

    @property
    def cliente(self):
        return self.cliente

    @property
    def historico(self):
        return self.historico

    def sacar(self, valor):
        saldo = self.saldo
        excedeu_saldo = valor > saldo

        if excedeu_saldo:
            print('\n@@@ Operação falhou! Você não tem saldo suficiente. @@@')

        elif valor > 0:
            self._saldo -= valor
            print('\n=== Saque realizado com sucesso. ===')
            return True

        else:
            print('\n@@@ Operação falhou! O valor informado é inváldo. @@@')

        return False

    def depositar(self, valor):
        if valor > 0:
            self._saldo += valor
            print('\n=== Depósito realizado com sucesso. ===')
        else:
            print('\n@@@ Operação falhou! O valor informado é inválido. @@@')
            return False

        return True

class ContaCorrrente(Conta):
    def __init__(self, numero, cliente, limite=500, limite_saques=3):
        super().__init__(numero, cliente)
        self.limite = limite
        self.limite_saques = limite_saques

    def sacar(self, valor):
        numero_saques = len(
            [transacao for transacao in self.historico.transacoes
             if transacao["tipo"] == Saque.__name__])

        excedeu_limite = valor > self.limite
        excedeu_saques = numeros_saques > self.limite_saques

        if excedeu_limite:
            print('\n@@@ Operação falhou! O valor do saque excede o limite. @@@')
        elif excedeu_saques:
            print('\n@@@ Operação falhou! Número máximo de saques excedido. @@@')
        else:
            return super().sacar(valor)

        return False

    def __str__(self):
        return f"""\
            Agência:\t{self.agencia}
            CC:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
    """

class Historico:
    def __init__(self):
        self.transacoes = []

    @property
    def transacoes(self):
        return self._transacoes

    def adicionar_transacao(self, transacao):
        self.transacoes.append({
            "tipo": transacao.__class__.__name__,
            "valor": transacao.valor,
            "data": datetime.now().strftime("%d-%m-%Y %H:%M:%s"),
        })

class Transacao(ABC):
    @property
    @abstractproperty
    def valor(self):
        pass

    @abstractclassmethod
    def registrar(self, conta):
        pass

class Saque(Transacao):
    def __init__(self, valor):
        self.valor = valor

    @property
    def valor(self):
        return self.valor

    def registrar(self, conta):
        sucesso_transacao = conta.sacar(self.valor)

        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)

class Deposito(Transacao):
    def __init__(self, valor):
        self.valor = valor

    @property
    def valor(self):
        return self.valor

    def registrar(self, conta):
        sucesso_transacao = conta.depositar(self.valor)

        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)

def menu():
    ######################################################
    ##                     MENU                         ##
    ######################################################

    menu = """
    ################ SISTEMA BANCÁRIO 2.0 ################
        1. [c] Cadastro de Cliente
        2. [n] Cadastro Nova Conta
        3. [d] Depósito
        4. [e] Extrato
        5. [s] Saque
        6. [f] Finaliza
    ######################################################    
    """
    return input(textwrap.dedent(menu))

def filtrar_cliente(cpf, clientes):
    clientes_filtrados = [cliente for cliente in clientes if cliente.cpf == cpf]
    return clientes_filtrados[0] if clientes_filtrados else None

def recuperar_conta_cliente(cliente):
    if not cliente.contas:
        print('\n@@@ Cliente não possui conta! @@@')
        return

    #FIXME: não permite cliente escolher a conta
    return cliente.contas[0]

def depositar(clientes):
    cpf = input('Informe o CPF do cliente: ')
    cliente = filtrar_cliente(cpf, clientes)

    if not cliente:
        print('\n@@@ Cliente não encontrado!! @@@')
        return

    valor = float(input('Informe o valor do depósito: '))
    transacao = Deposito(valor)

    conta = recuperar_conta_cliente(cliente)
    if not conta:
        return

    cliente.realizar_transacao(conta, transacao)

def sacar(clientes):
    cpf = input('Informe o CPF do cliente: ')
    cliente = filtrar_cliente(cpf, clientes)

    if not cliente:
        print('\n@@@ Cliente não encontrado! @@@')
        return

    valor = float(input('Informe o valor do saque: '))
    transacao = Saque(valor)

    conta = recuperar_conta_cliente(cliente)
    if not conta:
        return

    cliente.realizar_transacao(conta, transacao)

def exibir_extrato(clientes):
    cpf = input('Informe o CPF do cliente: ')
    cliente = filtrar_cliente(cpf, clientes)

    if not cliente:
        print('\n@@@ Cliente não encontrado! @@@')
        return

    conta = recuperar_conta_cliente(cliente)
    if not conta:
        return

    print('\n============= EXTRATO =============')
    transacoes = conta.historico.transacoes

    extrato = ''
    if not transacoes:
        extrato = 'Não foram realizadas movimentações'
    else:
        for transacao in transacoes:
            extrato += f"\n{transacao['tipo']}:\n\tR$ {transacao['valor']:.2f}"

    print(extrato)
    print(f'\nSaldo:\n\tR$ {conta.saldo:.2f}')
    print('======================================')

def criar_conta(numero_conta, clientes, contas):
    cpf = input('Informe o CPF do cliente: ')
    cliente = filtrar_cliente(cpf, clientes)

    if not cliente:
        print('\n@@@ Cliente não encontrado, fluxo de criação de conta encerrado! @@@')
        return

    conta = ContaCorrente.nova_conta(cliente=cliente, numero=numero_conta)
    contas.append(conta)
    clientes.contas.append(conta)

    print('\n=== Conta criada com sucesso! ===')

def listar_contas(contas):
    for conta in contas:
        print('=' * 100)
        print(textwrap.dedent(str(conta)))

def main():
    clientes = []
    contas = []

    while True:
        opcao = menu()

        if opcao.upper() == "D":
            depositar(clientes)

        elif opcao.upper() == "S":
            sacar(clientes)

        elif opcao.upper() == "E":
            exibir_extrato(clientes)

        elif opcao.upper() == "N":
            criar_clientes(clientes)

        elif opcao.upper() == "C":
            numero_conta = len(contas) + 1
            criar_conta(numero_conta, clientes, contas)

        elif opcao.upper() == "L":
            listar_contas(contas)

        ## BLOCO FINALIZA SESSÃO
        elif opcao.upper() == 'F':
            print('Fim da Execução, volte sempre')
            break

        ## BLOCO OPÇÃO INVÁLIDA
        elif opcao.upper not in ('D', 'S', 'E', 'F', 'C', 'N', 'L'):
            print('Opção Invalida')
