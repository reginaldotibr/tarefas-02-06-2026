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


