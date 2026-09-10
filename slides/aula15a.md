---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 15: Testes
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

**Aula 15**: Testes

---
# Por que Testar?
- Detectar regressões: mudanças que quebram funcionalidades existentes;
- Confiança para refatorar código;
- Documentação viva do comportamento esperado;
- Redução de bugs em produção;
- Desenvolvimento mais rápido a longo prazo.

---
# Tipos de Teste
- **Unitário:** testa uma unidade isolada (função, método, classe);
- **Integração:** testa a interação entre componentes (view + model);
- **Funcional / E2E:** simula o comportamento do usuário no sistema;
- Na pirâmide de testes: muitos unitários, poucos E2E.

---
# `unittest` vs `pytest`
| | `unittest` | `pytest` |
|---|---|---|
| Padrão Python | Sim | Terceiro |
| Verbosidade | Alta | Baixa |
| Fixtures | `setUp`/`tearDown` | `@pytest.fixture` |
| Django suporte | Nativo | `pytest-django` |

- Django usa `unittest.TestCase` internamente;
- `pytest` pode executar testes do Django com `pytest-django`.

---
# Django `TestCase`
```python
from django.test import TestCase

class MeuTesteCase(TestCase):

    def setUp(self):
        """Executado antes de cada teste"""
        self.usuario = User.objects.create_user(
            username='teste',
            password='senha123'
        )

    def tearDown(self):
        """Executado após cada teste"""
        pass  # Django já faz rollback do banco
```

---
# Convenções
- Arquivos de teste: `tests.py` ou diretório `tests/`;
- Nomes de classes: `class MinhaClasseTests(TestCase)`;
- Nomes de métodos: `def test_descricao_do_que_testa(self)`;
- Cada teste deve ser independente;
- Banco de dados zerado a cada teste (transação com rollback).

---
# Testando Models — Criação
```python
from django.test import TestCase
from .models import Tarefa

class TarefaModelTests(TestCase):

    def test_criacao_tarefa(self):
        tarefa = Tarefa.objects.create(
            titulo='Estudar Django',
            concluida=False
        )
        self.assertEqual(tarefa.titulo, 'Estudar Django')
        self.assertFalse(tarefa.concluida)
        self.assertEqual(Tarefa.objects.count(), 1)
```

---
# Testando Models — Métodos
```python
class TarefaModelTests(TestCase):

    def test_str_retorna_titulo(self):
        tarefa = Tarefa(titulo='Minha tarefa')
        self.assertEqual(str(tarefa), 'Minha tarefa')

    def test_concluir_tarefa(self):
        tarefa = Tarefa.objects.create(titulo='Fazer algo')
        tarefa.concluir()
        self.assertTrue(tarefa.concluida)

    def test_titulo_nao_pode_ser_vazio(self):
        with self.assertRaises(Exception):
            Tarefa.objects.create(titulo='')
```

---
# Django Test `Client`
- Simula um navegador sem interface gráfica;
- Faz requisições GET/POST para as views;
- Verifica status code, contexto, redirecionamentos;
- Disponível como `self.client` nos `TestCase`.

---
# Testando Views — GET
```python
from django.test import TestCase
from django.urls import reverse

class TarefaViewTests(TestCase):

    def test_lista_tarefas_status_200(self):
        response = self.client.get(reverse('tarefas:lista'))
        self.assertEqual(response.status_code, 200)

    def test_lista_usa_template_correto(self):
        response = self.client.get(reverse('tarefas:lista'))
        self.assertTemplateUsed(response, 'tarefas/lista.html')
```

---
# Testando Views — POST
```python
class TarefaViewTests(TestCase):

    def test_criar_tarefa_post(self):
        dados = {'titulo': 'Nova tarefa', 'concluida': False}
        response = self.client.post(
            reverse('tarefas:criar'),
            dados
        )
        self.assertEqual(response.status_code, 302)  # redirect
        self.assertEqual(Tarefa.objects.count(), 1)
        self.assertEqual(
            Tarefa.objects.first().titulo,
            'Nova tarefa'
        )
```

---
# Testando Redirecionamento
```python
class TarefaViewTests(TestCase):

    def test_redirect_apos_criar(self):
        dados = {'titulo': 'Tarefa teste'}
        response = self.client.post(
            reverse('tarefas:criar'), dados
        )
        # Verifica URL de destino do redirect
        self.assertRedirects(
            response,
            reverse('tarefas:lista')
        )
```

---
# Testando Forms — Dados Válidos
```python
from django.test import TestCase
from .forms import TarefaForm

class TarefaFormTests(TestCase):

    def test_form_valido(self):
        dados = {'titulo': 'Estudar testes', 'concluida': False}
        form = TarefaForm(data=dados)
        self.assertTrue(form.is_valid())

    def test_titulo_obrigatorio(self):
        form = TarefaForm(data={'titulo': ''})
        self.assertFalse(form.is_valid())
        self.assertIn('titulo', form.errors)
```

---
# Testando Forms — Dados Inválidos
```python
class TarefaFormTests(TestCase):

    def test_titulo_muito_longo(self):
        dados = {'titulo': 'x' * 300}  # acima do max_length
        form = TarefaForm(data=dados)
        self.assertFalse(form.is_valid())

    def test_erros_especificos(self):
        form = TarefaForm(data={})
        self.assertFalse(form.is_valid())
        self.assertEqual(len(form.errors), 1)  # só título obrigatório
```

---
# Testando Autenticação — login_required
```python
class AutenticacaoTests(TestCase):

    def test_view_protegida_redireciona_anonimo(self):
        response = self.client.get(reverse('tarefas:criar'))
        self.assertEqual(response.status_code, 302)
        self.assertIn('/login/', response['Location'])

    def test_view_protegida_permite_logado(self):
        User.objects.create_user('user1', password='senha')
        self.client.login(username='user1', password='senha')
        response = self.client.get(reverse('tarefas:criar'))
        self.assertEqual(response.status_code, 200)
```

---
# Testando Permissões
```python
class PermissaoTests(TestCase):

    def setUp(self):
        self.user = User.objects.create_user(
            username='comum', password='senha'
        )
        self.admin = User.objects.create_user(
            username='admin', password='senha',
            is_staff=True
        )

    def test_usuario_comum_nao_acessa_admin_view(self):
        self.client.login(username='comum', password='senha')
        response = self.client.get(reverse('tarefas:admin'))
        self.assertEqual(response.status_code, 403)
```

---
# Fixtures — Dados de Teste
```python
# Usando setUp para criar dados reutilizáveis
class TarefaTests(TestCase):

    def setUp(self):
        self.user = User.objects.create_user(
            username='teste', password='senha123'
        )
        self.tarefa = Tarefa.objects.create(
            titulo='Tarefa de teste',
            usuario=self.user
        )

    def test_usuario_ve_propria_tarefa(self):
        self.client.login(username='teste', password='senha123')
        response = self.client.get(reverse('tarefas:lista'))
        self.assertContains(response, 'Tarefa de teste')
```

---
# Fixtures — Arquivo JSON
```bash
# Exportar dados existentes
python manage.py dumpdata tarefas --indent 2 > fixtures/tarefas.json
```
```python
# Usar fixture no teste
class TarefaTests(TestCase):
    fixtures = ['tarefas.json']

    def test_carregou_dados(self):
        self.assertEqual(Tarefa.objects.count(), 5)
```

---
# Executando os Testes
```bash
# Todos os testes
python manage.py test

# Testes de uma app específica
python manage.py test tarefas

# Uma classe específica
python manage.py test tarefas.tests.TarefaModelTests

# Um método específico
python manage.py test tarefas.tests.TarefaModelTests.test_criacao_tarefa

# Com verbosidade
python manage.py test -v 2
```

---
# Asserções Comuns
```python
# Igualdade
self.assertEqual(a, b)
self.assertNotEqual(a, b)

# Booleanos
self.assertTrue(expr)
self.assertFalse(expr)

# Nulo
self.assertIsNone(x)
self.assertIsNotNone(x)

# Coleções
self.assertIn(item, lista)
self.assertNotIn(item, lista)
```

---
# Asserções Django
```python
# HTTP
self.assertEqual(response.status_code, 200)
self.assertRedirects(response, url)
self.assertTemplateUsed(response, 'template.html')

# Conteúdo HTML
self.assertContains(response, 'texto')
self.assertNotContains(response, 'texto')

# Formulários
self.assertFormError(response, 'form', 'campo', 'erro')
```

---
# Cobertura de Código (`coverage`)
```bash
# Instalar
pip install coverage

# Executar testes com cobertura
coverage run --source='.' manage.py test

# Relatório no terminal
coverage report

# Relatório HTML detalhado
coverage html
# Abre htmlcov/index.html no navegador
```
- Indica quais linhas de código foram exercitadas pelos testes;
- Meta comum: ≥ 80% de cobertura.

---
# Interpretando o Relatório
```
Name                      Stmts   Miss  Cover
---------------------------------------------
tarefas/models.py            20      2    90%
tarefas/views.py             45     10    78%
tarefas/forms.py             15      0   100%
---------------------------------------------
TOTAL                        80     12    85%
```
- `Stmts`: total de linhas executáveis;
- `Miss`: linhas não cobertas;
- Cobertura alta não garante testes de qualidade!

---
# Boas Práticas
- Teste o comportamento, não a implementação;
- Um teste = uma verificação;
- Nomes descritivos: `test_usuario_sem_permissao_recebe_403`;
- Mantenha os testes rápidos (evite I/O real);
- Não teste código do framework (Django já tem seus testes);
- Execute os testes antes de cada commit.

---
# Referências
- https://docs.djangoproject.com/en/5.1/topics/testing/
- https://docs.djangoproject.com/en/5.1/topics/testing/tools/
- https://coverage.readthedocs.io/
- https://pytest-django.readthedocs.io/

---
# <!--fit--> Dúvidas? 🤔
