import json
import os
from datetime import datetime, timedelta

# ==============================
# VARIÁVEIS GLOBAIS
# ==============================
tarefas = []
proximo_id = 1
ARQUIVO_TAREFAS = "tarefas.json"
ARQUIVO_ARQUIVADAS = "tarefas_arquivadas.json"

# =====================================================================================
# FUNÇÕES DE ARQUIVOS
# =====================================================================================

def carregar_arquivos():
    """
    Carrega ou cria os arquivos JSON necessários.
    Preenche a lista global de tarefas e ajusta o contador de IDs.
    """
    print("Executando a função carregar_arquivos")

    global tarefas, proximo_id

    # Criar arquivos vazios caso não existam
    if not os.path.exists(ARQUIVO_TAREFAS):
        with open(ARQUIVO_TAREFAS, "w") as f:
            json.dump([], f)

    if not os.path.exists(ARQUIVO_ARQUIVADAS):
        with open(ARQUIVO_ARQUIVADAS, "w") as f:
            json.dump([], f)

    # Carregar lista de tarefas
    with open(ARQUIVO_TAREFAS, "r") as f:
        tarefas = json.load(f)

    # Ajustar ID automático
    if tarefas:
        proximo_id = max(t["id"] for t in tarefas) + 1


def salvar_arquivos():
    """
    Salva a lista atual de tarefas no arquivo JSON antes de sair do sistema.
    """
    print("Executando a função salvar_arquivos")

    with open(ARQUIVO_TAREFAS, "w") as f:
        json.dump(tarefas, f, indent=4)


# =====================================================================================
# FUNÇÕES PRINCIPAIS
# =====================================================================================

def criar_tarefa():
    """
    Cria uma nova tarefa solicitando dados ao usuário e adiciona à lista global.
    """
    print("Executando a função criar_tarefa")
    global proximo_id, tarefas

    titulo = input("Título da tarefa: ").strip()
    if not titulo:
        print("Título obrigatório.")
        return

    descricao = input("Descrição: ")

    prioridades = ["Urgente", "Alta", "Média", "Baixa"]
    print("Prioridades: ", prioridades)
    prioridade = input("Informe a prioridade: ").title()

    if prioridade not in prioridades:
        print("Prioridade inválida!")
        return

    origens = ["Email", "Telefone", "Chamado"]
    print("Origem da tarefa:", origens)
    origem = input("Informe a origem: ").title()

    if origem not in [o.title() for o in origens]:
        print("Origem inválida!")
        return

    tarefa = {
        "id": proximo_id,
        "titulo": titulo,
        "descricao": descricao,
        "prioridade": prioridade,
        "status": "Pendente",
        "origem": origem,
        "data_criacao": datetime.now().isoformat(),
        "data_conclusao": None
    }

    tarefas.append(tarefa)
    proximo_id += 1
    print("Tarefa criada com sucesso!")


def pegar_tarefa():
    """
    Seleciona a tarefa de maior prioridade e muda o status para Fazendo.
    """
    print("Executando a função pegar_tarefa")

    prioridades_ordem = ["Urgente", "Alta", "Média", "Baixa"]

    for p in prioridades_ordem:
        for t in tarefas:
            if t["status"] == "Pendente" and t["prioridade"] == p:
                t["status"] = "Fazendo"
                print("Tarefa selecionada:")
                print(t)
                return

    print("Nenhuma tarefa pendente encontrada.")


def atualizar_prioridade():
    """
    Atualiza a prioridade de uma tarefa já registrada.
    """
    print("Executando a função atualizar_prioridade")

    try:
        id_tarefa = int(input("ID da tarefa: "))
    except:
        print("ID inválido.")
        return

    prioridades = ["Urgente", "Alta", "Média", "Baixa"]

    for t in tarefas:
        if t["id"] == id_tarefa:
            print("Novas opções:", prioridades)
            nova = input("Nova prioridade: ").title()

            if nova in prioridades:
                t["prioridade"] = nova
                print("Prioridade atualizada!")
            else:
                print("Prioridade inválida.")
            return

    print("Tarefa não encontrada.")


def concluir_tarefa():
    """
    Marca uma tarefa como concluída e registra data e hora da conclusão.
    """
    print("Executando a função concluir_tarefa")

    try:
        id_tarefa = int(input("ID da tarefa: "))
    except:
        print("ID inválido.")
        return

    for t in tarefas:
        if t["id"] == id_tarefa:
            t["status"] = "Concluída"
            t["data_conclusao"] = datetime.now().isoformat()
            print("Tarefa concluída!")
            return

    print("Tarefa não encontrada.")


def arquivar_automatico():
    """
    Arquiva automaticamente tarefas concluídas há mais de 7 dias.
    """
    print("Executando a função arquivar_automatico")

    agora = datetime.now()
    arquivadas = []

    for t in tarefas:
        if t["status"] == "Concluída" and t["data_conclusao"]:
            data = datetime.fromisoformat(t["data_conclusao"])
            if (agora - data) > timedelta(days=7):
                t["status"] = "Arquivado"
                arquivadas.append(t)

    if arquivadas:
        with open(ARQUIVO_ARQUIVADAS, "r") as f:
            historico = json.load(f)
        historico.extend(arquivadas)
        with open(ARQUIVO_ARQUIVADAS, "w") as f:
            json.dump(historico, f, indent=4)
        print("Tarefas arquivadas automaticamente.")


def excluir_tarefa():
    """
    Realiza exclusão lógica de uma tarefa, marcando status como Excluída.
    """
    print("Executando a função excluir_tarefa")

    try:
        id_tarefa = int(input("ID da tarefa: "))
    except:
        print("ID inválido.")
        return

    for t in tarefas:
        if t["id"] == id_tarefa:
            t["status"] = "Excluída"
            print("Tarefa excluída logicamente.")
            return

    print("Tarefa não encontrada.")


def relatorio():
    """
    Exibe todas as tarefas com todas as informações e calcula tempo de execução.
    """
    print("Executando a função relatorio")

    for t in tarefas:
        print("-" * 40)
        print("ID:", t["id"])
        print("Título:", t["titulo"])
        print("Status:", t["status"])
        print("Prioridade:", t["prioridade"])

        if t["status"] == "Concluída" and t["data_conclusao"]:
            inicio = datetime.fromisoformat(t["data_criacao"])
            fim = datetime.fromisoformat(t["data_conclusao"])
            duração = fim - inicio
            print("Tempo execução:", duração)


def relatorio_arquivadas():
    """
    Exibe apenas tarefas arquivadas (não mostra excluídas).
    """
    print("Executando a função relatorio_arquivadas")

    with open(ARQUIVO_ARQUIVADAS, "r") as f:
        historico = json.load(f)

    print("\n--- TAREFAS ARQUIVADAS ---")
    for t in historico:
        print(t)


# =====================================================================================
# MENU PRINCIPAL
# =====================================================================================

def menu():
    carregar_arquivos()
    arquivar_automatico()

    while True:
        print("\n=== MENU ===")
        print("1 - Criar Tarefa")
        print("2 - Pegar Tarefa")
        print("3 - Atualizar Prioridade")
        print("4 - Concluir Tarefa")
        print("5 - Excluir Tarefa")
        print("6 - Relatório Completo")
        print("7 - Relatório Arquivadas")
        print("8 - Sair")

        try:
            opcao = int(input("Escolha: "))
        except:
            print("Opção inválida.")
            continue

        if opcao == 1:
            criar_tarefa()
        elif opcao == 2:
            pegar_tarefa()
        elif opcao == 3:
            atualizar_prioridade()
        elif opcao == 4:
            concluir_tarefa()
        elif opcao == 5:
            excluir_tarefa()
        elif opcao == 6:
            relatorio()
        elif opcao == 7:
            relatorio_arquivadas()
        elif opcao == 8:
            salvar_arquivos()
            print("Saindo...")
            exit()
        else:
            print("Opção inexistente!")


menu()
