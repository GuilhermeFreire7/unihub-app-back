# UniHub API

API backend do UniHub, construída com [FastAPI](https://fastapi.tiangolo.com/) e [SQLAlchemy](https://www.sqlalchemy.org/), com PostgreSQL como banco de dados. Fornece autenticação de usuários e gerenciamento de disciplinas, grade curricular e atividades acadêmicas.

## Stack

- **FastAPI** 0.115 — framework web
- **SQLAlchemy** 2.0 — ORM
- **Alembic** 1.16 — migrações de banco de dados
- **PostgreSQL** (via `psycopg2-binary`)
- **python-jose** — geração/validação de JWT
- **passlib[bcrypt]** — hash de senhas
- **Uvicorn** — servidor ASGI

## Requisitos

- Python 3.11+
- PostgreSQL em execução

## Instalação

```bash
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Linux/Mac

pip install fastapi uvicorn sqlalchemy alembic psycopg2-binary python-jose passlib[bcrypt] python-multipart pydantic
```

> O projeto ainda não possui um `requirements.txt`. Recomenda-se gerar um com `pip freeze > requirements.txt` após instalar as dependências.

## Configuração

A conexão com o banco é definida em [app/database.py](app/database.py) e a chave de assinatura JWT em [app/auth.py](app/auth.py). Atualmente ambas estão fixas no código-fonte:

```python
DATABASE_URL = "postgresql://usuario:senha@localhost:5433/unihub"
SECRET_KEY = "sua_chave_supersecreta"
```

**Recomendado:** antes de usar em produção, mova esses valores para variáveis de ambiente (ex. via `python-dotenv`) e nunca faça commit de credenciais reais.

## Banco de dados

O schema é criado automaticamente na inicialização (`Base.metadata.create_all`), mas as migrações são gerenciadas via Alembic:

```bash
# aplicar migrações
alembic upgrade head

# gerar uma nova migração após alterar app/models.py
alembic revision --autogenerate -m "descrição da mudança"
```

## Executando a aplicação

```bash
uvicorn app.main:app --reload
```

A API sobe em `http://localhost:8000`. Documentação interativa disponível em `http://localhost:8000/docs` (Swagger) e `http://localhost:8000/redoc`.

## Estrutura do projeto

```
app/
├── main.py           # criação da app FastAPI, CORS e registro das rotas
├── database.py       # engine, sessão e Base do SQLAlchemy
├── models.py         # modelos ORM (Usuario, Disciplina, Grade, Atividade)
├── schemas.py        # schemas Pydantic de entrada/saída
├── crud.py           # operações de acesso ao banco
├── auth.py           # hashing de senha, JWT e dependência get_current_user
└── routers/
    ├── login.py       # POST /login
    ├── usuarios.py     # /usuarios
    ├── disciplinas.py  # /disciplinas
    ├── grades.py       # /grades
    └── atividades.py   # /atividades

alembic/              # migrações de banco de dados
alembic.ini            # configuração do Alembic
```

## Endpoints

### Autenticação
| Método | Rota      | Descrição                                  |
|--------|-----------|---------------------------------------------|
| POST   | `/login/` | Autentica usuário (email/senha) e retorna JWT |

### Usuários
| Método | Rota                  | Descrição                |
|--------|------------------------|---------------------------|
| POST   | `/usuarios/`           | Cria um novo usuário       |
| GET    | `/usuarios/{usuario_id}` | Obtém um usuário por ID  |

### Disciplinas
| Método | Rota                  | Descrição                        |
|--------|------------------------|-------------------------------------|
| POST   | `/disciplinas/`        | Cria uma disciplina                 |
| GET    | `/disciplinas/`        | Lista todas as disciplinas          |
| POST   | `/disciplinas/batch`   | Cria várias disciplinas em lote (ignora códigos duplicados) |

### Grade curricular
| Método | Rota                     | Descrição                              |
|--------|---------------------------|------------------------------------------|
| GET    | `/grades/usuario/{usuario_id}` | Lista a grade de um usuário          |
| POST   | `/grades/usuario/{usuario_id}` | Adiciona uma disciplina à grade      |
| PUT    | `/grades/{grade_id}`      | Atualiza um item da grade                |
| DELETE | `/grades/{grade_id}`      | Remove um item da grade                  |

### Atividades
| Método | Rota                           | Descrição                        |
|--------|----------------------------------|-------------------------------------|
| POST   | `/atividades/usuario/{usuario_id}` | Cria uma atividade para o usuário |
| GET    | `/atividades/usuario/{usuario_id}` | Lista atividades do usuário       |
| PUT    | `/atividades/{atividade_id}`   | Atualiza uma atividade              |
| DELETE | `/atividades/{atividade_id}`   | Remove uma atividade                |

## Autenticação

O login (`POST /login/`) retorna um token JWT (`access_token`) via fluxo `OAuth2PasswordRequestForm`. Envie esse token no header `Authorization: Bearer <token>` para acessar rotas protegidas.

> Nota: a dependência `get_current_user` (em [app/auth.py](app/auth.py)) já está implementada, mas ainda não é aplicada nos routers existentes — as rotas de usuários, disciplinas, grades e atividades estão atualmente abertas, sem exigir autenticação.
