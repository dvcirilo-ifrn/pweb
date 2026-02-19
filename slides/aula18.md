---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 18: Segurança
author: Diego Cirilo

---
<style>
img {
  display: block;
  margin: 0 auto;
}
</style>

# <!-- fit --> Programação de Aplicação Web

### Prof. Diego Cirilo

**Aula 18**: Segurança

---
# Por que Segurança Importa?
- Dados de usuários são responsabilidade do desenvolvedor;
- Falhas de segurança geram prejuízo financeiro e reputacional;
- Legislações como a LGPD (Brasil) e GDPR (Europa) impõem sanções;
- Ataques aumentam conforme a aplicação cresce;
- Segurança deve ser pensada desde o início, não como ajuste final.

---
# OWASP Top 10
A Open Web Application Security Project lista as 10 vulnerabilidades mais críticas:
1. Broken Access Control
2. Cryptographic Failures
3. **Injection** (SQL, XSS, etc.)
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable Components
7. Authentication Failures
8. Data Integrity Failures
9. Security Logging Failures
10. SSRF

---
# SQL Injection
- Inserção de código SQL malicioso em campos de entrada;
- Pode ler, modificar ou deletar dados do banco;
- Exemplo clássico:

```sql
-- Input: ' OR '1'='1
SELECT * FROM users WHERE username='' OR '1'='1';
-- Retorna TODOS os usuários!
```

---
# SQL Injection — Django ORM
- O ORM do Django **protege automaticamente** via *parameterized queries*;
- Nunca interpola strings diretamente no SQL.

```python
# SEGURO — Django escapa automaticamente
Usuario.objects.filter(username=input_do_usuario)

# PERIGOSO — nunca faça isso!
Usuario.objects.extra(
    where=[f"username = '{input_do_usuario}'"]
)

# Se precisar SQL raw, use parâmetros:
from django.db import connection
cursor.execute(
    "SELECT * FROM auth_user WHERE username = %s",
    [input_do_usuario]  # ← lista de parâmetros, não f-string
)
```

---
# XSS — Cross-Site Scripting
- Injeção de scripts maliciosos na página web;
- O script executa no navegador das vítimas;
- Pode roubar cookies de sessão, redirecionar usuários.

```html
<!-- Input do usuário: <script>alert('XSS')</script> -->

<!-- Sem proteção: executa o script -->
<p>{{ comentario }}</p>

<!-- Django escapa por padrão: exibe o texto literal -->
<p>&lt;script&gt;alert(&#x27;XSS&#x27;)&lt;/script&gt;</p>
```

---
# XSS — Proteção no Django
- Templates Django **escapam automaticamente** todas as variáveis;
- Nunca desabilite o escape sem necessidade.

```django
{# Escapado automaticamente — SEGURO #}
{{ usuario.comentario }}

{# Desativa escape — use apenas com HTML confiável! #}
{{ html_de_confianca|safe }}
```

```python
# No Python, use mark_safe() apenas para HTML gerado pelo servidor
from django.utils.safestring import mark_safe

# PERIGOSO: nunca faça com input do usuário
html = mark_safe(f"<b>{request.POST['nome']}</b>")

# SEGURO: use apenas com conteúdo gerado internamente
html = mark_safe("<b>Texto fixo do servidor</b>")
```

---
# CSRF — Cross-Site Request Forgery
- Força o navegador do usuário a enviar requisições não autorizadas;
- Ex.: link em outro site que faz POST para sua aplicação;
- O navegador envia os cookies de sessão automaticamente!

```
1. Usuário está logado em banco.com
2. Abre email malicioso com link para evil.com
3. evil.com tem formulário oculto que faz POST para banco.com/transferir
4. O navegador envia os cookies de sessão → transferência ocorre!
```

---
# CSRF — Proteção no Django
- Django usa um token único por sessão;
- O servidor valida o token em todo POST/PUT/DELETE;
- Já está habilitado por padrão via `CsrfViewMiddleware`.

```django
{# Obrigatório em todo formulário POST! #}
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Enviar</button>
</form>
```

```python
# AJAX: envie o token no header
headers: {'X-CSRFToken': getCookie('csrftoken')}
```

---
# Logs no Django
- Registro de eventos do sistema: erros, acessos, tentativas de login;
- Fundamental para auditoria e diagnóstico de incidentes;
- Django usa o módulo `logging` do Python.

```python
import logging

logger = logging.getLogger(__name__)

def minha_view(request):
    logger.debug('Processando requisição')
    logger.info('Usuário %s acessou a página', request.user)
    logger.warning('Tentativa de acesso não autorizado')
    logger.error('Erro ao processar pagamento: %s', str(e))
    logger.critical('Banco de dados indisponível!')
```

---
# Configurando Logging — `settings.py`
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'arquivo': {
            'class': 'logging.FileHandler',
            'filename': 'django.log',
        },
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'root': {
        'handlers': ['arquivo', 'console'],
        'level': 'WARNING',
    },
    'loggers': {
        'django': {'level': 'ERROR'},
        'minha_app': {'level': 'DEBUG'},
    },
}
```

---
# Captcha
- Diferencia humanos de bots automatizados;
- Protege formulários de cadastro, login e contato;
- Opções: `django-simple-captcha`, hCaptcha, Google reCAPTCHA.

```bash
pip install django-simple-captcha
```
```python
# settings.py
INSTALLED_APPS = [..., 'captcha']

# forms.py
from captcha.fields import CaptchaField

class ContatoForm(forms.Form):
    nome = forms.CharField()
    mensagem = forms.CharField(widget=forms.Textarea)
    captcha = CaptchaField()  # campo obrigatório anti-bot
```

---
# Captcha — Rota e Template
```python
# urls.py
urlpatterns = [
    path('captcha/', include('captcha.urls')),
    ...
]
```
```django
{# template: gera imagem + campo de texto #}
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Enviar</button>
</form>
```
```bash
# Migração necessária
python manage.py migrate
```

---
# Compressão de Assets
- Reduz o tamanho dos arquivos CSS/JS enviados ao navegador;
- Melhora performance e dificulta leitura do código-fonte;
- `django-compressor` agrupa e minifica arquivos estáticos.

```bash
pip install django-compressor
```
```python
# settings.py
INSTALLED_APPS = [..., 'compressor']
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    'compressor.finders.CompressorFinder',
]
```

---
# Compressor — Template
```django
{% load compress %}

{% compress css %}
<link href="{% static 'css/base.css' %}" rel="stylesheet">
<link href="{% static 'css/forms.css' %}" rel="stylesheet">
{% endcompress %}

{% compress js %}
<script src="{% static 'js/app.js' %}"></script>
<script src="{% static 'js/utils.js' %}"></script>
{% endcompress %}
```
- Em produção, gera um único arquivo minificado para cada bloco.

---
# `DEBUG=False` — Configuração de Produção
```python
# settings.py em produção
DEBUG = False  # NUNCA True em produção!

# Lista de domínios permitidos
ALLOWED_HOSTS = ['meusite.com', 'www.meusite.com']

# Arquivos estáticos (servidor web serve diretamente)
STATIC_ROOT = BASE_DIR / 'staticfiles'
python manage.py collectstatic
```
- Com `DEBUG=True` em produção: stacktraces visíveis para atacantes;
- Com `DEBUG=True`: Django serve estáticos mas **não** é adequado para produção.

---
# `SECRET_KEY` e Variáveis de Ambiente
- `SECRET_KEY` é usada para assinar cookies de sessão e tokens CSRF;
- Nunca versione a `SECRET_KEY` no git!
- Use variáveis de ambiente ou arquivos `.env`.

```python
# .env (nunca commitar!)
SECRET_KEY=sua-chave-super-secreta-aqui
DATABASE_URL=postgres://user:pass@host/db

# settings.py
import os
SECRET_KEY = os.environ.get('SECRET_KEY')

# Ou com python-decouple
from decouple import config
SECRET_KEY = config('SECRET_KEY')
```

---
# `.gitignore` e Segredos
```gitignore
# .gitignore — impede commit de arquivos sensíveis
.env
*.env
local_settings.py
db.sqlite3
```
```bash
# Verificar se um segredo foi commitado
git log --all --full-history -- .env

# Se commitado por acidente: revogar e gerar nova chave!
# Histórico do git é público — o segredo está comprometido.
```

---
# Checklist de Segurança Django
```bash
# Django tem um checklist embutido
python manage.py check --deploy
```
Verifica automaticamente:
- `DEBUG=False`
- `SECRET_KEY` segura
- `ALLOWED_HOSTS` configurado
- `SECURE_SSL_REDIRECT`
- Cookies com `Secure` e `HttpOnly`
- `X-Frame-Options`
- `Content-Security-Policy`

---
# Cabeçalhos de Segurança
```python
# settings.py
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'  # impede clickjacking

# Apenas com HTTPS:
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000  # força HTTPS por 1 ano
```

---
# Referências
- https://owasp.org/www-project-top-ten/
- https://docs.djangoproject.com/en/5.1/topics/security/
- https://docs.djangoproject.com/en/5.1/howto/deployment/checklist/
- https://django-simple-captcha.readthedocs.io/
- https://django-compressor.readthedocs.io/
- https://docs.python.org/3/howto/logging.html

---
# <!--fit--> Dúvidas? 🤔
