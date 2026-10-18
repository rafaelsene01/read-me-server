# read-me-server

Lista compartilhada de servidores de livros do marketplace da extensão **ReadMe**.

Ninguém precisa adicionar esses servidores à mão: a extensão lê o arquivo [`servers.json`](servers.json) sozinha sempre que o navegador abre, quando é instalada ou atualizada e quando o marketplace é aberto, visita cada servidor da lista e mostra os livros encontrados no marketplace, com o selo **Lista compartilhada**.

Ela lê sempre deste endereço:

```
https://raw.githubusercontent.com/rafaelsene01/read-me-server/HEAD/servers.json
```

`HEAD` é a branch padrão do repositório (hoje `master`). Por isso o arquivo tem que ficar **na raiz**, com **este nome**, na **branch padrão**. Mudar o nome ou o lugar exige mudar `SERVER_LIST_URL` em `lib/marketplace.ts` na extensão e publicar uma nova versão.

## Formato

Um array JSON de URLs de pastas:

```json
[
  "https://outro-servidor.com/livros/"
]
```

- Cada URL aponta para uma **pasta**, não para um arquivo, e termina com `/`.
- Só `http://` ou `https://` (prefira `https://`).
- Nada além de strings: a extensão ignora qualquer outro valor e, se o arquivo não for um array, não muda nada.
- JSON puro: sem comentários e sem vírgula depois do último item.

## Adicionar um servidor

1. Configure o servidor (veja [Requisitos do servidor](#requisitos-do-servidor)).
2. Verifique que ele responde (veja [Conferir antes de subir](#conferir-antes-de-subir)).
3. Adicione a URL ao `servers.json` e envie para a branch padrão (direto ou por pull request).

## Remover um servidor

Apague a URL do `servers.json` e envie para a branch padrão. Na próxima vez que o navegador abrir ou que o marketplace for aberto, o servidor some do marketplace de todos os usuários.

Na extensão, servidores da lista não têm botão de remover: voltariam no próximo início. Remover é sempre por aqui.

## Requisitos do servidor

A pasta precisa:

1. **Listar os arquivos** (directory listing): abrir a URL no navegador mostra a lista.
2. **Liberar CORS**: responder com `Access-Control-Allow-Origin: *`.

Apache: um `.htaccess` na pasta dos livros.

```apache
Options +Indexes
IndexOptions FancyIndexing HTMLTable NameWidth=* Charset=UTF-8
IndexIgnore .htaccess .ht*
AddType application/epub+zip .epub
<IfModule mod_headers.c>
  Header set Access-Control-Allow-Origin "*"
</IfModule>
```

Nginx ignora `.htaccess`; vai na configuração do site:

```nginx
location /livros/ {
  autoindex on;
  add_header Access-Control-Allow-Origin *;
}
```

Formatos lidos: **PDF, EPUB, DOC, DOCX, TXT, MD**. Subpastas não são lidas.

### Capas

Uma imagem com o **mesmo nome** do livro, na mesma pasta, vira a capa dele (JPG, JPEG, PNG ou WEBP; maiúsculas e minúsculas não importam):

```
alice.pdf
alice.jpg
```

Para gerar a capa de um PDF a partir da primeira página (poppler-utils):

```sh
pdftoppm -jpeg -singlefile -scale-to 320 alice.pdf alice
```

Sem imagem, a extensão mostra a inicial do nome.

## Conferir antes de subir

```sh
# 200, uma página "Index of ..." e o header de CORS
curl -i https://server.com/ | head -20

# o JSON é válido
python -m json.tool servers.json
```

## Como a extensão trata a lista

| Situação | O que acontece |
|---|---|
| URL nova na lista | O servidor entra no marketplace. |
| URL retirada da lista | O servidor sai do marketplace. |
| Servidor fora do ar | Continua com os livros da última leitura. |
| `servers.json` inacessível ou inválido | Nada muda. |
| Servidor que o usuário adicionou à mão | Nunca é tocado; se a mesma URL estiver na lista, passa a contar como da lista. |

O GitHub guarda cópia do arquivo por cerca de 5 minutos: uma mudança recente pode levar esse tempo para chegar, e só é lida no próximo início do navegador ou abertura do marketplace.
