# Guia: Criando Repositórios e Chaves SSH via Terminal

Este guia demonstra o método atualizado para criar repositórios e configurar a autenticação via SSH no GitHub, tudo através da linha de comando.

## ⚠️ Pré-requisitos

Antes de começar, você vai precisar de:

1.  **Uma conta no GitHub.**
2.  **Um terminal Linux** (ou um ambiente similar como o WSL no Windows).
3.  **Um Personal Access Token (Token de Acesso Pessoal)**.

Para criar seu token, siga o caminho: [github.com/settings/tokens](https://github.com/settings/tokens).
![Permissões necessárias para o token no GitHub](https://media.licdn.com/dms/image/v2/D4D12AQFTjt04O2kFVQ/article-inline_image-shrink_400_744/B4DZatAdKYHsAY-/0/1746659300378?e=1755129600&v=beta&t=sth7ctlaAL5L3aDtrHDfJY57LhskS4Ao3_U-VTFBELU)

* Clique em "Generate new token".
* Dê um nome para o seu token (ex: "API Terminal").
* Defina uma data de expiração.
* Em **Repository permissions**, marque a permissão `repo` (controle total de repositórios privados).
* Em **Administration permissions**, marque `write:public_key` para gerenciar chaves SSH.
* Clique em "Generate token", **copie o token gerado e salve-o em um local seguro**. Você não poderá vê-lo novamente.

---

## 🚀 Criando um Repositório Remoto via API

No comando `curl`, você precisa passar seu token no cabeçalho (header) `Authorization`, na frente da palavra `Bearer`, como mostra o exemplo abaixo.

![Exemplo de como formatar o header de autorização com o token](https://media.licdn.com/dms/image/v2/D4D12AQEIOxdVgHzFHQ/article-inline_image-shrink_400_744/B4DZatCSaRHQAk-/0/1746659780641?e=1755129600&v=beta&t=TjJM1TCwQ3C4pN3XbkLL8nd_9kuP52vuAOXz15ap644)


### Comando para Criar Repositório

O comando a seguir envia uma requisição `POST` para a API do GitHub, criando um novo repositório na sua conta.

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <SEU_TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  [https://api.github.com/user/repos](https://api.github.com/user/repos) \
  -d '{"name":"NomeDoSeuRepo","description":"Descrição do repositório.","private":false}'
```

**Onde:**
* `Authorization: Bearer <SEU_TOKEN>`: Substitua `<SEU_TOKEN>` pelo seu Personal Access Token.
* `-d '{...}'`: Este é o corpo da requisição em formato JSON.
    * `name`: O nome do seu repositório (ex: "MeuProjeto-InfraAgil").
    * `description`: Uma breve descrição do que se trata o projeto.
    * `private`: Defina como `true` para um repositório privado ou `false` para um público.

Se o comando for executado com sucesso, a resposta será um JSON contendo os detalhes do repositório recém-criado, incluindo o link `ssh_url`.

![Exemplo da resposta JSON da API com os links do novo repositório](https://media.licdn.com/dms/image/v2/D4D12AQGm3kaXFKTslQ/article-inline_image-shrink_1000_1488/B4DZatCsglHwAQ-/0/1746659887780?e=1755129600&v=beta&t=5ofPoXFUqlhc017HutzIe8tqfM6unWw6cLgA78wQslg)


---

## 🔑 Configurando a Chave SSH

A chave SSH cria um canal de comunicação seguro entre sua máquina local e o GitHub, permitindo que você envie e receba dados sem precisar digitar sua senha a cada vez.

### 1. Gerando uma Nova Chave SSH

O GitHub recomenda o uso do algoritmo **ed25519**. Para gerar um par de chaves (pública e privada), use o comando:

```bash
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
```
*Substitua o email pelo mesmo que você usa na sua conta do GitHub.*

![Execução do comando ssh-keygen no terminal](https://media.licdn.com/dms/image/v2/D4D12AQG_M0UVbC_QKw/article-inline_image-shrink_1000_1488/B4DZax5D.tGwAQ-/0/1746741248491?e=1755129600&v=beta&t=rIfh_ojRsMO393EfcF_tATEDUzAUbE43cyEh2wR_19Q)

Este comando criará dois arquivos no diretório `~/.ssh/`:
* `id_ed25519`: Sua chave **privada**. 🤫 **NUNCA compartilhe este arquivo!**
* `id_ed25519.pub`: Sua chave **pública**. É esta que você irá adicionar ao GitHub.

Você pode listar os arquivos para confirmar a criação com `ls -l ~/.ssh`.

![Listagem dos arquivos de chave pública e privada no diretório .ssh](https://media.licdn.com/dms/image/v2/D4D12AQFXSD574DD4Yw/article-inline_image-shrink_1500_2232/B4DZax5067HAAc-/0/1746741448815?e=1755129600&v=beta&t=9jrGahpZkR-OCxaAzS-RfLW6TZklgtVZB6cskaGAZgU)


### 2. Adicionando a Chave ao `ssh-agent`

O `ssh-agent` é um programa que gerencia suas chaves SSH e evita que você precise digitar sua senha repetidamente.

```bash
# Inicia o ssh-agent em background
eval $(ssh-agent -s)
```

![Comando para iniciar o ssh-agent](https://media.licdn.com/dms/image/v2/D4D12AQEtElJzxchsXg/article-inline_image-shrink_400_744/B4DZax7zvIHEAc-/0/1746741968213?e=1755129600&v=beta&t=vUvLojrcbgBWUVEWSKDG3xOoUc2RGktO2bBcgdGCGMo)

```bash
# Adiciona sua chave SSH privada ao agente
ssh-add ~/.ssh/id_ed25519
```

### 3. Adicionando a Chave Pública ao GitHub

Primeiro, exiba e copie o conteúdo da sua chave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```
*Copie toda a saída, desde `ssh-ed25519` até o final do seu email.*

![Conteúdo da chave pública exibido no terminal após o comando cat](https://media.licdn.com/dms/image/v2/D4D12AQF0j2TxNP7k4Q/article-inline_image-shrink_1000_1488/B4DZayApgzHsAQ-/0/1746743237102?e=1755129600&v=beta&t=9T_adrgwbD80UGDizixXFSCzYlSuB-j6nhehmMfg6LA)

Agora, use o `curl` para enviar a chave pública para sua conta do GitHub:

![Exemplo do comando curl preenchido com o token e a chave pública](https://media.licdn.com/dms/image/v2/D4D12AQEGXs51BBmiEA/article-inline_image-shrink_1500_2232/B4DZayEWBsGwAY-/0/1746744205931?e=1755129600&v=beta&t=QKKXBhQc3r-n651qTIycCUsaZGwpil6qzXLB9lwroOY)

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <SEU_TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  [https://api.github.com/user/keys](https://api.github.com/user/keys) \
  -d '{"title":"Nome-da-Chave","key":"COLE_SUA_CHAVE_PUBLICA_AQUI"}'
```

**Onde:**
* `"title"`: Um nome para identificar a chave (ex: "Meu-Notebook-Linux").
* `"key"`: Cole o conteúdo completo da sua chave pública que você copiou no passo anterior.

Para verificar se a chave foi adicionada com sucesso, acesse a seção de **[Chaves SSH e GPG](https://github.com/settings/keys)** nas configurações do seu GitHub.

![Página de Chaves SSH no GitHub mostrando a nova chave adicionada](https://media.licdn.com/dms/image/v2/D4D12AQESRqV6gSxJcg/article-inline_image-shrink_1000_1488/B4DZayGOGvGwAU-/0/1746744697801?e=1755129600&v=beta&t=tvH_E5pHEi_D3VhcwwYvNk1q4Aps6SXR5vBG2bNKmgQ)

---

## 📚 Documentação Oficial

Para mais detalhes, consulte a documentação oficial do GitHub:

* [Gerando uma nova chave SSH e adicionando-a ao ssh-agent](https://docs.github.com/pt/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
* [API REST para Repositórios (Criar um repositório para o usuário autenticado)](https://docs.github.com/pt/rest/repos/repos?apiVersion=2022-11-28#create-a-repository-for-the-authenticated-user)
* [API REST para Chaves Públicas de Usuários](https://docs.github.com/pt/rest/users/keys?apiVersion=2022-11-28)
