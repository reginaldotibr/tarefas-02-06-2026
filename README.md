###  — Inicializar o projeto Node.js
````Execute o comando`:
```bash
npm init -y
```
## 2 dependencias do projeto
npm install express mysql2 bcryptjs express-session ejs prisma @prisma/client

Criar toda a estrutura de pastas


  New-Item -ItemType Directory -Force -Path "src\models"
  New-Item -ItemType Directory -Force -Path "src\controllers"
  New-Item -ItemType Directory -Force -Path "src\routes"
  New-Item -ItemType Directory -Force -Path "src\middlewares"
  New-Item -ItemType Directory -Force -Path "src\views\tarefas"
  New-Item -ItemType Directory -Force -Path "src\prisma"
  New-Item -ItemType Directory -Force -Path "public"

— Criar todos os arquivos 

  New-Item -ItemType File -Path "server.js"
  New-Item -ItemType File -Path "db.js"
  New-Item -ItemType File -Path ".env"
  New-Item -ItemType File -Path "src\models\tarefaModel.js"
  New-Item -ItemType File -Path "src\models\usuarioModel.js"
  New-Item -ItemType File -Path "src\controllers\tarefaController.js"
  New-Item -ItemType File -Path "src\controllers\usuarioController.js"
  New-Item -ItemType File -Path "src\controllers\authController.js"
  New-Item -ItemType File -Path "src\routes\tarefasRoutes.js"
  New-Item -ItemType File -Path "src\routes\usuariosRoutes.js"
  New-Item -ItemType File -Path "src\routes\authRoutes.js"
  New-Item -ItemType File -Path "src\routes\viewRoutes.js"
  New-Item -ItemType File -Path "src\middlewares\authMiddleware.js"
  New-Item -ItemType File -Path "src\views\layout.ejs"
  New-Item -ItemType File -Path "src\views\login.ejs"
  New-Item -ItemType File -Path "src\views\tarefas\index.ejs"
  New-Item -ItemType File -Path "src\views\tarefas\form.ejs"
  New-Item -ItemType File -Path "src\prisma\client.js"



— Criar o banco de dados e as tabelas no MySQL

  Abra o MySQL Workbench e execute:

  CREATE DATABASE sistema_tarefas;

  USE sistema_tarefas;

  CREATE TABLE tarefas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(200) NOT NULL,
    descricao TEXT,
    status VARCHAR(50) DEFAULT 'pendente',
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );

  CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    senha VARCHAR(255) NOT NULL,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );



.env e cole:

  DATABASE_URL="mysql://root:sua_senha@localhost:3306/sistema_tarefas"

  ▎ Troque sua_senha pela senha do seu MySQL.


# Preencher o arquivo db.js

 Abra db.js e cole:

  const mysql = require('mysql2');

  const conexao = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'sua_senha',      // troque pela sua senha
    database: 'sistema_tarefas'
  });

  conexao.connect((err) => {
    if (err) {
      console.error('Erro ao conectar ao MySQL:', err.message);
      return;
    }
    console.log('Conectado ao MySQL!');
  });

  module.exports = conexao;

#  — Preencher o server.js


  const express = require('express');
  const session = require('express-session');
  const path = require('path');

  const tarefasRoutes    = require('./src/routes/tarefasRoutes');
  const usuariosRoutes   = require('./src/routes/usuariosRoutes');
  const authRoutes       = require('./src/routes/authRoutes');
  const viewRoutes       = require('./src/routes/viewRoutes');

  const app = express();


# // Configurar EJS como template engine
  app.set('view engine', 'ejs');
  app.set('views', path.join(__dirname, 'src/views'));


#  // Servir arquivos estáticos (CSS, imagens)
  app.use(express.static('public'));

 # // Interpretar JSON e formulários HTML
  app.use(express.json());
  app.use(express.urlencoded({ extended: true }));

 # // Configurar sessões
  app.use(session({
    secret: 'segredo_super_secreto',
    resave: false,
    saveUninitialized: false,
    cookie: { maxAge: 1000 * 60 * 60 } // 1 hora
  }));

 # // Registrar rotas da API
  app.use('/api/tarefas',   tarefasRoutes);
  app.use('/api/usuarios',  usuariosRoutes);
  app.use('/api',           authRoutes);

 # // Registrar rotas do navegador (views)
  app.use('/', viewRoutes);

  app.listen(3000, () => console.log('Servidor rodando na porta 3000'));

# — CRUD DE TAREFAS

# — Preencher tarefaModel.js   Abra src/models/tarefaModel.js e cole:

 const db = require('../../db');

  const listar = (callback) => {
    db.query('SELECT * FROM tarefas ORDER BY criado_em DESC', callback);
  };

  const buscarPorId = (id, callback) => {
    db.query('SELECT * FROM tarefas WHERE id = ?', [id], callback);
  };

  const criar = (dados, callback) => {
    const sql = 'INSERT INTO tarefas (titulo, descricao, status) VALUES (?, ?, ?)';
    db.query(sql, [dados.titulo, dados.descricao, dados.status || 'pendente'], callback);
  };

  const atualizar = (id, dados, callback) => {
    const sql = 'UPDATE tarefas SET titulo=?, descricao=?, status=? WHERE id=?';
    db.query(sql, [dados.titulo, dados.descricao, dados.status, id], callback);
  };

  const excluir = (id, callback) => {
    db.query('DELETE FROM tarefas WHERE id = ?', [id], callback);
  };

  module.exports = { listar, buscarPorId, criar, atualizar, excluir };

#  — Preencher tarefaController.js   Abra src/controllers/tarefaController.js e cole:

  const tarefaModel = require('../models/tarefaModel');

  const listar = (req, res) => {
    tarefaModel.listar((err, tarefas) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.json(tarefas);
    });
  };

  const buscarPorId = (req, res) => {
    tarefaModel.buscarPorId(req.params.id, (err, resultado) => {
      if (err) return res.status(500).json({ erro: err.message });
      if (resultado.length === 0) return res.status(404).json({ mensagem: 'Tarefa não encontrada' });
      res.json(resultado[0]);
    });
  };

  const criar = (req, res) => {
    tarefaModel.criar(req.body, (err, resultado) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.status(201).json({ mensagem: 'Tarefa criada!', id: resultado.insertId });
    });
  };

  const atualizar = (req, res) => {
    tarefaModel.atualizar(req.params.id, req.body, (err) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.json({ mensagem: 'Tarefa atualizada!' });
    });
  };

  const excluir = (req, res) => {
    tarefaModel.excluir(req.params.id, (err) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.json({ mensagem: 'Tarefa excluída com sucesso!' });
    });
  };

  module.exports = { listar, buscarPorId, criar, atualizar, excluir };


