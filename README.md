
### Documentação

Para rodar a documentação localmente:

```bash
poetry install # instala dependências
mkdocs serve -a localhost:9000  # cria documentacao em: http://localhost:9000/
```

Atualize os docs adicionando ou editando os arquivos `.md` em `docs/`.

Para adicionar seu arquivo no sumário da documentação, adicione-o em
`mkdocs.yml` sob a chave `nav`:

```yaml
nav:
  - Home: index.md
  - Tutoriais:
    - Como acessar dados no datalake: tutoriais/como-acessar-dados.md
  - Contato: contato.md
    - [Seu novo título]: <seu_arquivo>.md
```
