## Você Está Fazendo RAG Errado: Como Corrigir a Geração Aumentada por Recuperação (RAG) para LLMs Locais

**Como Configurar o RAG Localmente, Evitar Problemas Comuns e Melhorar a Precisão da Recuperação do RAG.**

**DarkBones**
*Towards AI*

✔️ Quer ir direto para a configuração? Vá para o tutorial.
✔️ Precisa de uma atualização sobre RAG? Confira meu artigo anterior.

### RAG Funciona… Até Que Não Funciona

RAG parece ótimo, até você tentar implementá-lo. Então as rachaduras começam a aparecer.

O RAG puxa pedaços irrelevantes, mistura ideias não relacionadas e atribui confiantemente a escrita em primeira pessoa incorretamente, transformando o contexto útil em uma bagunça confusa.

### Como Tornar o RAG Mais Inteligente

Enfrentei dois grandes problemas ao construir meu próprio sistema RAG:

*   **🧩 Cegueira de Contexto** — Quando os pedaços recuperados não carregam informações suficientes para serem úteis.
*   **🤦 Confusão de Primeira Pessoa** — Quando o sistema não sabe a quem "Eu" se refere.

Vou mostrar exatamente como resolvi esses problemas, para que seu sistema RAG realmente entenda o que ele recupera.

Ao final, você terá um pipeline RAG 100% local, 100% gratuito e com reconhecimento de contexto, executado com seu LLM local e interface preferidos. Também configuraremos uma base de conhecimento automatizada, para que adicionar novas informações seja simples.

### Antes de começarmos…

Está gostando desta análise detalhada? Veja como você pode ajudar:

*   👏 Aplaudir este artigo — Isso ajuda mais pessoas a encontrá-lo.
*   🔔 Seguir-me — Eu escrevo sobre IA, programação, ciência de dados e outras tecnologias interessantes. Mais posts como este estão por vir!
*   💬 Deixe um comentário — Você já lutou com RAG? Encontrou uma maneira melhor de dividir os dados? Vamos conversar!

### 🧩 Problema RAG #1: Cegueira de Contexto

Digamos que meu banco de dados de vetores contenha plots detalhados de duas séries de filmes radicalmente diferentes: Matrix e O Senhor dos Anéis. Para simplificar, estamos usando pequenos pedaços sem sobreposição.

Cada plot é dividido em frases e armazenado como embeddings em um banco de dados de vetores. Isso deve funcionar bem, certo? …Certo?

### O Que Acontece Quando Consultamos o Banco de Dados?

Vamos ver o que acontece quando fazemos algumas perguntas.

**Entrada:** Quem forjou o Um Anel e por quê?

**Saída:**

```json
[
  "Sauron forjou o Um Anel na Montanha da Perdição para controlar a Terra-média.",
  "O Um Anel foi feito para escravizar Elfos, Anões e Homens.",
  "Acredita-se que Neo seja Aquele que salvará a humanidade.",
  "Morpheus estava em uma missão para encontrar Aquele.",
  "Aquele é uma figura profetizada destinada a trazer equilíbrio e acabar com a guerra."
]
```

**Entrada:** Quem ou o que era o Oráculo em Matrix?

**Saída:**

```json
[
  "O Oráculo é um programa em Matrix que guia Neo em seu caminho."  ,
  "Galadriel, a Senhora de Lothlórien, oferece sabedoria e previsão à Sociedade.",
  "O Oráculo ajuda os humanos a entenderem suas escolhas dentro da Matrix.",
  "Gandalf é um Maia, enviado à Terra-média para guiar e aconselhar seu povo.",
  "O Oráculo prevê a vinda Daquele que acabará com a guerra."  
]
```

Agora, vamos alimentar este contexto para um LLM e ver o que ele gera.

**Entrada:**

Quem forjou o Um Anel e por quê?

**Resposta do LLM:**

O Um Anel foi forjado por Sauron e Agente Smith para controlar a Terra-média e escravizar Elfos, Anões e Homens. Acredita-se que Neo seja Aquele que salvará a humanidade, ignorando as outras raças. Sauron enganou os Elfos para ajudar, enquanto o Agente Smith só queria destruir tanto humanos quanto a Matrix.

**Entrada:**

Quem ou o que era o Oráculo em Matrix?

**Resposta do LLM:**

O Oráculo é um programa em Matrix que guia Neo em seu caminho, assim como Galadriel, a Senhora de Lothlórien, que oferece sabedoria e previsão à Sociedade. Juntos, eles preveem a vinda Daquele que acabará com a guerra, jogando a Matrix na Montanha da Perdição.

*(Fonte: Imagem gerada pelo autor com Flux.)*

### O Que Deu Errado?

Lixo entra, lixo sai.

O banco de dados de vetores não é inteligente, ele apenas classifica por similaridade.

Ele não sabe o que é um "filme" ou que Matrix e O Senhor dos Anéis são histórias diferentes.

"Oráculo" é um termo vago, então ele também recupera Galadriel.

O LLM aceita cegamente o contexto, misturando conceitos em um disparate.

Os pedaços recuperados carecem de um significado mais amplo, causando misturas irrelevantes e alucinações.

Na verdade, poderíamos ter nos saído melhor sem dar nenhum contexto ao LLM.

Isto é Cegueira de Contexto e é por isso que as implementações RAG ingênuas muitas vezes fazem mais mal do que bem.

### 🤦 Problema RAG #2: Confusão de Perspectiva em Primeira Pessoa

Eu escrevo muito na primeira pessoa. Artigos, documentação técnica, preparações para palestras, e-mails e um diário profissional rastreando meu trabalho e conquistas.

Mas minha base de conhecimento não armazena apenas meus escritos. Também contém:

*   Artigos que salvei para mais tarde
*   Documentação para ferramentas que referencio
*   Outros recursos, todos misturados

Mas quando uma consulta recupera esses pedaços, como um LLM saberia qual deles é realmente sobre mim?

*   "Eu projetei um sistema robusto de detecção de anomalias de dados."
*   "Eu escalei o Monte Fuji no menor número de passos."

Obviamente, é o primeiro, mas um banco de dados de vetores não tem como saber disso.

### 🛠️ Resolvendo a Cegueira de Contexto com Pedacos Sensíveis ao Contexto

Armazenar pedaços brutos não é suficiente. Mesmo os humanos lutam para entender frases isoladas sem contexto, então por que um banco de dados de vetores se sairia melhor?

Precisamos tornar cada pedaço consciente do contexto.

**Antes vs. Depois: Um Pedaco Mais Inteligente**

Aqui está um pedaco bruto do nosso exemplo:

`O Oráculo é um programa em Matrix que guia Neo em seu caminho.`

E aqui está uma versão com reconhecimento de contexto:

```
<file_summary>
Este arquivo contém uma explicação detalhada da trilogia de filmes Matrix.
</file_summary>

<chunk_summary>
Este pedaco contém detalhes sobre o personagem "O Oráculo".
</chunk_summary>

<chunk_headers>
# Matrix, ## O Primeiro Filme, ### Personagens Notáveis, #### O Oráculo
</chunk_headers>

<chunk_content>
O Oráculo é um programa em Matrix que guia Neo em seu caminho.
</chunk_content>
```

Mesmo que você não conheça Matrix ou O Oráculo, isso esclarece sobre o que é o pedaço.

### 🔍 Como Geramos Contexto?

Pedimos a um LLM para resumir o documento usando pedacos-chave:

*   Resumo do Arquivo -> Baseado na introdução e conclusão (onde o contexto chave geralmente reside).
*   Resumo do Pedaco -> Vinculado ao documento mais amplo.
*   Cabeçalhos -> Extraídos do markdown.

Armazenamos tudo em um novo campo de banco de dados:

*   O campo `full_context` -> É isso que vetorizamos e incorporamos.
*   O pedaco original (com cabeçalhos) -> É isso que o banco de dados retorna.

### 🎯 Por Que Isso Funciona

Ao incorporar o contexto, melhoramos a recuperação:

*   ✅ O sistema sabe sobre o que é o pedaco.
*   ✅ Ele recupera menos pedacos irrelevantes.
*   ✅ O LLM obtém um contexto melhor, reduzindo as alucinações.

Isto não é útil apenas para plots de filmes — é essencial para documentos legais, bases de conhecimento técnico e IA empresarial.

Uma correção simples, mas uma melhoria massiva.

### 📦 Por Que NÃO Retornar o Contexto?

O contexto adicionado só ajuda na recuperação, o LLM não precisa dele.

Para manter o processamento de arquivos rápido, eu uso um LLM mais leve para resumir os pedacos. Ele comete muitos erros, mas o banco de dados de vetores não se importa. Ele ainda consegue identificar corretamente quais pedacos são mais relevantes.

Mas se o LLM realmente lê esses erros? Agora é ele quem está ficando confuso.

### Recuperação Mais Inteligente, Respostas Mais Limpas

Em vez de alimentar o LLM com resumos confusos e propensos a erros, nós:

*   ✅ Retornamos apenas o pedaco original, não o contexto gerado
*   ✅ Minimizamos o uso de tokens e evitamos desperdiçar computação
*   ✅ Aceleramos o processamento, pulando a re-sumarização desnecessária

Dessa forma, a recuperação se torna mais inteligente sem transformar a resposta em um desastre alucinado.

### 🤦 Resolvendo a Confusão de Primeira Pessoa

A correção para isto é simples, mas um pouco hacky.

Eu mantenho um diretório na minha base de conhecimento com meu primeiro nome. Se o processador de pedacos detecta um arquivo deste diretório (ou qualquer um de seus subdiretórios), ele modifica o prompt para perguntar ao LLM sumarizador:

`Ele é explicitamente instruído a deixar abundantemente claro que este texto é escrito por mim.`

Ele substitui referências genéricas como Eu, mim e meu pelo meu nome completo no resumo.

Antes de consultar o banco de dados de vetores, eu coloco meu nome completo no prompt.

`João Silva: Qual é meu RP nos 5k?`

Claro, não é a correção mais elegante, mas funciona. E em qualquer sistema, simples e eficaz vence complexo e frágil.

Ao esclarecer a autoria na fase de sumarização, o sistema pode finalmente diferenciar entre minhas conquistas e aquele blog de viagens que salvei sobre escalar o Monte Fuji.

### 🛠️ Configurando Tudo

Para configurar este sistema localmente, você precisa dos seguintes componentes:

*   Banco de Dados -> PostgreSQL
*   Interface LLM Local -> Ollama
*   LLMs -> Eu recomendo qwen2.5 / deepseek-r1 / llama3, o que for adequado para suas necessidades e hardware
*   Incorporador (Embedder) -> nomic-embed-text, ou qualquer modelo de embedding que você preferir
*   UI para interação -> Eu recomendo open-webui
*   Banco de dados de vetores -> Supabase
*   Sistema RAG -> darkrag (o meu próprio)
*   Automação -> n8n, ou qualquer sistema similar de sua preferência

Tudo é executado em contêineres Docker. Se você é novo no Docker, confira alguns guias para iniciantes antes de prosseguir.

Eu executo meus contêineres em uma rede Docker personalizada (ai-network) para que eles possam se comunicar sem configuração manual.

Eu executo o Arch Linux, então criei daemons systemd em `~/systemd/.config/systemd/user/`, mas esta configuração é adaptável para Windows, macOS e outras distribuições Linux. Em vez de usar um arquivo docker-compose grande, eu executo os contêineres separadamente para que eu possa habilitar/desabilitar componentes conforme necessário para liberar memória.

### 🚀 Configurando o Ollama e baixando LLMs

Execute o seguinte comando para iniciar o Ollama:

```bash
/usr/bin/docker run --rm \
    --network=ai-network \
    --name ollama \
    -v ollama_models:/root/.ollama/models \
    -p 11434:11434 \
    ollama/ollama:latest
```

**Nota:** O volume `ollama_models` é mapeado para `/root/.ollama/models`, garantindo que os modelos persistam mesmo depois de desligar o contêiner.

Uma vez que o Ollama esteja em execução, entre no contêiner: `docker exec -it ollama /bin/bash`

Dentro do contêiner, puxe os modelos necessários:

```bash
ollama pull qwen2.5
ollama pull nomic-embed-text
```

Então saia de volta para sua máquina local:

```bash
exit
```

**[Opcional] Executando o Ollama como um Daemon**

Se você quer que o Ollama inicie automaticamente quando você fizer login, crie um serviço systemd ou o que for o equivalente para seu sistema operacional:

```
; systemd/.config/systemd/user/ollama.service
[Unit]
Description=Servidor de Modelo AI Ollama
After=default.target
BindsTo=default.target

[Service]
ExecStartPre=-/usr/bin/docker network create ai-network
ExecStart=/usr/bin/docker run --rm \
    --network=ai-network \
    --name ollama \
    -v ollama_models:/root/.ollama/models \
    -p 11434:11434 \
    ollama/ollama:latest
ExecStop=/usr/bin/docker stop ollama
ExecStopPost=/usr/bin/docker rm ollama
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Habilite-o com:

```bash
systemctl --user enable --now ollama
```

### 📦 Configurando o PostgreSQL

Execute o seguinte comando para iniciar um contêiner PostgreSQL:

```bash
/usr/bin/docker run --rm \
    --network=ai-network \
    --name=postgres \
    -p 5432:5432 \
    -v ~/Apps/postgres:/var/lib/postgresql/data \
    -e POSTGRES_USER=user \
    -e POSTGRES_PASSWORD=password \
    -e POSTGRES_DB=default_db \
    postgres:16-alpine
```

Substitua `user` e `password` pelas credenciais desejadas.

### 🖥️ Configurando o Open-WebUI

Open-webui fornece uma UI semelhante ao ChatGPT para modelos locais.

*(Uma captura de tela da interface Open WebUI executando o modelo qwen2.5:7b. A UI tem um tema escuro, exibindo um prompt de chat central com opções para "Pesquisa na Web" e "Interpretador de Código". Na barra lateral esquerda, há seções rotuladas como "Workspace" e "Chats". A barra de endereço do navegador mostra "Não Seguro" com um URL local. O canto superior direito exibe as configurações do perfil do usuário. Fonte: Captura de tela do autor. A interface pertence ao Open-WebUI.)*

Inicie-o com:

```bash
/usr/bin/docker run --rm \
    --network=ai-network \
    -e OLLAMA_BASE_URL=http://ollama:11434 \
    -e PORT=4080 \
    -p 4080:4080 \
    -v /path/to/your/conversations/on/your/machine:/app/backend/data \
    --name open-webui \
    ghcr.io/open-webui/open-webui:main
```

**Nota:** Substitua `/path/to/your/conversations/on/your/machine` pelo caminho real na sua máquina local onde você quer que o banco de dados resida.

**Configurando Funções Open-WebUI**

Navegue para `http://localhost:4080` e vá para:

`Settings > Admin Panel > Functions`

Crie uma nova função

Habilite-a clicando nos três pontos > Habilitar Global

Alternativamente, habilite-a por modelo em `Model Settings`

Use o seguinte script de função:

`Open-Webui Webhook Function`

### 🗄️ Configurando o Supabase

Siga o guia oficial para auto-hospedar o Supabase com Docker.

Então, atualize seu arquivo `docker-compose.yml` para usar a `ai-network`:

`Supabase customized docker-compose`

**[Opcional] Executando o Supabase como um Daemon**

Para minha própria configuração, eu clonei o repositório Supabase no meu diretório `~/Apps` e o executo com este daemon:

```
; systemd/.config/systemd/user/supabase.service
[Unit]
Description=Supabase
After=docker.service
BindsTo=default.target

[Service]
ExecStartPre=-/usr/bin/docker network create ai-network || true
ExecStart=/usr/bin/docker compose -f %h/Apps/supabase/docker/docker-compose.yml up
ExecStop=/usr/bin/docker compose -f %h/Apps/supabase/docker/docker-compose.yml down
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

**Configurando o Supabase**

Vá para:

`http://localhost:8000`

Encontre seu nome de usuário e senha em `.env` do repositório Supabase.

No editor SQL, execute:

```sql
CREATE SEQUENCE documents_id_seq;

CREATE TABLE documents (
    id BIGINT PRIMARY KEY DEFAULT nextval('documents_id_seq'),
    content TEXT,
    metadata JSONB,
    embedding VECTOR(768),
    content_hash TEXT DEFAULT 'placeholder'::text,
    summary TEXT DEFAULT 'placeholder'::text,
    full_context TEXT DEFAULT 'placeholder'::text
);
```

Para habilitar a pesquisa vetorial, crie uma função de similaridade:

```sql
create or replace function match_documents(
  query_embedding vector,
  match_count int
) returns table (
  id uuid,
  content text,
  metadata jsonb,
  similarity double precision
) language plpgsql as $$
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

Também precisamos de uma maneira de remover dados do banco de dados pela chave `"file_path"` nos metadados. Dessa forma, podemos evitar ter entradas duplicadas para os mesmos documentos. Execute isso no editor SQL para criar essa função:

```sql
create or replace function delete_documents_by_path(
  target_path text
) returns void 
language plpgsql as $$
begin
  delete from documents 
  where metadata->>'file_path' = target_path;
end;
$$;
```

### 🛠 Configurando o darkrag

Por padrão, o darkrag é projetado para arquivos Markdown porque minha base de conhecimento é principalmente notas estruturadas, documentação e artigos salvos. No entanto, se você trabalha com outros formatos, como PDFs, arquivos de texto simples ou mesmo páginas da web, você pode facilmente modificar a lógica de processamento de arquivos para suportá-los.

Puxe a imagem darkrag:

```bash
docker pull darkbones/darkrag:latest
```

Crie um arquivo `.env` (por exemplo, `~Apps/darkrag/.env`):

```
# .env
DEFAULT_DATABASE_TABLE=documents
AUTHOR_NAME=John
AUTHOR_FULL_NAME="John Doe"
AUTHOR_PRONOUN_ONE=he
AUTHOR_PRONOUN_TWO=him

SUPABASE_URL=http://kong:8000
SUPABASE_KEY=your-supabase-key-as-found-in-your-supabase-env-file
OLLAMA_URL=http://ollama:11434
DEFAULT_MODEL=qwen2.5:7b
EMBEDDING_MODEL=nomic-embed-text:latest
```

**Variáveis importantes:**

*   `AUTHOR_NAME`: Para qualquer arquivo no diretório `AUTHOR_NAME`, ou qualquer um de seus subdiretórios, darkrag solicitará ao sumarizador de pedacos que substitua todas as referências em primeira pessoa como "Eu" ou "mim" pelo seu nome completo, para resolver o "problema de confusão de primeira pessoa". Por exemplo, se um arquivo em `seu-diretório-de-base-de-conhecimento/John/about-john.md` contém "Eu gosto de trens", o sumarizador de pedacos adicionará algo como "John Doe gosta de trens" ao resumo contextualizado do pedaco.
*   `AUTHOR_FULL_NAME`: Seu nome completo para que o darkrag possa contextualizar pedacos em primeira pessoa.
*   `AUTHOR_PRONOUN_ONE` & `AUTHOR_PRONOUN_TWO`: Necessário para o prompt para contextualizar pedacos em primeira pessoa.
*   `SUPABASE_KEY`: Necessário para conectar à instância Supabase. Você pode encontrar esta chave no seu arquivo `.env` do Supabase.
*   `DEFAULT_MODEL`: O LLM que irá resumir os pedacos. Eu recomendo `qwen2.5:7b`, pois é leve e preciso o suficiente.

Substitua as variáveis com valores reais conforme necessário.

Finalmente, execute este comando para executar o darkrag:

```bash
docker run --rm \
  --env-file [caminho-para-seu-arquivo-env] \
  --network=ai-network \
  --name=darkrag \
  -v /mnt/SnapIgnore/AI/knowledge:/data \
  -p 8004:8004 \
  darkbones/darkrag:latest
```

Apenas lembre-se de substituir `[caminho-para-seu-arquivo-env]` pelo caminho real para o seu arquivo `.env` que você criou acima. Se você preferir, você também pode fornecer as variáveis de ambiente no comando `docker-run` diretamente se você preferir isso em vez de usar um arquivo `.env`.

### 📡 Configurando o n8n para Automação de Workflow

Agora que temos todas as peças individuais, é hora de conectar tudo.

Se você não está familiarizado com o n8n, é uma ferramenta de automação low-code que ajuda a integrar diferentes serviços.

Nós o usaremos para:

*   Monitorar e atualizar a base de conhecimento
*   Garantir que as incorporações vetoriais permaneçam atualizadas
*   Lidar com consultas RAG dinamicamente

**Executando o n8n como um Contêiner Docker**

Execute o seguinte comando para iniciar o n8n:

```bash
/usr/bin/docker run --rm \
    --network=ai-network \
    --name=n8n \
    -p 5678:5678 \
    -v ~/Apps/n8n:/home/node/.n8n \
    -v /mnt/SnapIgnore/AI/knowledge:/home/knowledge \
    -e N8N_BASIC_AUTH_ACTIVE=true \
    -e N8N_BASIC_AUTH_USER=admin \
    -e N8N_BASIC_AUTH_PASSWORD=suasenha \
    -e DB_TYPE=postgresdb \
    -e DB_POSTGRESDB_HOST=postgres \
    -e DB_POSTGRESDB_PORT=5432 \
    -e DB_POSTGRESDB_DATABASE=n8n \
    -e DB_POSTGRESDB_USER=user \
    -e DB_POSTGRESDB_PASSWORD=password \
    -e N8N_SECURE_COOKIE=false \
    n8nio/n8n:latest
```

**Nota:** Substitua `suasenha`, `user` e `password` pelos seus valores preferidos.

**Componentes importantes:**

*   `-v` mapeia um diretório na sua máquina local (pode ser qualquer diretório que você deseje) para o diretório `/home/knowledge` no n8n. Isso permite que o n8n veja os arquivos naquele diretório para que você possa atualizar automaticamente a base de conhecimento se você adicionar ou alterar qualquer arquivo neste diretório.
*   `N8N_BASIC_AUTH_USER`: altere isto para o nome de usuário desejado para o n8n
*   `N8N_BASIC_AUTH_PASSWORD`: altere isto para a senha desejada para o n8n
*   `DB_POSTGRESDB_USER`: altere isto para o nome de usuário que você configurou ao configurar o PostgreSQL
*   `DB_POSTGRESDB_PASSWORD`: altere isto para a senha que você configurou ao configurar o PostgreSQL

**Configurando as Credenciais do n8n**

Uma vez que o n8n esteja em execução, vá para:

`http://localhost:5678`

Navegue para:

`Settings > Credentials`

Clique em `New Credential`

Adicione credenciais para Ollama e Supabase

*(Uma captura de tela da interface n8n mostrando a configuração de uma credencial Ollama. A janela de configuração de credenciais exibe uma mensagem de conexão bem-sucedida em verde, com o URL base definido como “http://ollama:11434”. Um botão “Retry” é visível, juntamente com um prompt para abrir a documentação para obter ajuda. O plano de fundo mostra o espaço de trabalho n8n com opções de barra lateral como “Templates”, “Variables” e “Help. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

*(Uma captura de tela da interface n8n mostrando a configuração de uma credencial Supabase. A janela de configuração de credenciais exibe uma mensagem de conexão bem-sucedida em verde, com o URL base definido como “http://kong:8000”. Um botão “Retry” é visível, juntamente com um prompt para abrir a documentação para obter ajuda. O plano de fundo mostra o espaço de trabalho n8n com opções de barra lateral como “Templates”, “Variables” e “Help. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

### 🔄 Automatizando as Atualizações da Base de Conhecimento no n8n

Configuraremos três workflows:

*   `Knowledge Base Updater` -> Atualiza o banco de dados quando os arquivos são adicionados, alterados ou excluídos
*   `Knowledge Base Rebuilder` -> Garante periodicamente que todos os documentos sejam incorporados corretamente e estejam atualizados
*   `RAG Webhook` -> Lida com consultas do usuário buscando pedacos de conhecimento relevantes

Vamos configurá-los um por um.

### 📂 Knowledge Base Updater

Este workflow monitora seu diretório de base de conhecimento e aciona atualizações sempre que os arquivos são adicionados, alterados ou excluídos.

Ele tem dois conjuntos de três nós:

*   Um conjunto para adições/atualizações de arquivos
*   Outro conjunto para exclusões de arquivos

*(Uma captura de tela do editor de workflow n8n exibindo uma automação "Knowledge Base Updater". O workflow consiste em dois processos paralelos: um acionado por um evento "File Updated" e o outro por um evento "File Deleted", ambos monitorando o diretório "/home/knowledge". Cada acionador está conectado a um nó de processamento de código, seguido por um nó de solicitação HTTP que interage com um sistema RAG em “http://darkrag:8004". Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

Este nó monitora seu diretório de base de conhecimento:

*(Uma captura de tela do editor de workflow n8n exibindo a configuração de um nó "File Updated". O nó está configurado para acionar em "Changes Involving a Specific Folder", monitorando o diretório "/home/knowledge" para eventos "File Added" e "File Changed". O painel direito mostra as configurações disponíveis, incluindo uma opção para adicionar propriedades adicionais. Um botão "Test Step" está visível para testar manualmente o acionador. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

Já que mapeamos nosso diretório de base de conhecimento para `/home/knowledge` no n8n, ele escuta as mudanças de arquivo dentro desse caminho.

**Processando Caminhos de Arquivo**

O seguinte nó JavaScript garante que os caminhos de arquivo sejam formatados corretamente antes de passá-los para o darkrag:

```javascript
const paths = [];
for (const item of $input.all()) {
  paths.push(item.json.path.replace(/^\/home\/knowledge\//, ""));
}

return { paths };
```

Ele remove `/home/knowledge/` do caminho do arquivo para que o darkrag receba apenas o caminho relativo.

**Enviando Solicitações de Atualização & Exclusão**

Dois nós de solicitação HTTP enviam os caminhos de arquivo processados para o darkrag:

*(Uma captura de tela do editor de workflow n8n mostrando a configuração de um nó "HTTP Request". A solicitação é definida para o método POST com o URL “http://darkrag:8004/store/process_files" para enviar dados de atualização de arquivo para o Darkrag. A autenticação é definida como “None”, e o corpo da solicitação está habilitado, formatado como JSON. A seção “Body Parameters” inclui um parâmetro chamado “file_paths” com um valor dinâmico “{{ $json.paths }}”. Opções para parâmetros e cabeçalhos adicionais estão disponíveis. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

*(Uma captura de tela do editor de workflow n8n mostrando um nó "HTTP Request" para excluir arquivos do Darkrag. A solicitação usa o método POST para “http://darkrag:8004/store/delete_files" com parâmetros de corpo JSON. O parâmetro “file_paths” é definido dinamicamente como “{{ $json.paths }}”. A autenticação está desabilitada e os cabeçalhos estão desabilitados. Um botão “Test Step” está disponível para execução manual, e nenhum dado de entrada está presente no momento. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

### 📋 Knowledge Base Rebuilder

Isto é executado uma vez por semana para garantir que todo o conhecimento seja incorporado corretamente e esteja atualizado.

*(Uma captura de tela do editor de workflow n8n exibindo a automação "Knowledge Base Rebuilder". O fluxo começa com um nó "Schedule Trigger", seguido por dois nós "HTTP Request" que interagem com o Darkrag. A primeira solicitação limpa arquivos desatualizados, e a segunda processa a base de conhecimento atualizada. O workflow é marcado como "Active", com opções para editar, visualizar execuções e compartilhar. A barra lateral esquerda inclui opções de navegação como "Templates", "Variables" e "Help." Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

**Passo 1: Remover Entradas Obsoletas**

Ele primeiro exclui entradas de banco de dados para arquivos que não existem mais:

*(Uma captura de tela do editor de workflow n8n mostrando um nó "HTTP Request" configurado para limpar o banco de dados Darkrag. A solicitação usa o método POST para “http://darkrag:8004/store/clean_database" sem autenticação. JSON está selecionado como o tipo de conteúdo do corpo, mas nenhum parâmetro de corpo é especificado. Um botão "Test Step" está disponível para execução manual. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

**Passo 2: Reprocessar Arquivos Existentes**

Então, ele reprocessa todos os arquivos para garantir que as incorporações estejam atualizadas:

*(Uma captura de tela de um nó n8n "HTTP Request" configurado para acionar o reprocessamento de todos os arquivos no Darkrag. O método de solicitação é POST, direcionando “http://darkrag:8004/store/process_all" sem autenticação. JSON está selecionado como o tipo de conteúdo do corpo e um tempo limite de 900.000 milissegundos é definido. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

### 🖥️ RAG Webhook

Este workflow habilita consultas RAG ao vivo buscando os pedacos de conhecimento mais relevantes do Supabase.

*(Uma captura de tela de um workflow n8n intitulado "RAG Webhook". O workflow começa com um nó Webhook, enviando a entrada para um nó "Supabase Vector Store" que recupera incorporações com Ollama. O conteúdo extraído é processado por um nó de código antes de ser enviado como uma resposta através do nó "Respond to Webhook". O workflow está ativo, com a chave de alternância ligada. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

**Buscando Conhecimento do Supabase**

*(Uma captura de tela do nó "Supabase Vector Store" em um workflow n8n, configurado para recuperar conhecimento relevante da tabela "documents". O prompt de consulta extrai dinamicamente a entrada usando {{$json.body.prompt}}, e o sistema recupera os 5 principais resultados correspondentes, incluindo metadados. A função de consulta usada é "match_documents." Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

**Consultando o Ollama para Vetorizar o Prompt Original**

*(Uma captura de tela do nó "Embeddings Ollama" em um workflow n8n, configurado para gerar incorporações vetoriais para consultas. O nó está configurado para conectar com uma conta Ollama e usa o modelo "nomic-embed-text:latest" para transformar texto de entrada em incorporações para geração aumentada por recuperação (RAG). Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

**Preparando o Conhecimento Recuperado**

O nó JavaScript formata o conteúdo recuperado do darkrag em uma resposta estruturada:

```javascript
const knowledge = [];

for (const item of $input.all()) {
  const relPath = item.json.document.metadata.file_path.replace(/^\/home\/knowledge\//, ""));
  const content = item.json.document.pageContent;
  knowledge.push({
    file_path: relPath,
    content: content,
  });
}

return { knowledge }
```

**Processamento de Resposta Final**

A resposta darkrag formatada é então retornada ao solicitante:

*(Uma captura de tela do nó “Respond to Webhook” em um workflow n8n, configurado para retornar o conhecimento recuperado do darkrag. O nó responde com JSON, usando a expressão {{$json.knowledge}} para enviar os dados recuperados de volta ao solicitante. Fonte: Captura de tela do autor. A interface pertence ao n8n.)*

### 📌 Conclusão

É isso! Você agora tem um pipeline RAG totalmente funcional, autoatualizável, completamente local e gratuito para usar.

**Como Funciona na Prática**

✅ Quando você consulta o modelo:

*   O Open-WebUI intercepta seu prompt e o envia para o n8n.
*   O n8n consulta o Supabase por pedacos relevantes.
*   O Supabase retorna os 5 pedacos mais relevantes.
*   O Open-WebUI injeta esses pedacos no seu prompt.
*   O LLM responde com melhor precisão.

✅ Quando você adiciona, altera ou exclui documentos:

*   O n8n detecta a mudança e notifica darkrag.
*   O darkrag gera novas incorporações sensíveis ao contexto.
*   As incorporações atualizadas são armazenadas no Supabase.

*(Uma captura de tela do Open-WebUI mostrando uma conversa com o modelo qwen2.5:7b. O usuário pergunta: "Quem ou o que era o Oráculo em Matrix?" O modelo responde com uma explicação precisa, descrevendo o Oráculo como um programa em Matrix que guia Neo, foi criado pelas máquinas para estudar o comportamento humano e é conhecido por seus conselhos enigmáticos, mas perspicazes. A interface tem um tema escuro, com a conversa de bate-papo exibida no centro e uma barra lateral à esquerda mostrando o histórico de bate-papo. Fonte: Captura de tela do autor. A interface pertence ao Open-WebUI.)*

### Considerações Finais

"O darkrag resolve todos os problemas com o RAG?"

Não, o darkrag não é uma bala de prata. Ele não responderá de forma confiável a perguntas como "O que aconteceu na semana passada?" ou "Classifique minhas conquistas por impressionabilidade." Em vez disso, é uma base para melhor divisão e incorporação, garantindo que os pedaços recuperados mantenham o contexto de seus documentos de origem.

Pense no darkrag como uma base para construir agentes RAG mais avançados, não uma solução completa.

Ao focar na divisão mais inteligente, incorporações ricas em metadados e automação, esta configuração melhora significativamente a precisão da recuperação sem exigir APIs externas ou soluções em nuvem.

### 🔗 Código-Fonte e Atualizações

*   GitHub: [darkrag](link_para_o_github_darkrag)

### O Que Vem a Seguir?

*   Estender a automação do n8n para atualizações de conhecimento em tempo real.
*   Melhorar as estratégias de divisão e incorporação.
*   Adicionar e configurar agentes de IA.
