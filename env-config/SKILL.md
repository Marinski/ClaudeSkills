---
name: env-config
description: Environment configuration and secrets management skill using UV for Python projects. Handles .env files, environment variables, secrets encryption, multi-environment setups, and secure configuration patterns. Use when setting up project environments, managing API keys, or implementing configuration best practices.
---

# Environment Configuration Skill

## Overview

This skill provides comprehensive guidance for managing environment configurations, secrets, and environment variables in Python projects using UV (the modern Python package and project manager). It covers secure configuration patterns, multi-environment setups, .env file management, and secrets handling with encryption support.

## Purpose

Environment configuration is critical for:
- Separating configuration from code (12-factor app principles)
- Managing secrets securely across development, staging, and production
- Enabling different configurations per environment
- Preventing credential leaks in version control
- Supporting team collaboration with shared configuration patterns

## When to Use This Skill

Use this skill when you need to:
- Set up environment configuration for a new Python project
- Implement secure secrets management
- Configure multi-environment setups (dev/staging/prod)
- Migrate from hardcoded configs to environment variables
- Audit existing configuration for security issues
- Standardize configuration across team projects
- Set up UV-based Python project with proper config management

## Core Principles

### 1. Never Hardcode Secrets
- All API keys, passwords, tokens go in environment variables or encrypted secrets
- Configuration files with secrets must be in .gitignore
- Use templates for sharing structure, not actual secrets

### 2. Separate by Environment
- Different configurations for development, staging, production
- Environment-specific .env files (.env.development, .env.production)
- Clear naming conventions for environment variables

### 3. Fail Securely
- Validate required environment variables on startup
- Provide clear error messages for missing configuration
- Use sensible defaults only for non-sensitive values

### 4. Use UV for Dependency Management
- UV provides fast, reliable Python package management
- Replaces pip, pip-tools, virtualenv, and more
- Ensures reproducible environments across machines

### 5. Document Everything
- Template files show structure without exposing secrets
- README explains required variables and how to set them
- Comments describe purpose and format of variables

## UV Integration

### Why UV?

UV is a modern replacement for pip, pip-tools, virtualenv, and poetry. Benefits include:
- **Speed**: 10-100x faster than pip
- **Reliability**: Deterministic dependency resolution
- **Simplicity**: Single tool for all Python project needs
- **Compatibility**: Works with existing pip/requirements.txt projects

### Installing UV

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or with pip
pip install uv

# Verify installation
uv --version
```

### UV Project Setup

```bash
# Create new project with UV
uv init my-project
cd my-project

# Initialize with pyproject.toml
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Add dependencies
uv add python-dotenv cryptography

# Add dev dependencies
uv add --dev pytest black ruff

# Sync dependencies (like npm install)
uv sync
```

## Environment Configuration Workflow

### Phase 1: Project Initialization

**1. Create UV Project Structure**

```bash
# Initialize UV project
uv init your-project-name
cd your-project-name

# Create virtual environment
uv venv

# Activate environment
source .venv/bin/activate  # macOS/Linux
# .venv\Scripts\activate  # Windows
```

**2. Set Up Configuration Files**

Create these essential files:
- `.env.template` - Template showing required variables (committed)
- `.env` - Actual secrets (in .gitignore)
- `.env.development` - Development-specific config
- `.env.production` - Production-specific config
- `pyproject.toml` - UV project configuration
- `secrets_template.json` - JSON-based secrets template (optional)

**3. Update .gitignore**

Add to `.gitignore`:
```
# Environment files
.env
.env.local
.env.*.local
secrets.json

# UV
.venv/
__pycache__/
*.pyc
.pytest_cache/
.ruff_cache/

# Editor
.vscode/
.idea/
*.swp
```

**4. Install Dependencies**

```bash
# Add core dependencies
uv add python-dotenv  # For .env file loading
uv add cryptography   # For secrets encryption (optional)
uv add pydantic       # For config validation (optional)

# Development dependencies
uv add --dev pytest pytest-env
uv add --dev python-dotenv[cli]  # For CLI tools
```

### Phase 2: Configuration Implementation

**1. Create .env.template**

```bash
# .env.template - Commit this file
# Copy to .env and fill in actual values

# Application Settings
APP_NAME=MyApp
APP_ENV=development
DEBUG=true
LOG_LEVEL=INFO

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
DATABASE_POOL_SIZE=5

# API Keys (Replace with actual keys)
ANTHROPIC_API_KEY=sk-ant-api03-xxx
OPENAI_API_KEY=sk-xxx
OPENROUTER_API_KEY=sk-or-v1-xxx

# External Services
REDIS_URL=redis://localhost:6379/0
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# Security
SECRET_KEY=generate-random-secret-key-here
JWT_SECRET=another-random-secret
ENCRYPTION_KEY=base64-encoded-encryption-key

# Feature Flags
ENABLE_ANALYTICS=false
ENABLE_CACHING=true
RATE_LIMIT_ENABLED=true
```

**2. Create Config Loading Module**

Create `config.py`:
```python
"""
Environment configuration management with UV.
Loads and validates environment variables.
"""
import os
import sys
from pathlib import Path
from typing import Optional
from dotenv import load_dotenv


class ConfigError(Exception):
    """Raised when required configuration is missing."""
    pass


class Config:
    """Application configuration from environment variables."""

    def __init__(self, env: str = None):
        """
        Initialize configuration.

        Args:
            env: Environment name (development, staging, production)
                 If None, uses APP_ENV environment variable
        """
        # Determine environment
        self.env = env or os.getenv('APP_ENV', 'development')

        # Load environment-specific .env file
        self._load_env_file()

        # Validate required variables
        self._validate_required()

    def _load_env_file(self):
        """Load appropriate .env file based on environment."""
        # Try environment-specific file first
        env_file = Path(f'.env.{self.env}')
        if env_file.exists():
            load_dotenv(env_file, override=True)
            print(f"✓ Loaded configuration from {env_file}")

        # Then load .env (can override)
        if Path('.env').exists():
            load_dotenv('.env', override=False)  # Don't override env-specific
            print("✓ Loaded configuration from .env")

    def _validate_required(self):
        """Validate that required environment variables are set."""
        required = self.get_required_vars()
        missing = [var for var in required if not os.getenv(var)]

        if missing:
            raise ConfigError(
                f"Missing required environment variables: {', '.join(missing)}\n"
                f"Please check .env.template for required configuration."
            )

    @staticmethod
    def get_required_vars() -> list[str]:
        """
        Define required environment variables.
        Override this in subclasses for custom requirements.
        """
        return [
            'APP_NAME',
            'APP_ENV',
        ]

    # Application Settings
    @property
    def app_name(self) -> str:
        return os.getenv('APP_NAME', 'MyApp')

    @property
    def app_env(self) -> str:
        return self.env

    @property
    def debug(self) -> bool:
        return os.getenv('DEBUG', 'false').lower() in ('true', '1', 'yes')

    @property
    def log_level(self) -> str:
        return os.getenv('LOG_LEVEL', 'INFO')

    # Database
    @property
    def database_url(self) -> Optional[str]:
        return os.getenv('DATABASE_URL')

    # API Keys
    @property
    def anthropic_api_key(self) -> Optional[str]:
        return os.getenv('ANTHROPIC_API_KEY')

    @property
    def openai_api_key(self) -> Optional[str]:
        return os.getenv('OPENAI_API_KEY')

    # Security
    @property
    def secret_key(self) -> str:
        key = os.getenv('SECRET_KEY')
        if not key and not self.debug:
            raise ConfigError("SECRET_KEY must be set in production")
        return key or 'dev-secret-key-change-in-production'


# Global config instance
config = Config()


# Helper function for getting env vars with defaults
def get_env(key: str, default: str = None, required: bool = False) -> str:
    """
    Get environment variable with optional default and validation.

    Args:
        key: Environment variable name
        default: Default value if not set
        required: If True, raises error when not set

    Returns:
        Environment variable value

    Raises:
        ConfigError: If required=True and variable not set
    """
    value = os.getenv(key, default)
    if required and value is None:
        raise ConfigError(f"Required environment variable '{key}' is not set")
    return value
```

**3. Using Config in Your Application**

```python
# main.py
from config import config

def main():
    print(f"Starting {config.app_name} in {config.app_env} mode")
    print(f"Debug mode: {config.debug}")

    if config.anthropic_api_key:
        print("✓ Anthropic API key loaded")

    # Use config throughout your app
    if config.debug:
        print(f"Database: {config.database_url}")

if __name__ == "__main__":
    main()
```

### Phase 3: Multi-Environment Setup

**1. Create Environment-Specific Files**

`.env.development`:
```bash
APP_ENV=development
DEBUG=true
LOG_LEVEL=DEBUG

# Use local services
DATABASE_URL=postgresql://localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379/0

# Test API keys (use free tier)
ANTHROPIC_API_KEY=sk-ant-api03-dev-key
```

`.env.staging`:
```bash
APP_ENV=staging
DEBUG=false
LOG_LEVEL=INFO

# Staging database
DATABASE_URL=postgresql://staging-host:5432/myapp_staging

# Real API keys (use staging/test accounts)
ANTHROPIC_API_KEY=sk-ant-api03-staging-key
```

`.env.production`:
```bash
APP_ENV=production
DEBUG=false
LOG_LEVEL=WARNING

# Production database (never commit this file!)
DATABASE_URL=postgresql://prod-host:5432/myapp_prod

# Production API keys
ANTHROPIC_API_KEY=sk-ant-api03-prod-key
SECRET_KEY=super-secure-random-key
```

**2. Switch Between Environments**

```bash
# Development (default)
uv run python main.py

# Staging
export APP_ENV=staging
uv run python main.py

# Production
export APP_ENV=production
uv run python main.py
```

### Phase 4: Secrets Management

**1. Using JSON Secrets File (Alternative Pattern)**

`secrets_template.json`:
```json
{
  "anthropic_api_key": "sk-ant-api03-xxx",
  "openai_api_key": "sk-xxx",
  "database_password": "your-password-here",
  "encryption_key": "base64-encoded-key",
  "comment": "Copy to secrets.json and fill in real values. Keep secrets.json private!"
}
```

**2. Loading JSON Secrets**

```python
# secrets_loader.py
import json
import os
from pathlib import Path


def load_secrets(secrets_file: str = 'secrets.json') -> dict:
    """
    Load secrets from JSON file with fallback to environment variables.

    Args:
        secrets_file: Path to secrets JSON file

    Returns:
        Dictionary of secrets
    """
    secrets_path = Path(secrets_file)

    # Try loading from JSON file
    if secrets_path.exists():
        try:
            with open(secrets_path, 'r') as f:
                secrets = json.load(f)
            print(f"✓ Loaded secrets from {secrets_file}")
            return secrets
        except json.JSONDecodeError as e:
            print(f"⚠ Error parsing {secrets_file}: {e}")
            print("  Falling back to environment variables")
    else:
        print(f"ℹ {secrets_file} not found, using environment variables")

    # Fallback to environment variables
    return {
        'anthropic_api_key': os.getenv('ANTHROPIC_API_KEY', ''),
        'openai_api_key': os.getenv('OPENAI_API_KEY', ''),
        'database_password': os.getenv('DATABASE_PASSWORD', ''),
    }


# Usage
secrets = load_secrets()
api_key = secrets.get('anthropic_api_key', os.getenv('ANTHROPIC_API_KEY', ''))
```

**3. Encrypted Secrets (Advanced)**

For highly sensitive environments, use the encryption helper:

```python
# In your app
from scripts.env_helper import encrypt_secrets, decrypt_secrets

# Encrypt secrets file
encrypt_secrets('secrets.json', 'secrets.encrypted', 'your-encryption-password')

# Decrypt at runtime
secrets = decrypt_secrets('secrets.encrypted', 'your-encryption-password')
```

See `scripts/env_helper.py` for encryption utilities.

## Configuration Validation

### Using Pydantic for Type-Safe Config

```python
# config_pydantic.py
from pydantic import BaseSettings, Field, validator


class Settings(BaseSettings):
    """Type-safe application settings."""

    # Application
    app_name: str = Field(default='MyApp', env='APP_NAME')
    app_env: str = Field(default='development', env='APP_ENV')
    debug: bool = Field(default=False, env='DEBUG')

    # Database
    database_url: str = Field(..., env='DATABASE_URL')  # Required
    database_pool_size: int = Field(default=5, env='DATABASE_POOL_SIZE')

    # API Keys
    anthropic_api_key: str = Field(default='', env='ANTHROPIC_API_KEY')

    @validator('app_env')
    def validate_env(cls, v):
        allowed = ['development', 'staging', 'production']
        if v not in allowed:
            raise ValueError(f'app_env must be one of {allowed}')
        return v

    @validator('database_pool_size')
    def validate_pool_size(cls, v):
        if v < 1 or v > 100:
            raise ValueError('database_pool_size must be between 1 and 100')
        return v

    class Config:
        env_file = '.env'
        env_file_encoding = 'utf-8'
        case_sensitive = False


# Usage
settings = Settings()
print(settings.app_name)
print(settings.database_url)
```

## Security Best Practices

### 1. .gitignore Configuration

Always add to `.gitignore`:
```
# Secrets and environment files
.env
.env.local
.env.*.local
secrets.json
secrets.encrypted

# Never commit production configs
.env.production

# UV and Python
.venv/
__pycache__/
*.pyc
```

### 2. Secret Rotation

```python
# Implement secret rotation
def rotate_api_key(old_key: str, new_key: str):
    """
    Rotate API key gracefully.

    1. Add new key to environment
    2. Update all services to use new key
    3. Verify new key works
    4. Remove old key
    """
    # Load current config
    env_file = Path('.env')
    content = env_file.read_text()

    # Replace old key with new
    updated = content.replace(old_key, new_key)

    # Backup old config
    backup = Path('.env.backup')
    backup.write_text(content)

    # Write new config
    env_file.write_text(updated)

    print("✓ API key rotated. Backup saved to .env.backup")
```

### 3. Environment Variable Auditing

```python
# Check for exposed secrets
def audit_environment():
    """Audit environment variables for security issues."""
    issues = []

    # Check for default/example values
    dangerous_patterns = [
        'xxx',
        'example',
        'test123',
        'password',
        'changeme',
    ]

    for key, value in os.environ.items():
        if any(pattern in value.lower() for pattern in dangerous_patterns):
            issues.append(f"⚠ {key} appears to have a default/test value")

    # Check for required keys in production
    if os.getenv('APP_ENV') == 'production':
        required = ['SECRET_KEY', 'DATABASE_URL']
        for key in required:
            if not os.getenv(key):
                issues.append(f"❌ Required key {key} not set in production")

    if issues:
        print("Security Issues Found:")
        for issue in issues:
            print(f"  {issue}")
    else:
        print("✓ No security issues detected")
```

## Testing Configuration

### pytest with Environment Variables

`conftest.py`:
```python
import pytest
import os


@pytest.fixture
def test_env():
    """Set up test environment variables."""
    original = os.environ.copy()

    # Set test values
    os.environ['APP_ENV'] = 'testing'
    os.environ['DEBUG'] = 'true'
    os.environ['DATABASE_URL'] = 'sqlite:///:memory:'

    yield

    # Restore original environment
    os.environ.clear()
    os.environ.update(original)


@pytest.fixture
def config(test_env):
    """Provide clean config for each test."""
    from config import Config
    return Config(env='testing')
```

`test_config.py`:
```python
def test_config_loading(config):
    """Test configuration loads correctly."""
    assert config.app_env == 'testing'
    assert config.debug is True


def test_missing_required_var():
    """Test error raised for missing required variables."""
    import os
    from config import Config, ConfigError

    # Remove required var
    old_val = os.environ.pop('APP_NAME', None)

    try:
        with pytest.raises(ConfigError):
            Config()
    finally:
        if old_val:
            os.environ['APP_NAME'] = old_val
```

## UV Project Configuration

### pyproject.toml Example

See `examples/pyproject.toml` for a complete UV project configuration including:
- Project metadata
- Dependencies
- Development dependencies
- Scripts and commands
- Build system configuration

### UV Common Commands

```bash
# Install dependencies
uv sync

# Add new dependency
uv add requests
uv add --dev pytest

# Remove dependency
uv remove requests

# Update dependencies
uv lock --upgrade

# Run script
uv run python main.py
uv run pytest

# Show dependency tree
uv tree

# Create requirements.txt (for compatibility)
uv pip compile pyproject.toml -o requirements.txt
```

## Troubleshooting

### Common Issues

**1. Environment variables not loading**
```python
# Debug: Print loaded variables
import os
from dotenv import load_dotenv

load_dotenv(verbose=True)  # Shows what's being loaded
print(f"APP_NAME: {os.getenv('APP_NAME')}")
```

**2. Wrong environment loaded**
```python
# Explicitly set environment
os.environ['APP_ENV'] = 'development'
from config import config
print(f"Using environment: {config.app_env}")
```

**3. UV sync fails**
```bash
# Clear UV cache
uv cache clean

# Reinstall dependencies
rm -rf .venv
uv venv
uv sync
```

## Helper Scripts

This skill provides utility scripts in `scripts/`:

- `env_helper.py` - Core utilities for env management, validation, encryption
  - Parse and validate .env files
  - Check for missing variables
  - Encrypt/decrypt secrets files
  - Compare environments
  - Generate .env templates

See script documentation for usage examples.

## Additional Resources

**Examples Directory:**
- `.env.example` - Comprehensive .env template
- `pyproject.toml` - UV project configuration
- `secrets_template.json` - JSON secrets template

**UV Documentation:**
- Official docs: https://docs.astral.sh/uv/
- Installation: https://docs.astral.sh/uv/getting-started/installation/
- Project guide: https://docs.astral.sh/uv/guides/projects/

**Python Packages:**
- `python-dotenv`: https://github.com/theskumar/python-dotenv
- `pydantic`: https://docs.pydantic.dev/
- `cryptography`: https://cryptography.io/

## Quick Reference

### Setup Checklist

- [ ] Install UV: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- [ ] Create project: `uv init project-name`
- [ ] Create virtual env: `uv venv`
- [ ] Add dependencies: `uv add python-dotenv`
- [ ] Create `.env.template` from examples
- [ ] Copy to `.env` and fill in secrets
- [ ] Add `.env` to `.gitignore`
- [ ] Create `config.py` module
- [ ] Test configuration loading
- [ ] Set up environment-specific configs
- [ ] Implement validation
- [ ] Add tests for configuration

### Environment Variable Naming

Use consistent naming:
- `APP_*` - Application settings
- `DATABASE_*` - Database configuration
- `REDIS_*` - Redis configuration
- `*_API_KEY` - API keys and tokens
- `*_SECRET` - Secret keys
- `ENABLE_*` - Feature flags
- `*_URL` - Service endpoints

### Security Checklist

- [ ] No secrets in version control
- [ ] `.env` in `.gitignore`
- [ ] Production secrets separate from dev
- [ ] Required variables validated on startup
- [ ] Secrets encrypted at rest (if needed)
- [ ] Regular secret rotation
- [ ] Audit logs for config changes
- [ ] Team training on secure practices
