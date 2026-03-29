Campo,Tipo,Descrição
veiculo_id,FK,Chave estrangeira para a tabela de veículos.
cliente_id,FK,Chave estrangeira para a tabela de usuários/clientes.
data_inicio,DateTime,Início do uso.
data_fim,DateTime,Previsão de entrega.
valor_total,Decimal,Calculado pelo sistema no momento da reserva.
status,Enum,"Pendente, Ativa, Concluída, Cancelada."# Exemplo de validação da regra de negócio
def criar_reserva(veiculo, cliente, data_ini, data_fim):
    if veiculo.status != "Disponível":
        return "Erro: Veículo já está em uso ou em manutenção."
    
    if data_ini < hoje:
        return "Erro: A data de início não pode ser no passado."
    
    # Cálculo da regra de negócio
    dias = (data_fim - data_ini).days
    valor_estimado = dias * veiculo.valor_diaria
    
    # Salvar no banco (Branch: Código Inicial)
    reserva = Reserva.save(veiculo, cliente, data_ini, data_fim, valor_estimado)
    return reserva
    
