# PROJETO-BANCO-POO
from abc import ABC, abstractmethod
from datetime import datetime


# =====================
# HISTÓRICO
# =====================

class Historico:
    def __init__(self):
        self.transacoes = []

    def adicionar_transacao(self, transacao):
        self.transacoes.append(
            {
                "tipo": transacao.__class__.__name__,
                "valor": transacao.valor,
                "data": datetime.now().strftime("%d/%m/%Y %H:%M:%S")
            }
        )


# =====================
# CONTA
# =====================

class Conta:
    agencia = "0001"

    def __init__(self, cliente, numero):
        self._saldo = 0
        self._numero = numero
        self._agencia = Conta.agencia
        self._cliente = cliente
        self._historico = Historico()

    @property
    def saldo(self):
        return self._saldo

    @property
    def historico(self):
        return self._historico

    def sacar(self, valor):
        if valor > 0 and valor <= self._saldo:
            self._saldo -= valor
            return True

        print("Saque não realizado.")
        return False

    def depositar(self, valor):
        if valor > 0:
            self._saldo += valor
            return True

        print("Valor inválido.")
        return False


# =====================
# CONTA CORRENTE
# =====================

class ContaCorrente(Conta):
    def __init__(self, cliente, numero, limite=500, limite_saques=3):
        super().__init__(cliente, numero)
        self._limite = limite
        self._limite_saques = limite_saques

    def sacar(self, valor):
        quantidade_saques = sum(
            1
            for t in self.historico.transacoes
            if t["tipo"] == "Saque"
        )

        if valor > self._limite:
            print("Valor excede o limite.")
            return False

        if quantidade_saques >= self._limite_saques:
            print("Limite de saques atingido.")
            return False

        return super().sacar(valor)


# =====================
# CLIENTE
# =====================

class Cliente:
    def __init__(self, endereco):
        self.endereco = endereco
        self.contas = []

    def adicionar_conta(self, conta):
        self.contas.append(conta)

    def realizar_transacao(self, conta, transacao):
        transacao.registrar(conta)


# =====================
# PESSOA FÍSICA
# =====================

class PessoaFisica(Cliente):
    def __init__(self, nome, cpf, data_nascimento, endereco):
        super().__init__(endereco)

        self.nome = nome
        self.cpf = cpf
        self.data_nascimento = data_nascimento


# =====================
# INTERFACE TRANSAÇÃO
# =====================

class Transacao(ABC):

    @property
    @abstractmethod
    def valor(self):
        pass

    @abstractmethod
    def registrar(self, conta):
        pass


# =====================
# DEPÓSITO
# =====================

class Deposito(Transacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        if conta.depositar(self.valor):
            conta.historico.adicionar_transacao(self)


# =====================
# SAQUE
# =====================

class Saque(Transacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        if conta.sacar(self.valor):
            conta.historico.adicionar_transacao(self)


# =====================
# TESTE
# =====================

cliente = PessoaFisica(
    nome="Wellington",
    cpf="12345678900",
    data_nascimento="01/01/2000",
    endereco="Apodi-RN"
)

conta = ContaCorrente(cliente, 1)

cliente.adicionar_conta(conta)

cliente.realizar_transacao(
    conta,
    Deposito(1000)
)

cliente.realizar_transacao(
    conta,
    Saque(200)
)

print(f"Saldo: R$ {conta.saldo:.2f}")

print("\nHistórico:")
for t in conta.historico.transacoes:
    print(t)
