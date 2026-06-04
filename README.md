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


 # — Preencher tarefasRoutes.js   Abra src/routes/tarefasRoutes.js e cole:
  const express = require('express');
  const router = express.Router();
  const tarefaController = require('../controllers/tarefaController');
  const { autenticar } = require('../middlewares/authMiddleware');

  router.get('/',      autenticar, tarefaController.listar);
  router.get('/:id',   autenticar, tarefaController.buscarPorId);
  router.post('/',     autenticar, tarefaController.criar);
  router.put('/:id',   autenticar, tarefaController.atualizar);
  router.delete('/:id',autenticar, tarefaController.excluir);

  module.exports = router;


 # — CRUD DE USUÁRIOS
# — Preencher usuarioModel.js   Abra src/models/usuarioModel.js e cole:

const db = require('../../db');

  const criar = (dados, callback) => {
    const sql = 'INSERT INTO usuarios (nome, email, senha) VALUES (?, ?, ?)';
    db.query(sql, [dados.nome, dados.email, dados.senha], callback);
  };

  const listar = (callback) => {
    db.query('SELECT id, nome, email, criado_em FROM usuarios', callback);
  };

  const buscarPorId = (id, callback) => {
    db.query('SELECT id, nome, email, criado_em FROM usuarios WHERE id = ?', [id], callback);
  };

  const buscarPorEmail = (email, callback) => {
    db.query('SELECT * FROM usuarios WHERE email = ?', [email], callback);
  };

  const excluir = (id, callback) => {
    db.query('DELETE FROM usuarios WHERE id = ?', [id], callback);
  };

  module.exports = { criar, listar, buscarPorId, buscarPorEmail, excluir };



# — Preencher usuarioController.js   Abra src/controllers/usuarioController.js e cole:

 const bcrypt = require('bcryptjs');
  const usuarioModel = require('../models/usuarioModel');

  const cadastrar = async (req, res) => {
    const { nome, email, senha } = req.body;
    const senhaHash = await bcrypt.hash(senha, 10);
    usuarioModel.criar({ nome, email, senha: senhaHash }, (err, resultado) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.status(201).json({ mensagem: 'Usuário cadastrado!', id: resultado.insertId });
    });
  };

  const listar = (req, res) => {
    usuarioModel.listar((err, usuarios) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.json(usuarios);
    });
  };

  const buscarPorId = (req, res) => {
    usuarioModel.buscarPorId(req.params.id, (err, resultado) => {
      if (err) return res.status(500).json({ erro: err.message });
      if (resultado.length === 0) return res.status(404).json({ mensagem: 'Usuário não encontrado' });
      res.json(resultado[0]);
    });
  };

  const excluir = (req, res) => {
    usuarioModel.excluir(req.params.id, (err) => {
      if (err) return res.status(500).json({ erro: err.message });
      res.json({ mensagem: 'Usuário excluído com sucesso!' });
    });
  };

  module.exports = { cadastrar, listar, buscarPorId, excluir };

# Preencher usuariosRoutes.js   Abra src/routes/usuariosRoutes.js e cole:

 const express = require('express');
  const router = express.Router();
  const usuarioController = require('../controllers/usuarioController');
  const { autenticar } = require('../middlewares/authMiddleware');

  router.post('/',     usuarioController.cadastrar);
  router.get('/',      autenticar, usuarioController.listar);
  router.get('/:id',   autenticar, usuarioController.buscarPorId);
  router.delete('/:id',autenticar, usuarioController.excluir);

  module.exports = router;



# — AUTENTICAÇÃO COM SESSÃO Preencher authMiddleware.js   Abra src/middlewares/authMiddleware.js e cole:

const autenticar = (req, res, next) => {
    if (req.session && req.session.usuario) {
      next();
    } else {
      // Se for requisição de API retorna JSON, senão redireciona
      if (req.originalUrl.startsWith('/api')) {
        res.status(401).json({ mensagem: 'Acesso negado. Faça login primeiro.' });
      } else {
        res.redirect('/login');
      }
    }
  };

  module.exports = { autenticar };


#  18 — Preencher authController.js    Abra src/controllers/authController.js e cole:

  const bcrypt = require('bcryptjs');
  const usuarioModel = require('../models/usuarioModel');

  const login = (req, res) => {
    const { email, senha } = req.body;
    usuarioModel.buscarPorEmail(email, async (err, resultado) => {
      if (err) return res.status(500).json({ erro: err.message });
      if (resultado.length === 0) return res.status(401).json({ mensagem: 'Email ou senha inválidos' });

      const usuario = resultado[0];
      const senhaValida = await bcrypt.compare(senha, usuario.senha);
      if (!senhaValida) return res.status(401).json({ mensagem: 'Email ou senha inválidos' });

      req.session.usuario = { id: usuario.id, nome: usuario.nome, email: usuario.email };
      res.json({ mensagem: 'Login realizado com sucesso!', usuario: req.session.usuario });
    });
  };

  const logout = (req, res) => {
    req.session.destroy((err) => {
      if (err) return res.status(500).json({ erro: 'Erro ao encerrar sessão' });
      res.json({ mensagem: 'Logout realizado com sucesso!' });
    });
  };

  module.exports = { login, logout };

 # — Preencher authRoutes.js    Abra src/routes/authRoutes.js e cole:
const express = require('express');
  const router = express.Router();
  const authController = require('../controllers/authController');

  router.post('/login',  authController.login);
  router.post('/logout', authController.logout);

  module.exports = router;


# — FRONTEND COM EJS

# — Preencher layout.ejs   Abra src/views/layout.ejs e cole:

  <!DOCTYPE html>
  <html lang="pt-br">
  <head>
    <meta charset="UTF-8">
    <title><%= titulo %></title>
    <style>
      body { font-family: Arial, sans-serif; margin: 0; background: #f4f4f4; }
      nav { background: #2c3e50; padding: 12px 24px; display: flex; justify-content: space-between; align-items: center; }
      nav a { color: white; text-decoration: none; margin-right: 16px; }
      nav span { color: #ccc; font-size: 14px; }
      .container { max-width: 860px; margin: 30px auto; background: white; padding: 24px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
      table { width: 100%; border-collapse: collapse; margin-top: 16px; }
      th, td { padding: 10px 14px; border: 1px solid #ddd; text-align: left; }
      th { background: #2c3e50; color: white; }
      tr:nth-child(even) { background: #f9f9f9; }
      input, select, textarea { width: 100%; padding: 8px; margin: 5px 0 14px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
      button, .btn { padding: 9px 18px; background: #2c3e50; color: white; border: none; cursor: pointer; border-radius: 4px; text-decoration: none; display:
  inline-block; }
      .btn-excluir { background: #c0392b; padding: 6px 12px; }
      .btn-editar  { background: #2980b9; padding: 6px 12px; }
      .btn-novo    { background: #27ae60; margin-bottom: 14px; }
      .badge-pendente    { background: #e67e22; color: white; padding: 3px 8px; border-radius: 4px; font-size: 12px; }
      .badge-concluida   { background: #27ae60; color: white; padding: 3px 8px; border-radius: 4px; font-size: 12px; }
      .badge-em-andamento{ background: #2980b9; color: white; padding: 3px 8px; border-radius: 4px; font-size: 12px; }
    </style>
  </head>
  <body>
    <nav>
      <div>
        <a href="/tarefas">Tarefas</a>
        <a href="/tarefas/nova">+ Nova Tarefa</a>
      </div>
      <div>
        <% if (usuario) { %>
          <span>Olá, <%= usuario.nome %></span>
          <a href="/logout" style="margin-left:14px; color:#e74c3c;">Sair</a>
        <% } %>
      </div>
    </nav>
    <div class="container">
      <%- corpo %>
    </div>
  </body>
  </html>


#  — Preencher login.ejs   Abra src/views/login.ejs e cole:

  <!DOCTYPE html>
  <html lang="pt-br">
  <head>
    <meta charset="UTF-8">
    <title>Login</title>
    <style>
      body { font-family: Arial, sans-serif; background: #f4f4f4; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
      .box { background: white; padding: 32px; border-radius: 8px; width: 340px; box-shadow: 0 2px 8px rgba(0,0,0,0.15); }
      h2 { text-align: center; color: #2c3e50; margin-bottom: 20px; }
      label { font-size: 14px; color: #555; }
      input { width: 100%; padding: 9px; margin: 5px 0 14px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
      button { width: 100%; padding: 10px; background: #2c3e50; color: white; border: none; cursor: pointer; border-radius: 4px; font-size: 15px; }
      .erro { color: #c0392b; text-align: center; margin-bottom: 12px; font-size: 14px; }
    </style>
  </head>
  <body>
    <div class="box">
      <h2>Sistema de Tarefas</h2>
      <% if (typeof erro !== 'undefined') { %>
        <p class="erro"><%= erro %></p>
      <% } %>
      <form method="POST" action="/login">
        <label>Email</label>
        <input type="email" name="email" required placeholder="seu@email.com">
        <label>Senha</label>
        <input type="password" name="senha" required placeholder="••••••••">
        <button type="submit">Entrar</button>
      </form>
    </div>
  </body>
  </html>

# — Preencher tarefas/index.ejs   Abra src/views/tarefas/index.ejs e cole:

 <%
  const corpo = `
    <h2>Lista de Tarefas</h2>
    <a href="/tarefas/nova" class="btn btn-novo">+ Nova Tarefa</a>
    <table>
      <tr>
        <th>ID</th><th>Título</th><th>Status</th><th>Ações</th>
      </tr>
      ${ tarefas.length === 0
        ? '<tr><td colspan="4" style="text-align:center">Nenhuma tarefa cadastrada.</td></tr>'
        : tarefas.map(t => `
          <tr>
            <td>${t.id}</td>
            <td>${t.titulo}</td>
            <td><span class="badge-${t.status.replace(' ','-')}">${t.status}</span></td>
            <td>
              <a href="/tarefas/${t.id}/editar" class="btn btn-editar">Editar</a>
              <form method="POST" action="/tarefas/${t.id}/excluir" style="display:inline">
                <button class="btn btn-excluir" onclick="return confirm('Excluir esta tarefa?')">Excluir</button>
              </form>
            </td>
          </tr>
        `).join('')
      }
    </table>
  `;
  %>
  <%- include('../layout', { titulo: 'Tarefas', corpo, usuario }) %>

#  — Preencher tarefas/form.ejs   Abra src/views/tarefas/form.ejs e cole:
  <%
  const corpo = `
    <h2>${ tarefa ? 'Editar Tarefa' : 'Nova Tarefa' }</h2>
    <form method="POST" action="${ tarefa ? '/tarefas/' + tarefa.id + '/editar' : '/tarefas' }">
      <label>Título</label>
      <input type="text" name="titulo" value="${ tarefa ? tarefa.titulo : '' }" required>

      <label>Descrição</label>
      <textarea name="descricao" rows="4">${ tarefa ? tarefa.descricao || '' : '' }</textarea>

      <label>Status</label>
      <select name="status">
        <option value="pendente"      ${ tarefa && tarefa.status === 'pendente'      ? 'selected' : '' }>Pendente</option>
        <option value="em andamento"  ${ tarefa && tarefa.status === 'em andamento'  ? 'selected' : '' }>Em andamento</option>
        <option value="concluida"     ${ tarefa && tarefa.status === 'concluida'     ? 'selected' : '' }>Concluída</option>
      </select>

      <button type="submit">${ tarefa ? 'Salvar alterações' : 'Criar Tarefa' }</button>
      <a href="/tarefas" style="margin-left:12px; color:#555;">Cancelar</a>
    </form>
  `;
  %>
  <%- include('../layout', { titulo: tarefa ? 'Editar Tarefa' : 'Nova Tarefa', corpo, usuario }) %>


# — Preencher viewRoutes.js   Abra src/routes/viewRoutes.js e cole:

  const express = require('express');
  const router = express.Router();
  const bcrypt = require('bcryptjs');
  const db = require('../../db');
  const tarefaModel = require('../models/tarefaModel');
  const usuarioModel = require('../models/usuarioModel');
  const { autenticar } = require('../middlewares/authMiddleware');

  // Raiz — redireciona conforme login
  router.get('/', (req, res) => {
    res.redirect(req.session.usuario ? '/tarefas' : '/login');
  });

  // Exibe formulário de login
  router.get('/login', (req, res) => {
    res.render('login');
  });

  // Processa login do formulário
  router.post('/login', (req, res) => {
    const { email, senha } = req.body;
    usuarioModel.buscarPorEmail(email, async (err, resultado) => {
      if (err || resultado.length === 0) return res.render('login', { erro: 'Email ou senha inválidos' });
      const valido = await bcrypt.compare(senha, resultado[0].senha);
      if (!valido) return res.render('login', { erro: 'Email ou senha inválidos' });
      req.session.usuario = { id: resultado[0].id, nome: resultado[0].nome };
      res.redirect('/tarefas');
    });
  });

  // Logout
  router.get('/logout', (req, res) => {
    req.session.destroy(() => res.redirect('/login'));
  });

  // Lista de tarefas
  router.get('/tarefas', autenticar, (req, res) => {
    tarefaModel.listar((err, tarefas) => {
      if (err) return res.status(500).send('Erro ao buscar tarefas');
      res.render('tarefas/index', { tarefas, usuario: req.session.usuario });
    });
  });

  // Formulário nova tarefa
  router.get('/tarefas/nova', autenticar, (req, res) => {
    res.render('tarefas/form', { tarefa: null, usuario: req.session.usuario });
  });

  // Criar tarefa via formulário
  router.post('/tarefas', autenticar, (req, res) => {
    tarefaModel.criar(req.body, (err) => {
      if (err) return res.status(500).send('Erro ao criar tarefa');
      res.redirect('/tarefas');
    });
  });

  // Formulário editar tarefa
  router.get('/tarefas/:id/editar', autenticar, (req, res) => {
    tarefaModel.buscarPorId(req.params.id, (err, resultado) => {
      if (err || resultado.length === 0) return res.redirect('/tarefas');
      res.render('tarefas/form', { tarefa: resultado[0], usuario: req.session.usuario });
    });
  });

  // Salvar edição via formulário
  router.post('/tarefas/:id/editar', autenticar, (req, res) => {
    tarefaModel.atualizar(req.params.id, req.body, (err) => {
      if (err) return res.status(500).send('Erro ao atualizar tarefa');
      res.redirect('/tarefas');
    });
  });

  // Excluir tarefa via formulário
  router.post('/tarefas/:id/excluir', autenticar, (req, res) => {
    tarefaModel.excluir(req.params.id, (err) => {
      if (err) return res.status(500).send('Erro ao excluir tarefa');
      res.redirect('/tarefas');
    });
  });

  module.exports = router;



###  — PRISMA ORM  — Inicializar o Prisma
# — Inicializar o Prisma # ▎ Isso atualiza o .env e cria prisma/schema.prisma
npx prisma init

 # — Configurar o schema.prisma

