# Criando um projeto Django

Neste tutorial criaremos um repositório com um projeto Django e duas páginas estáticas.

## Configurando o ambiente

Siga o Tutorial 01, garantindo que o `venv` esteja criado e ativado.

Instale o pacote Django com:
```
pip install django
```

Adicione o novo pacote ao `requirements.txt` com:
```
pip freeze > requirements.txt
```

## Iniciando um projeto Django com um App

Inicialize um projeto Django chamado `config` na pasta atual:
```
django-admin startproject config .
```

Esse comando criará a pasta `config` e o arquivo `manage.py`.
O `manage.py` é o *script* que utilizaremos daqui pra frente em **todos** os comandos do Django.

Para criar um app chamado `meuapp`:
```
python manage.py startapp myapp
```

Devemos adicionar os Apps utilizados em `INSTALLED_APPS` no arquivo `config/settings.py`, no caso, `meuapp`:
```
INSTALLED_APPS = [
    # ... as outras coisas que já existiam, com vírgula no final!
    'meuapp',
]
```

Pronto, o projeto com app já está criado e habilitado.

## Configurando as URLs

Na pasta `config` temos o arquivo `urls.py` do projeto. Ele que recebe as requisições do navegador e redireciona para alguma *view*.

Nesse arquivo temos uma lista chamada `urlpatterns` com os endereços que serão digitados no navegador ou acessados através de links.

É comum criar um arquivo de URLs para cada app, e no arquivo `config/urls.py` apenas incluir esses arquivos individuais. Isso facilita o reúso dos apps.

Para o incluir as URLs de `meuapp` em `config/urls.py` devemos adicionar:
```python
from django.urls import path, include # no início do arquivo, basta adicionar o include

urlpatterns = [
    path('', include('meuapp.urls')), # vai incluir as urls de meuapp quando o site principal for acessado
    path('admin/', admin.site.urls),
]
```

Pronto, agora o Django vai procurar `meuapp/urls.py` quando o site for acessado. Porém esse arquivo não existe ainda e deve ser criado.

Dentro de `meuapp` crie um arquivo `urls.py` com o seguinte conteúdo:
```python
from django.urls import path
from . import views # Importa as views desse app

urlpatterns = [
    path('', views.index, name='index'), # Página inicial/index do projeto
]
```

Já temos as URLs configuradas para chamar a view `index` quando a página inicial for acessada. Porém essa view não existe ainda.

## Criando uma view

Vamos trabalhar com *function-based views* (FBV), ou *views* baseadas em função, logo cada *view* será uma função Python no arquivo `meuapp/views.py`.

As *views* sempre recebem o objeto `request`, que é a requisição HTTP enviada pelo navegador/cliente. Ao final, a *view* normalmente retorna uma página HTML renderizada, junto ao objeto `request` novamente.

Para criar sua primeira *view* acesse o arquivo `meuapp/views.py` e adicione o seguinte conteúdo:
```python
def index(request):
    return render(request, "index.html")
```