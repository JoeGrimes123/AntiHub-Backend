# Shared Account Management System

A FastAPI-based shared account management system with integrated Plug-in API functionality, supporting traditional username/password login and OAuth SSO single sign-on, providing complete AI chat services and quota management.

## Features

### ✅ Implemented Features

- **User Authentication**
  - Traditional username/password login
  - OAuth 2.0 SSO single sign-on (supports Linux.do, GitHub)
  - JWT token authentication
  - **Seamless refresh mechanism** (Refresh Token)
  - Session management
  - Token blacklist mechanism
  - Token rotation security mechanism

- **User Management**
  - User information storage (PostgreSQL)
  - OAuth token management
  - User status management (active/disabled/silenced)
  - Trust level system

- **Plug-in API Integration**
  - Automatic account creation: Automatically creates Plug-in API accounts upon user registration
  - API key secure storage: Uses Fernet encryption algorithm for secure storage
  - Proxy requests: All Plug-in API requests are proxied through the backend
  - OpenAI compatible interface: Supports standard OpenAI API format
  - Complete feature support: Account management, quota management, chat completions, etc.

- **AI Chat Service**
  - OpenAI compatible chat completion API
  - Supports streaming and non-streaming output
  - Multi-model support (Gemini, etc.)
  - Intelligent Cookie selection and rotation

- **Quota Management System**
  - User shared quota pool
  - Automatic quota recovery mechanism
  - Quota consumption tracking and statistics
  - Dedicated/shared Cookie priority settings

- **Security Features**
  - bcrypt password hashing (rounds=12)
  - JWT Access Token (HS256, default 24-hour validity)
  - JWT Refresh Token (default 7-day validity)
  - OAuth state validation (prevents CSRF attacks)
  - Token auto-rotation mechanism
  - API key encryption storage

- **Cache System**
  - Redis session storage
  - Token blacklist
  - Refresh Token storage and management
  - OAuth state temporary storage

## Technology Stack

- **Web Framework**: FastAPI 0.104+
- **Database**: PostgreSQL + SQLAlchemy 2.0 (async)
- **Cache**: Redis
- **Authentication**: PyJWT + Passlib
- **HTTP Client**: httpx
- **Database Migration**: Alembic
- **Data Validation**: Pydantic

## Quick Start

### 1. Requirements

- Python 3.10+
- PostgreSQL 12+
- Redis 6+
- Plug-in API service (optional, for AI chat functionality)

### 2. Install Dependencies

```bash
# Using uv
uv sync
```

### 3. Configure Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
cp .env.example .env
```

Edit the `.env` file and configure the following required items:

```bash
# Application configuration
APP_ENV=development
LOG_LEVEL=INFO

# Database configuration
DATABASE_URL=postgresql+asyncpg://user:password@localhost:5432/shared_accounts

# Redis configuration
REDIS_URL=redis://localhost:6379/0
# Or with password: redis://:password@localhost:6379/0

# JWT configuration
JWT_SECRET_KEY=your-secret-key-change-this-in-production
JWT_ALGORITHM=HS256
JWT_EXPIRE_HOURS=24

# Refresh Token configuration
REFRESH_TOKEN_EXPIRE_DAYS=7
# REFRESH_TOKEN_SECRET_KEY=your-refresh-token-secret-key  # Optional, defaults to JWT_SECRET_KEY

# OAuth configuration (Linux.do)
OAUTH_CLIENT_ID=your-oauth-client-id
OAUTH_CLIENT_SECRET=your-oauth-client-secret
OAUTH_REDIRECT_URI=http://localhost:8008/api/auth/sso/callback
OAUTH_AUTHORIZATION_ENDPOINT=https://connect.linux.do/oauth2/authorize
OAUTH_TOKEN_ENDPOINT=https://connect.linux.do/oauth2/token
OAUTH_USER_INFO_ENDPOINT=https://connect.linux.do/api/user

# GitHub OAuth configuration
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_REDIRECT_URI=http://localhost:3000/api/auth/github/callback

# Plug-in API configuration (optional)
PLUGIN_API_BASE_URL=http://localhost:8045
PLUGIN_API_ADMIN_KEY=sk-admin-your-admin-key-here
PLUGIN_API_ENCRYPTION_KEY=your-encryption-key-here-min-32-chars
```

**Important**: `PLUGIN_API_ENCRYPTION_KEY` must be a valid Fernet key (32-byte URL-safe base64 encoded). You can generate one using the following Python code:

```python
from cryptography.fernet import Fernet
print(Fernet.generate_key().decode())
```

### 4. Database Migration

```bash
# Run database migration
uv run alembic upgrade head
```

### 5. Start the Service

```bash
# Using the startup script (recommended)
chmod +x run.sh
./run.sh

# Or directly using uvicorn
uv run uvicorn app.main:app --host 0.0.0.0 --port 8008 --reload

# Or using Python
uv run python app/main.py
```

The service will start at http://localhost:8008

## API Documentation

After starting the service, visit:

- **Swagger UI**: http://localhost:8008/api/docs
- **ReDoc**: http://localhost:8008/api/redoc
- **OpenAPI JSON**: http://localhost:8008/api/openapi.json

**Note**: API documentation is disabled in production environment.

## API Endpoints

### Authentication

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/auth/login` | Username/password login |
| POST | `/api/auth/refresh` | Refresh access token (seamless refresh) |
| GET | `/api/auth/sso/initiate` | Initiate OAuth SSO login |
| GET | `/api/auth/sso/callback` | OAuth callback handler |
| GET | `/api/auth/github/login` | Initiate GitHub OAuth login |
| POST | `/api/auth/github/callback` | GitHub OAuth callback handler |
| POST | `/api/auth/logout` | User logout |
| POST | `/api/auth/logout-all` | Logout from all devices |
| GET | `/api/auth/me` | Get current user info |
| GET | `/api/auth/check-username` | Check if username exists |

### Health Check

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | System health status check |

### API Key Management

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/api-keys` | Get user's API key list |
| POST | `/api/api-keys` | Create new API key |
| DELETE | `/api/api-keys/{key_id}` | Delete specified API key |

### Plug-in API Proxy (requires Plug-in API service configuration)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/plugin-api/accounts` | Get account list |
| POST | `/api/plugin-api/oauth/authorize` | Get OAuth authorization URL |
| GET | `/api/plugin-api/quotas/user` | Get user quota information |
| POST | `/api/plugin-api/chat/completions` | Chat completion (OpenAI compatible) |
| GET | `/api/plugin-api/models` | Get available model list |

### OpenAI Compatible Interface

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/models` | Get model list |
| POST | `/v1/chat/completions` | Chat completion (streaming/non-streaming) |

### Usage Statistics

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/usage/stats` | Get usage statistics |
| GET | `/api/usage/logs` | Get usage logs |

## Seamless Refresh Mechanism

The system implements a complete seamless refresh (Silent Refresh) mechanism, allowing users to avoid frequent re-logins.

### How It Works

1. **During Login**: After successful login, the system returns two tokens:
   - `access_token`: Short-term valid (default 24 hours), used for API request authentication
   - `refresh_token`: Long-term valid (default 7 days), used to refresh access_token

2. **API Requests**: Use `access_token` for authentication
   ```
   Authorization: Bearer <access_token>
   ```

3. **Token Refresh**: When `access_token` is about to expire or has expired, use `refresh_token` to get a new token pair
   ```bash
   POST /api/auth/refresh
   {
     "refresh_token": "<your_refresh_token>"
   }
   ```

4. **Token Rotation**: After each refresh, the old `refresh_token` becomes invalid, and a new token pair is returned (security mechanism)

### Frontend Implementation Suggestions

```javascript
// Example: Axios interceptor implementing seamless refresh
axios.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;
    
    // If it's a 401 error and not the refresh request itself
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // Use refresh_token to get new tokens
        const { data } = await axios.post('/api/auth/refresh', {
          refresh_token: localStorage.getItem('refresh_token')
        });
        
        // Save new tokens
        localStorage.setItem('access_token', data.access_token);
        localStorage.setItem('refresh_token', data.refresh_token);
        
        // Retry original request
        originalRequest.headers.Authorization = `Bearer ${data.access_token}`;
        return axios(originalRequest);
      } catch (refreshError) {
        // Refresh failed, need to login again
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);
```

### Login Response Example

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 86400,
  "user": {
    "id": 1,
    "username": "johndoe",
    "avatar_url": "https://example.com/avatar.jpg",
    "trust_level": 1,
    "is_active": true,
    "is_silenced": false
  }
}
```

### Refresh Response Example

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 86400
}
```

## Usage Examples

### Traditional Login

```bash
curl -X POST "http://localhost:8008/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

### Refresh Token

```bash
curl -X POST "http://localhost:8008/api/auth/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "YOUR_REFRESH_TOKEN"
  }'
```

### OAuth SSO Login

```bash
# 1. Get authorization URL
curl "http://localhost:8008/api/auth/sso/initiate"

# 2. User visits the returned authorization_url in browser
# 3. After authorization, will redirect to callback URL and automatically complete login
```

### GitHub OAuth Login

```bash
# 1. Get GitHub authorization URL
curl "http://localhost:8008/api/auth/github/login"

# 2. User visits the returned authorization_url in browser
# 3. After authorization, frontend calls callback endpoint to complete login
curl -X POST "http://localhost:8008/api/auth/github/callback" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "github_authorization_code",
    "state": "oauth_state"
  }'
```

### Get Current User Info

```bash
curl "http://localhost:8008/api/auth/me" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Logout

```bash
# Logout current device
curl -X POST "http://localhost:8008/api/auth/logout" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "YOUR_REFRESH_TOKEN"
  }'

# Logout all devices
curl -X POST "http://localhost:8008/api/auth/logout-all" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Create API Key

```bash
curl -X POST "http://localhost:8008/api/api-keys" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My API Key",
    "description": "Key for testing"
  }'
```

### AI Chat (requires Plug-in API configuration)

```bash
# Streaming chat
curl -X POST "http://localhost:8008/api/plugin-api/chat/completions" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-pro-high",
    "messages": [
      {"role": "user", "content": "Hello, please introduce yourself"}
    ],
    "stream": true
  }'

# OpenAI compatible format
curl -X POST "http://localhost:8008/v1/chat/completions" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-pro-high",
    "messages": [
      {"role": "user", "content": "Hello, please introduce yourself"}
    ],
    "stream": false
  }'
```

### Get Quota Information

```bash
curl "http://localhost:8008/api/plugin-api/quotas/user" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## Project Structure

```
antigv-backend/
├── alembic/                      # Database migration scripts
│   ├── versions/                 # Migration version files
│   ├── env.py                    # Alembic environment configuration
│   └── script.py.mako            # Migration script template
├── app/
│   ├── api/                      # API routes
│   │   ├── deps.py               # Dependency injection
│   │   ├── deps_flexible.py      # Flexible dependency injection
│   │   └── routes/               # Route endpoints
│   │       ├── auth.py           # Authentication routes
│   │       ├── health.py         # Health check
│   │       ├── plugin_api.py     # Plug-in API proxy
│   │       ├── api_keys.py       # API key management
│   │       ├── usage.py          # Usage statistics
│   │       └── v1.py             # OpenAI compatible interface
│   ├── cache/                    # Redis cache
│   │   └── redis_client.py       # Redis client
│   ├── core/                     # Core modules
│   │   ├── config.py             # Configuration management
│   │   ├── security.py           # Security features
│   │   └── exceptions.py         # Exception definitions
│   ├── db/                       # Database
│   │   ├── base.py               # Base classes
│   │   └── session.py            # Session management
│   ├── models/                   # Data models
│   │   ├── user.py               # User model
│   │   ├── api_key.py            # API key model
│   │   ├── oauth_token.py        # OAuth token model
│   │   ├── plugin_api_key.py     # Plug-in API key model
│   │   └── usage_log.py          # Usage log model
│   ├── repositories/             # Data repository layer
│   │   ├── user_repository.py    # User repository
│   │   ├── api_key_repository.py # API key repository
│   │   ├── oauth_token_repository.py # OAuth token repository
│   │   └── plugin_api_key_repository.py # Plug-in API key repository
│   ├── schemas/                  # Pydantic Schemas
│   │   ├── user.py               # User Schema
│   │   ├── auth.py               # Authentication Schema
│   │   ├── api_key.py            # API key Schema
│   │   ├── token.py              # Token Schema
│   │   └── plugin_api.py         # Plug-in API Schema
│   ├── services/                 # Business logic layer
│   │   ├── auth_service.py       # Authentication service
│   │   ├── user_service.py       # User service
│   │   ├── oauth_service.py      # OAuth service
│   │   ├── github_oauth_service.py # GitHub OAuth service
│   │   └── plugin_api_service.py # Plug-in API service
│   ├── utils/                    # Utility modules
│   │   └── encryption.py         # Encryption utilities
│   └── main.py                   # Application entry point
├── .env.example                  # Environment variables example
├── .gitignore                    # Git ignore file
├── .python-version               # Python version
├── alembic.ini                   # Alembic configuration
├── pyproject.toml                # Project configuration and dependencies
├── uv.lock                       # uv lock file
├── run.sh                        # Startup script
├── generate_encryption_key.py    # Encryption key generation tool
├── PLUGIN_API_INTEGRATION.md     # Plug-in API integration documentation
├── plug-in-API.md               # Plug-in API usage documentation
└── README.md                     # Project documentation
```

## Development Guide

### Database Migration

```bash
# Create new migration
uv run alembic revision --autogenerate -m "Description"

# Apply migration
uv run alembic upgrade head

# Rollback migration
uv run alembic downgrade -1

# View migration history
uv run alembic history

# View current version
uv run alembic current
```

### Code Style

The project uses type annotations and docstrings. Please maintain consistent code style:

- Use Python 3.10+ type annotations
- All public functions and classes require docstrings
- Follow PEP 8 code standards
- Use async programming patterns (async/await)

### Environment Configuration

#### Development Environment
```bash
APP_ENV=development
LOG_LEVEL=DEBUG
```

#### Production Environment
```bash
APP_ENV=production
LOG_LEVEL=INFO
# Ensure strong passwords and secure JWT keys
# Recommend using independent keys for Refresh Token
# Configure appropriate CORS origins
# Disable API documentation
```

### Plug-in API Integration Development

To add new Plug-in API proxy endpoints:

1. Add service methods in `app/services/plugin_api_service.py`
2. Add routes in `app/api/routes/plugin_api.py`
3. Add corresponding Schemas in `app/schemas/plugin_api.py`
4. Update API documentation

For detailed integration instructions, refer to [`PLUGIN_API_INTEGRATION.md`](PLUGIN_API_INTEGRATION.md).

### Testing

```bash
# Run tests (if available)
uv run pytest

# Run specific tests
uv run pytest tests/test_auth.py
```

### Deployment

#### Docker Deployment (Recommended)

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY . .

RUN pip install uv && uv sync

EXPOSE 8008

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8008"]
```

#### Traditional Deployment

```bash
# Install dependencies
uv sync

# Set environment variables
export APP_ENV=production

# Run database migration
uv run alembic upgrade head

# Start service
uv run uvicorn app.main:app --host 0.0.0.0 --port 8008 --workers 4
```

## Troubleshooting

### Common Issues

1. **Database Connection Failed**
   - Check `DATABASE_URL` configuration
   - Ensure PostgreSQL service is running
   - Verify database user permissions

2. **Redis Connection Failed**
   - Check `REDIS_URL` configuration
   - Ensure Redis service is running
   - Verify Redis password (if any)

3. **OAuth Login Failed**
   - Check OAuth client ID and secret
   - Verify callback URL configuration
   - Ensure OAuth server is accessible

4. **Plug-in API Functionality Issues**
   - Check `PLUGIN_API_BASE_URL` configuration
   - Verify admin key and encryption key
   - Ensure Plug-in API service is running

5. **Token Refresh Failed**
   - Check if `refresh_token` has expired (default 7 days)
   - Confirm `refresh_token` has not been revoked
   - Verify Redis service is running normally

### Viewing Logs

```bash
# View application logs
tail -f logs/app.log

# View specific level logs
grep "ERROR" logs/app.log
```

## License

MIT License

## Contributing

Welcome to submit Issues and Pull Requests!

### Contribution Guidelines

1. Fork the project
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Issue Reporting

When reporting issues, please include:

- Detailed problem description
- Steps to reproduce
- Environment information (OS, Python version, etc.)
- Relevant error logs

## Related Documentation

- [Plug-in API Integration Documentation](PLUGIN_API_INTEGRATION.md)
- [Plug-in API Usage Documentation](plug-in-API.md)
- [FastAPI Official Documentation](https://fastapi.tiangolo.com/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)