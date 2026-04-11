src/config/database.js, src/models/index.js, src/models/Veiculo.js, src/models/Reserva.js, src/services/ReservaService.js e src/app.js.
const { Sequelize } = require('sequelize');
require('dotenv').config();

// Conexão com Postgres via URL da variável de ambiente
const sequelize = new Sequelize(process.env.DB_URL, {
    dialect: 'postgres',
    logging: false,
    define: { timestamps: true, underscored: true }
});

module.exports = sequelize;
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Veiculo = sequelize.define('Veiculo', {
    modelo: DataTypes.STRING,
    status: { type: DataTypes.STRING, defaultValue: 'Disponível' },
    valorDiaria: { type: DataTypes.FLOAT, field: 'valor_diaria' }
});

module.exports = Veiculo;
const Veiculo = require('../models/Veiculo');
const Reserva = require('../models/Reserva');
const { Op } = require('sequelize');

class ReservaService {
    async criarReserva(veiculoId, clienteId, dataIni, dataFim) {
        // 1. Validação de Disponibilidade
        const veiculo = await Veiculo.findByPk(veiculoId);
        if (!veiculo || veiculo.status !== 'Disponível') {
            throw new Error("Veículo indisponível ou não encontrado.");
        }

        // 2. Lógica Anti-Overbooking (Conflito de datas)
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

        if (conflito) throw new Error("Conflito de datas: Veículo já reservado.");

        // 3. Cálculo de Diárias e Valor Total
        const d1 = new Date(dataIni);
        const d2 = new Date(dataFim);
        const dias = Math.ceil(Math.abs(d2 - d1) / (1000 * 60 * 60 * 24));
        const valorTotal = (dias || 1) * veiculo.valorDiaria;

        // 4. Persistência
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
{
  "name": "sistema-aluguel-veiculos",
  "version": "1.0.0",
  "main": "src/app.js",
  "scripts": {
    "start": "node src/app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "sequelize": "^6.31.1",
    "pg": "^8.10.0",
    "dotenv": "^16.0.3"
  }
}
Camada,Arquivo,Responsabilidade
Persistência,database.js,Garante propriedades ACID no PostgreSQL.
Domínio,Veiculo.js,Define os estados do objeto (Disponível/Manutenção).
Negócio,ReservaService.js,Executa o cálculo de tarifas e bloqueia o Overbooking.
Interface,ReservaController.js,Traduz requisições HTTP para chamadas de serviço.
const express = require('express');
const sequelize = require('./config/database');
const reservaRoutes = require('./routes/reservaRoutes');

const app = express();
app.use(express.json()); // Permite que o app entenda JSON no corpo das requisições

// Rotas
app.use('/reservas', reservaRoutes);

// Sincronização com o Banco de Dados e Inicialização
const PORT = process.env.PORT || 3000;

sequelize.sync().then(() => {
    console.log('Banco de dados sincronizado.');
    app.listen(PORT, () => {
        console.log(`Servidor rodando na porta ${PORT}`);
    });
}).catch(err => {
    console.error('Erro ao conectar ao banco:', err);
});
const ReservaService = require('../services/ReservaService');

class ReservaController {
    async store(req, res) {
        try {
            const { veiculoId, clienteId, dataIni, dataFim } = req.body;
            
            // Chama a lógica de negócio que você já definiu
            const reserva = await ReservaService.criarReserva(veiculoId, clienteId, dataIni, dataFim);
            
            return res.status(201).json(reserva);
        } catch (error) {
            // Retorna erros de negócio (ex: overbooking) de forma clara
            return res.status(400).json({ error: error.message });
        }
    }
}

module.exports = new ReservaController();
const express = require('express');
const router = express.Router();
const ReservaController = require('../controllers/ReservaController');

// Define que o método POST em /reservas criará uma reserva
router.post('/', ReservaController.store);

module.exports = router;
