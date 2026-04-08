const { Reserva, Veiculo } = require('../models');
const { Op } = require('sequelize');

class ReservaService {
    async criarReserva(veiculoId, clienteId, dataIni, dataFim) {
        // 1. Verificação de existência e status do veículo
        const veiculo = await Veiculo.findByPk(veiculoId);
        if (!veiculo || veiculo.status !== 'Disponível') {
            throw new Error("Veículo indisponível ou não encontrado.");
        }

        // 2. Validação de Regra de Negócio: Anti-Overbooking
        // Verifica se já existe reserva ativa para este veículo no período selecionado
        const conflito = await Reserva.findOne({
            where: {
                veiculo_id: veiculoId,
                status: { [Op.ne]: 'Cancelada' },
                [Op.or]: [
                    { data_inicio: { [Op.between]: [dataIni, dataFim] } },
                    { data_fim: { [Op.between]: [dataIni, dataFim] } }
                ]
            }
        });

        if (conflito) {
            throw new Error("Este veículo já possui uma reserva para o período selecionado.");
        }

        // 3. Cálculo de Valor (Lógica de Negócio)
        const dias = Math.ceil(Math.abs(new Date(dataFim) - new Date(dataIni)) / (1000 * 60 * 60 * 24));
        const valorTotal = dias * veiculo.valorDiaria;

        // 4. Persistência ACID
        return await Reserva.create({
            veiculo_id: veiculoId,
            cliente_id: clienteId,
            data_inicio: dataIni,
            data_fims: dataFim,
            valor_total: valorTotal,
            status: 'Pendente'
        });
    }
}

module.exports = new ReservaService();

Funcionalidade,Papel da Implementação,Tecnologia Chave
Login,Autenticação JWT com criptografia bcrypt,Node.js + JWT
Gestão de Reservas,Lógica de não-sobreposição (Anti-overbooking),PostgreSQL (Queries de data)
Vistoria Digital,Upload de fotos e checklist de avarias,AWS S3 + React
Cálculo de Tarifas,Processamento automático de valores,Node.js (Regras de Negócio)
    
/
├── src/
│   ├── config/         # Configuração de banco de dados (Sequelize/Postgres)
│   ├── controllers/    # Orquestração das requisições (ReservaController.js)
│   ├── models/         # Definição das entidades (Reserva.js, Veiculo.js)
│   ├── services/       # Regras de Negócio e Validações (ReservaService.js)
│   ├── routes/         # Definição dos Endpoints
│   └── app.js          # Entrada da aplicação Express
├── tests/              # Testes unitários (Essencial para nota máxima)
├── .env                # Variáveis de ambiente (DB_URL, JWT_SECRET)
├── package.json        # Dependências
└── DOCUMENTACAO.md     # O texto científico ajustado

const { Reserva, Veiculo } = require('../models');
const { Op } = require('sequelize');

class ReservaService {
    async criarReserva(veiculoId, clienteId, dataIni, dataFim) {
        // 1. Verificação de existência e status do veículo
        const veiculo = await Veiculo.findByPk(veiculoId);
        if (!veiculo || veiculo.status !== 'Disponível') {
            throw new Error("Veículo indisponível ou não encontrado.");
        }

        // 2. Validação de Regra de Negócio: Anti-Overbooking
        // Verifica se já existe reserva ativa para este veículo no período selecionado
        const conflito = await Reserva.findOne({
            where: {
                veiculo_id: veiculoId,
                status: { [Op.ne]: 'Cancelada' },
                [Op.or]: [
                    { data_inicio: { [Op.between]: [dataIni, dataFim] } },
                    { data_fim: { [Op.between]: [dataIni, dataFim] } }
                ]
            }
        });

        if (conflito) {
            throw new Error("Este veículo já possui uma reserva para o período selecionado.");
        }

        // 3. Cálculo de Valor (Lógica de Negócio)
        const dias = Math.ceil(Math.abs(new Date(dataFim) - new Date(dataIni)) / (1000 * 60 * 60 * 24));
        const valorTotal = dias * veiculo.valorDiaria;

        // 4. Persistência ACID (Conforme Date, 2004)
        return await Reserva.create({
            veiculo_id: veiculoId,
            cliente_id: clienteId,
            data_inicio: dataIni,
            data_fim: dataFim,
            valor_total: valorTotal,
            status: 'Pendente'
        });
    }
}

module.exports = new ReservaService();

Funcionalidade,Papel da Implementação,Tecnologia Chave
Login,Autenticação JWT com criptografia bcrypt,Node.js + JWT
Gestão de Reservas,Lógica de não-sobreposição (Anti-overbooking),PostgreSQL (Queries de data)
Vistoria Digital,Upload de fotos e checklist de avarias,AWS S3 + React
Cálculo de Tarifas,Processamento automático de valores,Node.js (Regras de Negócio)
