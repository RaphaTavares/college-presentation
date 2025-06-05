# 🚀 Workshop: Construindo uma API de Login do Zero

## Slide 1: Bem-vindos ao Workshop!

**Construindo uma API de Login com Node.js, Express, Prisma e PostgreSQL**

🎯 **Objetivo**: Criar uma API REST completa com autenticação JWT

📋 **O que vamos construir**:
- Cadastro de usuários (signup)
- Login de usuários (signin)
- Visualização de perfil (protegido)
- Atualização de perfil (protegido)

🛠️ **Tecnologias**:
- Node.js + Express.js
- Prisma ORM + PostgreSQL
- JWT + bcrypt
- Docker

---

## Minha Timeline Profissional

### 10/2020 - 01/2021
**DBA Estagiário** | MRS
- Aprendi nada como DBA
- Primeiro estágio profissional
- R$ 650 + 800 = 1450

---

### 01/2021 - 10/2021
**DevOps Estagiário** | Thomson Reuters
- Aprendi pra caramba
- Infraestrutura e DevOps
- R$ 1.600 + 600 = 2.200

---

### 10/2021 - 01/2022
**Backend Developer Junior** | Ilumeo
- Construi um sistema horrível
- Estudei pra melhorar
- R$ 4.300 + 600 = 4.900

---

### 01/2022 - 11/2023
**Backend Developer Pleno** | BTG Pactual
- Vendi minha alma e trabalhei 12 horas por dia por 2 anos
- Aprendi muito
- R$ 6150 + 800 = 6.750
- R$ 6550 + 800 = 7.350
- R$ 7100 + 1800 = 8.900

---

### 11/2023 - 09/2024
**Team Leader** | Wake Ecommerce
- Liderei dois devs (1 pleno e 1 junior)
- Fui o braço direito do meu gestor
- Liderança técnica
- R$ 10.500 + Não lembro = ?

---

### 02/2024 - Presente
**Sócio** | Lentium
- Abri minha própria empresa com 4 sócios que conheci aqui na faculdade
- Modelagem é importante pra caramba
- R$ 100 - 500 = - 400
---

### 09/2024 - 11/2024
**Senior Backend Developer** | Emerge
- Primeiro emprego pro exterior
- Sofri layoff com 50 dias
- USD 4.386 = R$ 24.800
---


### 12/2024 - 01/2025
**Desenvolvedor Pleno** | Petrobras
- Negociei mas mesmo assim o cara me contratou como pleno
- Saí com 3 semanas
- R$ 7.500 + Não lembro = ?
---

### 01/2025 - 04/2025
**Desenvolvedor Senior** | Bravium
- Tive baixa performance por estar procurando emprego pro exterior
- R$ 13.200 + não lembro = ?
---

### 04/2025 - Presente
**Desenvolvedor Senior** | Collectors Universe
- Segundo emprego pro exterior
- Desenvolvimento backend
- USD 4400 = R$ 25.000

## Slide 2: Preparação do Ambiente

### Passo 1: Inicializar o projeto

```bash
npm init -y
```

### Passo 2: Instalar dependências

```bash
npm install express bcryptjs jsonwebtoken express-validator cors nodemon dotenv
npm install prisma @prisma/client
```

### Passo 3: Configurar Docker para PostgreSQL

**Arquivo: `docker-compose.yml`**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: workshop_postgres
    environment:
      POSTGRES_DB: workshop_db
      POSTGRES_USER: workshop_user
      POSTGRES_PASSWORD: workshop_password
    ports:
      - "5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## Slide 3: Configuração do Prisma

### Passo 4: Inicializar Prisma

```bash
npx prisma init
```

### Passo 5: Configurar variáveis de ambiente

**Arquivo: `.env`**
```env
# Server Configuration
PORT=3000
NODE_ENV=development

# Database Configuration
DB_HOST=localhost
DB_PORT=5433
DB_NAME=workshop_db
DB_USER=workshop_user
DB_PASSWORD=workshop_password

# Prisma Database URL
DATABASE_URL="postgresql://workshop_user:workshop_password@localhost:5433/workshop_db?schema=public"

# JWT Configuration
JWT_SECRET=your_super_secret_jwt_key_change_this_in_production
JWT_EXPIRES_IN=24h
```

### Passo 6: Configurar Schema do Prisma

**Arquivo: `prisma/schema.prisma`**
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  firstName String   @map("first_name")
  lastName  String   @map("last_name")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")
}
```

---

## Slide 4: Configuração do Banco e Migrations

### Passo 7: Atualizar package.json com scripts Prisma

**Arquivo: `package.json`** (adicionar aos scripts)
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "db:up": "docker-compose up -d",
    "db:down": "docker-compose down",
    "db:logs": "docker-compose logs postgres",
    "db:migrate": "npx prisma migrate dev",
    "db:generate": "npx prisma generate",
    "db:studio": "npx prisma studio",
    "db:reset": "npx prisma migrate reset"
  }
}
```

### Passo 8: Iniciar banco e criar primeira migration

```bash
# Iniciar PostgreSQL
npm run db:up

# Aguardar alguns segundos e criar migration
npm run db:migrate
```

**Nome da migration**: `init`

### Passo 9: Gerar Prisma Client

```bash
npm run db:generate
```

---

## Slide 5: Servidor Básico (Tudo em um arquivo)

### Passo 10: Criar servidor básico

**Arquivo: `server.js`** (Versão inicial)
```javascript
const express = require('express');
const cors = require('cors');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const { body, validationResult } = require('express-validator');
const { PrismaClient } = require('./generated/prisma');

require('dotenv').config();

const app = express();
const prisma = new PrismaClient();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Request logging middleware
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    success: true,
    message: 'Server is running',
    timestamp: new Date().toISOString()
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Global error handler
app.use((error, req, res, next) => {
  console.error('Global error handler:', error);
  res.status(500).json({
    success: false,
    message: 'Internal server error'
  });
});

// Start server
const startServer = async () => {
  try {
    // Test database connection
    await prisma.$connect();
    console.log('Database connection successful');

    app.listen(PORT, () => {
      console.log(`Server is running on port ${PORT}`);
      console.log(`Health check: http://localhost:${PORT}/health`);
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
};

startServer();
```

---

## Slide 6: Adicionando Rota de Signup

### Passo 11: Adicionar validações e helper functions

**Arquivo: `server.js`** (Adicionar após as importações)
```javascript
// Regras de validação para cadastro
const signupValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Por favor, forneça um email válido'),
  body('password')
    .isLength({ min: 6 })
    .withMessage('A senha deve ter pelo menos 6 caracteres'),
  body('firstName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Nome é obrigatório'),
  body('lastName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Sobrenome é obrigatório')
];

// Função auxiliar para gerar token JWT
const generateToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRES_IN }
  );
};
```

### Passo 12: Adicionar rota de signup

**Arquivo: `server.js`** (Adicionar antes do 404 handler)
```javascript
// POST /api/auth/signup - Cadastro de usuário
app.post('/api/auth/signup', signupValidation, async (req, res) => {
  try {
    // Verificar erros de validação
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Falha na validação',
        errors: errors.array()
      });
    }

    const { email, password, firstName, lastName } = req.body;

    // Verificar se o usuário já existe
    const existingUser = await prisma.user.findUnique({
      where: { email }
    });

    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'Usuário com este email já existe'
      });
    }

    // Criptografar a senha
    const saltRounds = 12;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Criar novo usuário
    const newUser = await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        firstName,
        lastName
      },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        createdAt: true
      }
    });

    // Gerar token JWT
    const token = generateToken(newUser.id);

    res.status(201).json({
      success: true,
      message: 'Usuário criado com sucesso',
      data: {
        user: newUser,
        token
      }
    });

  } catch (error) {
    console.error('Erro no cadastro:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

---

## Slide 7: Adicionando Rota de Signin

### Passo 13: Adicionar validação para signin

**Arquivo: `server.js`** (Adicionar após signupValidation)
```javascript
// Regras de validação para login
const signinValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Por favor, forneça um email válido'),
  body('password')
    .notEmpty()
    .withMessage('Senha é obrigatória')
];
```

### Passo 14: Adicionar rota de signin

**Arquivo: `server.js`** (Adicionar após a rota de signup)
```javascript
// POST /api/auth/signin - Login do usuário
app.post('/api/auth/signin', signinValidation, async (req, res) => {
  try {
    // Verificar erros de validação
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Falha na validação',
        errors: errors.array()
      });
    }

    const { email, password } = req.body;

    // Buscar usuário pelo email
    const user = await prisma.user.findUnique({
      where: { email }
    });

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Email ou senha inválidos'
      });
    }

    // Verificar senha
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Email ou senha inválidos'
      });
    }

    // Gerar token JWT
    const token = generateToken(user.id);

    res.json({
      success: true,
      message: 'Login realizado com sucesso',
      data: {
        user: {
          id: user.id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName
        },
        token
      }
    });

  } catch (error) {
    console.error('Erro no login:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

---

## Slide 8: Adicionando Rotas de Perfil (Sem Autenticação)

### Passo 15: Adicionar validação para perfil

**Arquivo: `server.js`** (Adicionar após signinValidation)
```javascript
// Regras de validação para atualização de perfil
const updateProfileValidation = [
  body('firstName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Nome é obrigatório'),
  body('lastName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Sobrenome é obrigatório')
];
```

### Passo 16: Adicionar rota para visualizar perfil (temporariamente sem auth)

**Arquivo: `server.js`** (Adicionar após a rota de signin)
```javascript
// GET /api/profile - Visualizar perfil do usuário (temporariamente sem auth)
app.get('/api/profile', async (req, res) => {
  try {
    const { email } = req.body;

    // Verificar se o email foi fornecido
    if (!email) {
      return res.status(400).json({
        success: false,
        message: 'Email é obrigatório'
      });
    }

    // Buscar usuário pelo email
    const user = await prisma.user.findUnique({
      where: { email },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        createdAt: true,
        updatedAt: true
      }
    });

    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'Usuário não encontrado'
      });
    }

    res.json({
      success: true,
      message: 'Perfil recuperado com sucesso',
      data: {
        user: {
          id: user.id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName,
          createdAt: user.createdAt,
          updatedAt: user.updatedAt
        }
      }
    });

  } catch (error) {
    console.error('Erro ao buscar perfil:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

### Passo 17: Adicionar rota para atualizar perfil (temporariamente sem auth)

**Arquivo: `server.js`** (Adicionar após a rota GET /api/profile)
```javascript
// PUT /api/profile - Atualizar perfil do usuário (temporariamente sem auth)
app.put('/api/profile', updateProfileValidation, async (req, res) => {
  try {
    // Verificar erros de validação
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Falha na validação',
        errors: errors.array()
      });
    }

    const { firstName, lastName } = req.body;

    // Por enquanto, vamos atualizar o primeiro usuário para teste
    const firstUser = await prisma.user.findFirst();

    if (!firstUser) {
      return res.status(404).json({
        success: false,
        message: 'Nenhum usuário encontrado'
      });
    }

    // Atualizar perfil do usuário
    const updatedUser = await prisma.user.update({
      where: { id: firstUser.id },
      data: {
        firstName,
        lastName
      },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        updatedAt: true
      }
    });

    res.json({
      success: true,
      message: 'Perfil atualizado com sucesso',
      data: {
        user: updatedUser
      }
    });

  } catch (error) {
    console.error('Erro ao atualizar perfil:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

---

## Slide 9: Adicionando Middleware de Autenticação

### Passo 18: Criar middleware de autenticação

**Arquivo: `server.js`** (Adicionar após updateProfileValidation)
```javascript
// Middleware de autenticação
const authenticateToken = async (req, res, next) => {
  // Obter token do cabeçalho
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Token de acesso é obrigatório'
    });
  }

  try {
    // Verificar token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Buscar usuário no banco de dados
    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        createdAt: true,
        updatedAt: true
      }
    });

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Token inválido - usuário não encontrado'
      });
    }

    // Adicionar usuário ao objeto da requisição
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({
        success: false,
        message: 'Token inválido'
      });
    } else if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        success: false,
        message: 'Token expirado'
      });
    } else {
      console.error('Erro no middleware de autenticação:', error);
      return res.status(500).json({
        success: false,
        message: 'Erro interno do servidor'
      });
    }
  }
};
```

### Passo 19: Atualizar rotas de perfil para usar autenticação

**Atualizar a rota GET /api/profile:**
```javascript
// GET /api/profile - Visualizar perfil do usuário (agora com auth)
app.get('/api/profile', authenticateToken, async (req, res) => {
  try {
    // Usuário já está disponível através do middleware de auth
    const user = req.user;

    res.json({
      success: true,
      message: 'Perfil recuperado com sucesso',
      data: {
        user: {
          id: user.id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName,
          createdAt: user.createdAt,
          updatedAt: user.updatedAt
        }
      }
    });

  } catch (error) {
    console.error('Erro ao buscar perfil:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

**Atualizar a rota PUT /api/profile:**
```javascript
// PUT /api/profile - Atualizar perfil do usuário (agora com auth)
app.put('/api/profile', authenticateToken, updateProfileValidation, async (req, res) => {
  try {
    // Verificar erros de validação
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Falha na validação',
        errors: errors.array()
      });
    }

    const { firstName, lastName } = req.body;
    const userId = req.user.id; // Agora obtemos o ID do usuário através do token

    // Atualizar perfil do usuário
    const updatedUser = await prisma.user.update({
      where: { id: userId },
      data: {
        firstName,
        lastName
      },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        updatedAt: true
      }
    });

    res.json({
      success: true,
      message: 'Perfil atualizado com sucesso',
      data: {
        user: updatedUser
      }
    });

  } catch (error) {
    console.error('Erro ao atualizar perfil:', error);
    res.status(500).json({
      success: false,
      message: 'Erro interno do servidor'
    });
  }
});
```

---

## Slide 10: Testando a API (Primeira Versão)

### Passo 20: Executar o projeto

```bash
# Iniciar o banco de dados (se não estiver rodando)
npm run db:up

# Iniciar o servidor
npm start
```

### Teste 1: Cadastro de usuário
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123",
    "firstName": "Test",
    "lastName": "User"
  }'
```

### Teste 2: Login
```bash
curl -X POST http://localhost:3000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

### Teste 3: Ver perfil (sem autenticação ainda)
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com"
  }'
```

### Teste 4: Atualizar perfil (sem autenticação ainda)
```bash
curl -X PUT http://localhost:3000/api/profile \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Novo",
    "lastName": "Nome"
  }'
```

**Nota**: Neste ponto, as rotas de perfil ainda não requerem autenticação. Vamos adicionar isso no próximo passo!

---

## Slide 11: Testando com Autenticação

### Passo 21: Testar rotas protegidas

Agora que adicionamos autenticação, vamos testar novamente:

### Teste 1: Ver perfil (agora requer token)
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer SEU_TOKEN_AQUI"
```

### Teste 2: Atualizar perfil (agora requer token)
```bash
curl -X PUT http://localhost:3000/api/profile \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SEU_TOKEN_AQUI" \
  -d '{
    "firstName": "Novo",
    "lastName": "Nome"
  }'
```

### Teste 3: Tentar acessar sem token (deve dar erro)
```bash
curl -X GET http://localhost:3000/api/profile
```

**Resultado esperado**: Erro 401 - Token de acesso é obrigatório

---

## Slide 12: Refatoração - Separando em Arquivos

### Passo 22: Criar middleware separado

**Arquivo: `middleware/auth.js`**
```javascript
const jwt = require('jsonwebtoken');
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

const authenticateToken = async (req, res, next) => {
  // Get token from header
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access token is required'
    });
  }

  try {
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Get user from database
    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        createdAt: true,
        updatedAt: true
      }
    });

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid token - user not found'
      });
    }

    // Add user to request object
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({
        success: false,
        message: 'Invalid token'
      });
    } else if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        success: false,
        message: 'Token expired'
      });
    } else {
      console.error('Auth middleware error:', error);
      return res.status(500).json({
        success: false,
        message: 'Internal server error'
      });
    }
  }
};

module.exports = authenticateToken;
```
---

## Slide 13: Criar Rotas de Autenticação Separadas

### Passo 23: Criar arquivo de rotas de autenticação

**Arquivo: `routes/auth.js`**
```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const { body, validationResult } = require('express-validator');
const { PrismaClient } = require('@prisma/client');

const router = express.Router();
const prisma = new PrismaClient();

// Validation rules for signup
const signupValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters long'),
  body('firstName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('First name is required'),
  body('lastName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Last name is required')
];

// Validation rules for signin
const signinValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  body('password')
    .notEmpty()
    .withMessage('Password is required')
];

// Helper function to generate JWT token
const generateToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRES_IN }
  );
};

// POST /api/auth/signup - User registration
router.post('/signup', signupValidation, async (req, res) => {
  try {
    // Check for validation errors
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
    }

    const { email, password, firstName, lastName } = req.body;

    // Check if user already exists
    const existingUser = await prisma.user.findUnique({
      where: { email }
    });

    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'User with this email already exists'
      });
    }

    // Hash the password
    const saltRounds = 12;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Create new user
    const newUser = await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        firstName,
        lastName
      },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        createdAt: true
      }
    });

    // Generate JWT token
    const token = generateToken(newUser.id);

    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: {
        user: newUser,
        token
      }
    });

  } catch (error) {
    console.error('Signup error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});
```

---

## Slide 14: Completar Rotas de Autenticação

### Passo 24: Adicionar rota de signin

**Arquivo: `routes/auth.js`** (Continuação)
```javascript
// POST /api/auth/signin - User login
router.post('/signin', signinValidation, async (req, res) => {
  try {
    // Check for validation errors
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
    }

    const { email, password } = req.body;

    // Find user by email
    const user = await prisma.user.findUnique({
      where: { email }
    });

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }

    // Generate JWT token
    const token = generateToken(user.id);

    res.json({
      success: true,
      message: 'Login successful',
      data: {
        user: {
          id: user.id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName
        },
        token
      }
    });

  } catch (error) {
    console.error('Signin error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});

module.exports = router;
```

---

## Slide 15: Criar Rotas de Perfil Separadas

### Passo 25: Criar arquivo de rotas de perfil

**Arquivo: `routes/profile.js`**
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const { PrismaClient } = require('@prisma/client');
const authenticateToken = require('../middleware/auth');

const router = express.Router();
const prisma = new PrismaClient();

// Validation rules for profile update
const updateProfileValidation = [
  body('firstName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('First name is required'),
  body('lastName')
    .trim()
    .isLength({ min: 1 })
    .withMessage('Last name is required')
];

// GET /api/profile - View user profile
router.get('/', authenticateToken, async (req, res) => {
  try {
    // User is already available from the auth middleware
    const user = req.user;

    res.json({
      success: true,
      message: 'Profile retrieved successfully',
      data: {
        user: {
          id: user.id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName,
          createdAt: user.createdAt,
          updatedAt: user.updatedAt
        }
      }
    });

  } catch (error) {
    console.error('Get profile error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});

// PUT /api/profile - Update user profile
router.put('/', authenticateToken, updateProfileValidation, async (req, res) => {
  try {
    // Check for validation errors
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
    }

    const { firstName, lastName } = req.body;
    const userId = req.user.id;

    // Update user profile
    const updatedUser = await prisma.user.update({
      where: { id: userId },
      data: {
        firstName,
        lastName
      },
      select: {
        id: true,
        email: true,
        firstName: true,
        lastName: true,
        updatedAt: true
      }
    });

    res.json({
      success: true,
      message: 'Profile updated successfully',
      data: {
        user: updatedUser
      }
    });

  } catch (error) {
    console.error('Update profile error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
});

module.exports = router;
```

---

## Slide 16: Servidor Final Refatorado

### Passo 26: Atualizar servidor principal

**Arquivo: `server.js`** (Versão final)
```javascript
const express = require('express');
const cors = require('cors');
const { PrismaClient } = require('@prisma/client');

require('dotenv').config();

// Import routes
const authRoutes = require('./routes/auth');
const profileRoutes = require('./routes/profile');

const app = express();
const prisma = new PrismaClient();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Request logging middleware
app.use((req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
  next();
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    success: true,
    message: 'Server is running',
    timestamp: new Date().toISOString()
  });
});

// API Routes
app.use('/api/auth', authRoutes);
app.use('/api/profile', profileRoutes);

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Global error handler
app.use((error, req, res, next) => {
  console.error('Global error handler:', error);
  res.status(500).json({
    success: false,
    message: 'Internal server error'
  });
});

// Start server
const startServer = async () => {
  try {
    // Test database connection
    await prisma.$connect();
    console.log('Database connection successful');

    app.listen(PORT, () => {
      console.log(`Server is running on port ${PORT}`);
      console.log(`Health check: http://localhost:${PORT}/health`);
      console.log('\nAvailable endpoints:');
      console.log('POST /api/auth/signup - User registration');
      console.log('POST /api/auth/signin - User login');
      console.log('GET  /api/profile - View profile (requires auth)');
      console.log('PUT  /api/profile - Update profile (requires auth)');
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
};

startServer();
```

---

## Slide 17: Testando a API Final

### Teste 1: Cadastro de usuário
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123",
    "firstName": "Test",
    "lastName": "User"
  }'
```

### Teste 2: Login
```bash
curl -X POST http://localhost:3000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

### Teste 3: Ver perfil (usar token do login)
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer SEU_TOKEN_AQUI"
```

### Teste 4: Atualizar perfil
```bash
curl -X PUT http://localhost:3000/api/profile \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SEU_TOKEN_AQUI" \
  -d '{
    "firstName": "Novo",
    "lastName": "Nome"
  }'
```

---

## Slide 18: Conceitos Aprendidos

✅ **Prisma ORM**: Migrations e type-safe database access
✅ **REST API**: Endpoints seguindo princípios REST
✅ **Autenticação JWT**: Tokens seguros para autenticação
✅ **Hash de senhas**: Segurança com bcrypt
✅ **Validação de dados**: express-validator
✅ **PostgreSQL**: Operações CRUD com Prisma
✅ **Middleware**: Autenticação e logging
✅ **Tratamento de erros**: Respostas consistentes
✅ **Docker**: Containerização do banco
✅ **Refatoração**: Organização de código em módulos

---

## Slide 19: Arquitetura Final

```
📁 projeto/
├── 📄 server.js              # Servidor principal
├── 📄 docker-compose.yml     # PostgreSQL
├── 📄 .env                   # Variáveis de ambiente
├── 📁 prisma/
│   ├── 📄 schema.prisma      # Schema do banco
│   └── � migrations/        # Migrations do banco
├── 📁 middleware/
│   └── 📄 auth.js            # Autenticação JWT
└── 📁 routes/
    ├── 📄 auth.js            # Signup/Signin
    └── 📄 profile.js         # Perfil protegido
```

---

## Slide 20: Comandos Prisma Úteis

```bash
# Gerar Prisma Client
npm run db:generate

# Criar nova migration
npm run db:migrate

# Resetar banco (cuidado!)
npm run db:reset

# Abrir Prisma Studio
npm run db:studio

# Ver status das migrations
npx prisma migrate status

# Deploy migrations (produção)
npx prisma migrate deploy
```

---

## Slide 21: Próximos Passos

🚀 **Melhorias possíveis**:
- ✉️ Verificação de email
- 🔑 Reset de senha
- 🔄 Refresh tokens
- ⚡ Rate limiting
- 📊 Logs estruturados
- 🧪 Testes automatizados
- 🌐 Deploy em produção
- 🔒 HTTPS/SSL
- 📱 API versioning
- 🗃️ Relacionamentos no Prisma

---

## Slide 22: Recursos Adicionais

📚 **Documentação**:
- [Prisma](https://www.prisma.io/docs/)
- [Express.js](https://expressjs.com/)
- [JWT.io](https://jwt.io/)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [bcrypt](https://github.com/kelektiv/node.bcrypt.js)

🛠️ **Ferramentas úteis**:
- [Prisma Studio](https://www.prisma.io/studio) - Interface visual do banco
- [Postman](https://www.postman.com/) - Teste de APIs
- [pgAdmin](https://www.pgadmin.org/) - Interface PostgreSQL
- [Nodemon](https://nodemon.io/) - Auto-reload
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

---

## Slide 23: Perguntas e Respostas

💬 **Dúvidas?**

📧 **Contato**: [seu-email]
🔗 **GitHub**: [seu-github]
📱 **LinkedIn**: [seu-linkedin]

**Obrigado pela participação!** 🎉

### 🎯 Resumo do que construímos:
- ✅ API REST completa com 4 endpoints
- ✅ Prisma ORM com migrations
- ✅ Autenticação JWT segura
- ✅ PostgreSQL com Docker
- ✅ Validação de dados
- ✅ Tratamento de erros
- ✅ Código organizado e modular
- ✅ Desenvolvimento iterativo (endpoints → autenticação → refatoração)

---

## Slide 22: Perguntas e Respostas

💬 **Dúvidas?**

📧 **Contato**: [seu-email]
🔗 **GitHub**: [seu-github]
📱 **LinkedIn**: [seu-linkedin]

**Obrigado pela participação!** 🎉

### 🎯 Resumo do que construímos:
- ✅ API REST completa com 4 endpoints
- ✅ Prisma ORM com migrations
- ✅ Autenticação JWT segura
- ✅ PostgreSQL com Docker
- ✅ Validação de dados
- ✅ Tratamento de erros
- ✅ Código organizado e modular
- ✅ Desenvolvimento iterativo (monolito → módulos)