# VENDAS-PDV

## 🧩 MÓDULO 2 — VENDAS / PDV
🎯 Objetivo

Gerenciar pedidos, orçamentos, faturamentos e histórico de vendas.
Cada venda será relacionada a um cliente e conterá itens (produtos), quantidade, valor unitário, total, e status ("orcamento", "finalizado", "cancelado").

# 📁 1️⃣ Estrutura de Pastas

No backend:

modules/

└── vendas/

    ├── venda.model.js

    ├── venda.controller.js

    └── venda.routes.js

No frontend:

pages/

└── Vendas/

    ├── VendasList.jsx

    ├── VendaForm.jsx

    └── VendaDetalhes.jsx

## 🧠 2️⃣ Backend — Estrutura do módulo de vendas
📌 venda.model.js

import db from '../../database/connection.js';

import { DataTypes } from 'sequelize';

import Cliente from '../crm/crm.model.js'; // integra com CRM

const Venda = db.define('Venda', {

  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  cliente_id: { type: DataTypes.INTEGER, allowNull: false },

  data_venda: { type: DataTypes.DATE, defaultValue: DataTypes.NOW },
  total: { type: DataTypes.DECIMAL(10,2), allowNull: false },

  status: { 

    type: DataTypes.ENUM('orcamento', 'finalizado', 'cancelado'),

    defaultValue: 'orcamento'
  },
});

// Relacionamento
Venda.belongsTo(Cliente, { foreignKey: 'cliente_id' });

export default Venda;

📌 venda.controller.js

import Venda from './venda.model.js';
import Cliente from '../crm/crm.model.js';

// Criar uma nova venda
export async function criarVenda(req, res) {
  try {
    const venda = await Venda.create(req.body);
    res.status(201).json(venda);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
}

// Listar todas as vendas
export async function listarVendas(req, res) {
  const vendas = await Venda.findAll({ include: Cliente });
  res.json(vendas);
}

// Buscar venda por ID
export async function buscarVenda(req, res) {
  const venda = await Venda.findByPk(req.params.id, { include: Cliente });
  if (!venda) return res.status(404).json({ error: 'Venda não encontrada' });
  res.json(venda);
}

// Atualizar venda
export async function atualizarVenda(req, res) {
  const venda = await Venda.findByPk(req.params.id);
  if (!venda) return res.status(404).json({ error: 'Venda não encontrada' });
  await venda.update(req.body);
  res.json(venda);
}

// Deletar venda
export async function excluirVenda(req, res) {
  const venda = await Venda.findByPk(req.params.id);
  if (!venda) return res.status(404).json({ error: 'Venda não encontrada' });
  await venda.destroy();
  res.json({ message: 'Venda excluída com sucesso' });
}

📌 venda.routes.js

import express from 'express';
import {
  criarVenda,
  listarVendas,
  buscarVenda,
  atualizarVenda,
  excluirVenda
} from './venda.controller.js';

const router = express.Router();

router.post('/vendas', criarVenda);
router.get('/vendas', listarVendas);
router.get('/vendas/:id', buscarVenda);
router.put('/vendas/:id', atualizarVenda);
router.delete('/vendas/:id', excluirVenda);

export default router;

E adicione no main.ts (ou server.ts):

import vendaRoutes from './modules/vendas/venda.routes.js';
app.use('/api', vendaRoutes);


💾 3️⃣ Banco de Dados

Adicione no schema.sql:

CREATE TABLE vendas (
  id INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id INT NOT NULL,
  data_venda DATETIME DEFAULT CURRENT_TIMESTAMP,
  total DECIMAL(10,2) NOT NULL,
  status ENUM('orcamento', 'finalizado', 'cancelado') DEFAULT 'orcamento',
  FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

⚙️ 4️⃣ Frontend — React (Módulo Vendas)
📌 VendasList.jsx
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function VendasList() {
  const [vendas, setVendas] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:3001/api/vendas')
      .then(res => setVendas(res.data))
      .catch(err => console.error(err));
  }, []);

  return (
    <div className="p-4">
      <h2 className="text-xl font-semibold mb-3">Vendas Realizadas</h2>
      <table className="w-full border">
        <thead>
          <tr>
            <th>Cliente</th>
            <th>Data</th>
            <th>Total</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          {vendas.map(v => (
            <tr key={v.id}>
              <td>{v.Cliente?.nome}</td>
              <td>{new Date(v.data_venda).toLocaleDateString()}</td>
              <td>R$ {v.total}</td>
              <td>{v.status}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

📌 VendaForm.jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

export default function VendaForm() {
  const [clientes, setClientes] = useState([]);
  const [form, setForm] = useState({ cliente_id: '', total: '', status: 'orcamento' });

  useEffect(() => {
    axios.get('http://localhost:3001/api/clientes')
      .then(res => setClientes(res.data));
  }, []);

  const handleChange = e => setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = async e => {
    e.preventDefault();
    await axios.post('http://localhost:3001/api/vendas', form);
    alert('Venda registrada!');
    setForm({ cliente_id: '', total: '', status: 'orcamento' });
  };

  return (
    <form onSubmit={handleSubmit} className="p-4 bg-gray-100 rounded">
      <h2 className="text-lg font-semibold mb-3">Registrar Venda</h2>
      <select name="cliente_id" value={form.cliente_id} onChange={handleChange} className="block mb-2 p-1 border">
        <option value="">Selecione o Cliente</option>
        {clientes.map(cli => (
          <option key={cli.id} value={cli.id}>{cli.nome}</option>
        ))}
      </select>
      <input name="total" type="number" step="0.01" placeholder="Total" value={form.total} onChange={handleChange} className="block mb-2 p-1 border" />
      <select name="status" value={form.status} onChange={handleChange} className="block mb-2 p-1 border">
        <option value="orcamento">Orçamento</option>
        <option value="finalizado">Finalizado</option>
        <option value="cancelado">Cancelado</option>
      </select>
      <button className="bg-green-600 text-white px-3 py-1 rounded">Salvar</button>
    </form>
  );
}

📌 VendaDetalhes.jsx
export default function VendaDetalhes({ venda }) {
  if (!venda) return null;
  return (
    <div>
      <h3 className="text-lg font-semibold">Detalhes da Venda</h3>
      <p><strong>Cliente:</strong> {venda.Cliente?.nome}</p>
      <p><strong>Total:</strong> R$ {venda.total}</p>
      <p><strong>Status:</strong> {venda.status}</p>
      <p><strong>Data:</strong> {new Date(venda.data_venda).toLocaleString()}</p>
    </div>
  );
}

🔗 5️⃣ Integrações futuras
Módulo	Integração	Descrição
CRM	cliente_id	Cada venda pertence a um cliente.
Estoque	produtos_venda	Futuro: registrar movimentação de estoque.
Finanças	contas_receber	Futuro: gerar registro financeiro da venda.
Contador	NF-e	Enviar dados fiscais para emissão de nota.
🧭 6️⃣ Resumo

✅ CRUD de Vendas (com relacionamento Cliente)
✅ Integração inicial com CRM
✅ Estrutura React funcional (listagem + formulário)
✅ Preparado para integrar com Estoque e Financeiro


