---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 15: RichText/Formsets
author: Diego Cirilo

---
<style>
img {
  display: block;
  margin: 0 auto;
}
</style>

# <!-- fit --> Programação de Sistemas para Internet

### Prof. Diego Cirilo

**Aula 15**: Formsets

---
# Formsets
- Conjuntos (sets) de forms;
- Permite exibir e salvar mais de uma cópia de um form em uma mesma página;
- Ex. adicionar várias tarefas de uma só vez;
- Faz muito sentido quando utilizado com JS/AJAX.

---
# Exemplo
- Considerando um model Tarefa e um ModelForm TarefaForm (feitos normalmente)
- No forms.py
```python
from django import forms
from django.forms import formset_factory
from .models import Tarefa

class TarefaForm(forms.ModelForm):
    class Meta:
        model = Tarefa
        fields = "__all__"


TarefaFormSet = formset_factory(TarefaForm, extra=2)  # 2 forms por padrão
```

---
# Exemplo
- No views.py:
```python
from django.shortcuts import render, redirect
from .forms import TarefaFormSet

def criar_tarefas(request):
    if request.method == 'POST':
        formset = TarefaFormSet(request.POST)
        if formset.is_valid():
            for form in formset:       # Fazemos o for pois são vários forms
                if form.cleaned_data:  
                    task = form.save()  
            return redirect('success_url')  
    else:
        formset = TarefaFormSet()  # Cria o formset vazio

    context = {
        'formset': formset
    }
    return render(request, 'criar_tarefas.html', context)
```

---
# Exemplo
- No template:
```django
...
<form method="post">
    {% csrf_token %}
    {{ formset.management_form }}  <!-- Necessário para formsets -->
    
    <!-- Carrega os forms um por um -->
    {% for form in formset %}
        <div class="form">
            {{ form }}
        </div>
        <hr>
    {% endfor %}
    
    <button type="submit">Save</button>
</form>
...
```

---
# Editar Formsets
- Caso seja necessário carregar dados no formulário, como em uma view de editar:
- No views.py
```python
from .models import Tarefa
from .forms import TarefaFormSet

def editar_tarefas(request):
    if request.method == 'POST':
        formset = TaskFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data:
                    form.save()
            return redirect('success_url')
    else:
        tarefas = Tarefas.objects.all()
        initial_data = [{'title': task.title, 'description': task.description, 'completed': task.completed} for task in tasks]
        lista_tarefas = []
        for tarefa in tarefas:
            lista_tarefas.append(tarefa)
        formset = TarefasFormSet(initial=lista_tarefas)

    context = {
        'formset': formset,
    }

    return render(request, 'editar_tarefas.html', context)
```

---
# Management Form
- O `management_form` é essencial para o funcionamento do formset;
- Contém campos ocultos que controlam o formset:
    - `TOTAL_FORMS` - número total de forms
    - `INITIAL_FORMS` - forms com dados iniciais
    - `MIN_NUM_FORMS` - mínimo de forms
    - `MAX_NUM_FORMS` - máximo de forms
- Sem ele, o Django não consegue processar o formset!

---
# Parâmetros do formset_factory
```python
TarefaFormSet = formset_factory(
    TarefaForm,
    extra=2,           # forms extras vazios
    max_num=10,        # máximo de forms
    min_num=1,         # mínimo de forms
    validate_min=True, # valida mínimo
    validate_max=True, # valida máximo
    can_delete=True,   # permite marcar para deletar
    can_order=True,    # permite ordenar
)
```

---
# ModelFormSet
- Quando trabalhamos diretamente com Models;
- Usa `modelformset_factory` ao invés de `formset_factory`;
- Simplifica o código pois já faz o CRUD automaticamente.
```python
from django.forms import modelformset_factory
from .models import Tarefa

TarefaFormSet = modelformset_factory(
    Tarefa,
    fields=['titulo', 'descricao', 'concluida'],
    extra=2
)
```

---
# ModelFormSet na View
```python
def gerenciar_tarefas(request):
    if request.method == 'POST':
        formset = TarefaFormSet(request.POST)
        if formset.is_valid():
            formset.save()  # salva todos de uma vez!
            return redirect('lista_tarefas')
    else:
        formset = TarefaFormSet(queryset=Tarefa.objects.all())

    return render(request, 'gerenciar_tarefas.html', {'formset': formset})
```
- O `queryset` define quais objetos serão carregados para edição.

---
# Inline FormSets
- Para relações entre models (ForeignKey);
- Ex: Pedido com vários Itens;
- Usa `inlineformset_factory`;
```python
from django.forms import inlineformset_factory
from .models import Pedido, ItemPedido

ItemFormSet = inlineformset_factory(
    Pedido,           # model pai
    ItemPedido,       # model filho
    fields=['produto', 'quantidade', 'preco'],
    extra=3,
    can_delete=True
)
```

---
# Inline FormSets - Models
```python
# models.py
class Pedido(models.Model):
    cliente = models.CharField(max_length=100)
    data = models.DateField(auto_now_add=True)

class ItemPedido(models.Model):
    pedido = models.ForeignKey(Pedido, on_delete=models.CASCADE)
    produto = models.CharField(max_length=100)
    quantidade = models.IntegerField()
    preco = models.DecimalField(max_digits=10, decimal_places=2)
```

---
<style scoped>pre { font-size: 16px; }</style>
# Inline FormSets - View
```python
def criar_pedido(request):
    if request.method == 'POST':
        form = PedidoForm(request.POST)
        formset = ItemFormSet(request.POST)
        if form.is_valid() and formset.is_valid():
            pedido = form.save()
            formset.instance = pedido  # associa os itens ao pedido
            formset.save()
            return redirect('lista_pedidos')
    else:
        form = PedidoForm()
        formset = ItemFormSet()

    return render(request, 'criar_pedido.html', {
        'form': form,
        'formset': formset
    })
```

---
# Inline FormSets - Template
```django
<form method="post">
    {% csrf_token %}

    <h2>Dados do Pedido</h2>
    {{ form.as_p }}

    <h2>Itens do Pedido</h2>
    {{ formset.management_form }}
    {% for item_form in formset %}
        <div class="item">
            {{ item_form.as_p }}
        </div>
    {% endfor %}

    <button type="submit">Salvar Pedido</button>
</form>
```

---
<style scoped>pre { font-size: 16px; }</style>
# Editando com Inline FormSets
```python
def editar_pedido(request, pk):
    pedido = get_object_or_404(Pedido, pk=pk)

    if request.method == 'POST':
        form = PedidoForm(request.POST, instance=pedido)
        formset = ItemFormSet(request.POST, instance=pedido)
        if form.is_valid() and formset.is_valid():
            form.save()
            formset.save()
            return redirect('lista_pedidos')
    else:
        form = PedidoForm(instance=pedido)
        formset = ItemFormSet(instance=pedido)  # carrega itens existentes

    return render(request, 'editar_pedido.html', {
        'form': form,
        'formset': formset
    })
```

---
# Deletando Itens
- Com `can_delete=True`, cada form tem um checkbox DELETE;
- Ao salvar, os marcados são removidos automaticamente;
```django
{% for item_form in formset %}
    <div class="item">
        {{ item_form.as_p }}
        {% if item_form.instance.pk %}
            {{ item_form.DELETE }} Remover
        {% endif %}
    </div>
{% endfor %}
```

---
# Validação de Formsets
- Podemos criar validações personalizadas;
- Sobrescrevemos o método `clean` do BaseFormSet:
```python
from django.forms import BaseFormSet

class BaseItemFormSet(BaseFormSet):
    def clean(self):
        super().clean()
        if any(self.errors):
            return

        # verifica se há pelo menos um item
        forms_preenchidos = [f for f in self.forms if f.cleaned_data]
        if len(forms_preenchidos) < 1:
            raise forms.ValidationError("Adicione pelo menos um item.")
```

---
# Usando Validação Personalizada
```python
TarefaFormSet = formset_factory(
    TarefaForm,
    formset=BaseItemFormSet,  # usa a classe personalizada
    extra=2
)
```

---
# Formsets com Crispy Forms
- Podemos usar o Crispy Forms com formsets;
```django
{% load crispy_forms_tags %}

<form method="post">
    {% csrf_token %}
    {{ formset.management_form }}

    {% for form in formset %}
        <div class="card mb-3">
            <div class="card-body">
                {{ form|crispy }}
            </div>
        </div>
    {% endfor %}

    <button type="submit" class="btn btn-primary">Salvar</button>
</form>
```

---
# Adicionando Forms com JavaScript
- Podemos adicionar novos forms dinamicamente;
- Precisamos atualizar o `TOTAL_FORMS` do management form;
- O Django espera que os campos sigam o padrão `form-N-campo`.

---
<style scoped>pre { font-size: 14px; }</style>
# Exemplo - Template
```django
<form method="post" id="formset">
    {% csrf_token %}
    {{ formset.management_form }}

    <div id="forms-container">
        {% for form in formset %}
            <div class="form-item">
                {{ form.as_p }}
            </div>
        {% endfor %}
    </div>

    <button type="button" id="add-form">Adicionar</button>
    <button type="submit">Salvar</button>
</form>

<!-- Template vazio para clonar -->
<div id="empty-form" style="display:none;">
    {{ formset.empty_form.as_p }}
</div>
```

---
<style scoped>pre { font-size: 15px; }</style>
# Exemplo - JavaScript
```javascript
document.querySelector("#add-form").addEventListener("click", function() {
    const container = document.querySelector("#forms-container");
    const totalForms = document.querySelector("#id_form-TOTAL_FORMS");
    const formNum = parseInt(totalForms.value);

    // Clona o template vazio
    const emptyForm = document.querySelector("#empty-form").innerHTML;

    // Substitui __prefix__ pelo número do form
    const newForm = emptyForm.replace(/__prefix__/g, formNum);

    // Adiciona o novo form
    container.insertAdjacentHTML("beforeend",
        `<div class="form-item">${newForm}</div>`
    );

    // Atualiza o contador
    totalForms.value = formNum + 1;
});
```

---
# Removendo Forms com JavaScript
```javascript
function removerForm(button) {
    const formItem = button.closest(".form-item");
    const deleteInput = formItem.querySelector("input[name$='-DELETE']");

    if (deleteInput) {
        // Se já existe no banco, marca para deletar
        deleteInput.checked = true;
        formItem.style.display = "none";
    } else {
        // Se é novo, apenas remove do DOM
        formItem.remove();
        // Atualiza TOTAL_FORMS
        const totalForms = document.querySelector("#id_form-TOTAL_FORMS");
        totalForms.value = parseInt(totalForms.value) - 1;
    }
}
```

---
# Quando usar FormSets?
- Cadastro em lote (várias tarefas, produtos, etc.);
- Relações um-para-muitos (pedido com itens);
- Formulários dinâmicos (adicionar/remover campos);
- Edição em massa de registros.

---
# Rich Text no Django
- Campos de texto com formatação (negrito, itálico, listas, etc.);
- Útil para blogs, descrições de produtos, conteúdo editorial;
- O Django não tem suporte nativo a Rich Text;
- Precisamos usar bibliotecas de terceiros.

---
# O que é um Editor Rich Text?
- Editor WYSIWYG (*What You See Is What You Get*);
- Interface visual para formatar texto;
- Gera HTML que é salvo no banco de dados;
- Exemplos: TinyMCE, CKEditor, Quill, Summernote.

---
# django-tinymce
- Integra o editor TinyMCE ao Django;
- Fácil de configurar;
- Funciona bem com o Django Admin;
- Instalação:
```bash
pip install django-tinymce
```

---
# Configuração - settings.py
```python
INSTALLED_APPS = [
    ...
    'tinymce',
    ...
]

TINYMCE_DEFAULT_CONFIG = {
    'height': 360,
    'width': '100%',
    'menubar': False,
    'plugins': 'lists link image code table',
    'toolbar': 'undo redo | formatselect | bold italic | '
               'alignleft aligncenter alignright | '
               'bullist numlist | link image | code',
}
```

---
# Configuração - urls.py
```python
from django.urls import path, include

urlpatterns = [
    ...
    path('tinymce/', include('tinymce.urls')),
    ...
]
```

---
# Usando no Model
```python
from django.db import models
from tinymce.models import HTMLField

class Artigo(models.Model):
    titulo = models.CharField(max_length=200)
    # Campo Rich Text
    conteudo = HTMLField()
    data_publicacao = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.titulo
```
- `HTMLField` substitui o `TextField` comum.

---
# Usando no Form
- Se preferir usar em um Form ao invés do Model:
```python
from django import forms
from tinymce.widgets import TinyMCE

class ArtigoForm(forms.ModelForm):
    conteudo = forms.CharField(widget=TinyMCE())

    class Meta:
        model = Artigo
        fields = ['titulo', 'conteudo']
```

---
# No Django Admin
- O `HTMLField` já funciona automaticamente no Admin;
- Não precisa de configuração extra;
- O editor aparece no lugar do textarea padrão.

---
# No Template - Formulário
```django
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Salvar</button>
</form>

<!-- Necessário para carregar o TinyMCE -->
{{ form.media }}
```
- O `{{ form.media }}` carrega os scripts necessários.

---
# Exibindo o Conteúdo
- O conteúdo salvo é HTML;
- Use o filtro `safe` para renderizar:
```django
<article>
    <h1>{{ artigo.titulo }}</h1>
    <div class="conteudo">
        {{ artigo.conteudo|safe }}
    </div>
</article>
```
- **Cuidado**: só use `safe` com conteúdo confiável (de admins/editores).

---
# Segurança - XSS
- Rich Text pode ser vetor de ataques XSS;
- *Cross Site Scripting*;
- O usuário pode inserir scripts maliciosos;
- Soluções:
    - Limitar quem pode usar o editor (apenas admins);
    - Usar biblioteca de sanitização como `bleach`;
    - Configurar o TinyMCE para limitar tags permitidas.

---
# Sanitizando com Bleach
```bash
pip install bleach
```
```python
import bleach

ALLOWED_TAGS = ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a', 'img']
ALLOWED_ATTRS = {'a': ['href'], 'img': ['src', 'alt']}

class Artigo(models.Model):
    conteudo = HTMLField()

    def save(self, *args, **kwargs):
        self.conteudo = bleach.clean(
            self.conteudo,
            tags=ALLOWED_TAGS,
            attributes=ALLOWED_ATTRS
        )
        super().save(*args, **kwargs)
```

---
# Configurações Úteis do TinyMCE
```python
TINYMCE_DEFAULT_CONFIG = {
    'height': 400,
    'plugins': 'lists link image table code wordcount',
    'toolbar': 'undo redo | styles | bold italic | '
               'alignleft aligncenter alignright | '
               'bullist numlist outdent indent | link image table | code',
    'content_css': '/static/css/editor.css',  # CSS personalizado
    'valid_elements': 'p,br,strong,em,ul,ol,li,a[href],img[src|alt]',
    'language': 'pt_BR',
}
```

---
# Upload de Imagens
- Por padrão, TinyMCE não faz upload de imagens;
- Opções:
    - Usar URLs externas;
    - Configurar endpoint de upload próprio;
    - Usar `django-filebrowser` ou similar;
- Configuração básica (apenas URLs):
```python
TINYMCE_DEFAULT_CONFIG = {
    ...
    'image_advtab': True,
    'image_caption': True,
}
```

---
# Referências
- https://docs.djangoproject.com/en/5.1/topics/forms/formsets/
- https://docs.djangoproject.com/en/5.1/topics/forms/modelforms/#model-formsets
- https://docs.djangoproject.com/en/5.1/topics/forms/modelforms/#inline-formsets
- https://django-tinymce.readthedocs.io/
- https://github.com/summernote/django-summernote

---
# <!--fit--> Dúvidas? 🤔
