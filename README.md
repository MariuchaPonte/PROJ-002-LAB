PROJ-002-LAB
# VALIDAÇÃO DE HIPÓTESES
## Análise de dados de plataformas de streams para identificação de fatores de sucesso para um novo artista

2º Projeto da &lt;LAB>  
  
FERRAMENTAS UTILIZADAS:  
📈 BigQuery  
📊 PowerBI  
  
#Etapas 
  
🔵 Conectar/importar dados para as ferramentas  
🔵 Identificar e tratar valores nulos  
   📈  
**Identificando nulos na tabela:**  
Competition: 50 NULOS, na coluna: IN_SHAZAM_Charts
Tec_info: 95 nulos na coluna: Key
Spotify: não há nulos
  
**Fórmula SQL usada para encontrar onde havia nulos:**
>  
>SELECT  
> *  
>FROM  
>  `proj002-lab-mariucha-ponte.Projeto02.copetition`  
>WHERE  
>  track_id IS  NULL  
>  OR in_apple_charts IS  NULL  
>  OR in_apple_playlists IS  NULL  
>  OR in_deezer_charts IS  NULL  
>  OR in_deezer_playlists IS  NULL  
>  OR in_shazam_charts IS  NULL  

**Fórmula SQL usada para limpar os dados nulos, trazendo apenas as linhas cuja coluna x não tinha nulos:**  
>SELECT  
> *  
>FROM  
>  `proj002-lab-mariucha-ponte.Projeto02.copetition`  
>WHERE  
>  in_shazam_charts IS not NULL"  
  
🔵 Identificar e tratar valores duplicados	  
✔			"formula sql usadapara identificar duplicados:

SELECT
  track_name,
  artist_s_name,
  COUNT(*) as quant
FROM
  `proj002-lab-mariucha-ponte.Projeto02.spotify`
GROUP BY
  track_name,
  artist_s_name
having quant >1

spotify: 4 duplicados
tec_info: não há duplicados
competition: não há duplicados

para identificar todas as colunas das linhas onde identifiquei duplciados usei  ocodigo sql:

SELECT 
* 
FROM `proj002-lab-mariucha-ponte.Projeto02.spotify`
WHERE 
track_name = ""SNAP""
or track_name = ""About Damn Time""
or track_name = ""Take My Breath""
or track_name = ""SPIT IN MY FACE!""

apenas na faixa  About damn time, de Lizzo, há mes e dia de lancamento diferente. os demais nao identifiquei porque há duplicados.
verificar quando der o joind dos tres conjuntos de dados"  

🔵 Identificar e gerenciar dados fora do escopo de análise	  
✔			"codigo sqo para excluir uma coluna:

SELECT 
* except(key,mode)
 FROM `proj002-lab-mariucha-ponte.Projeto02.tec_info` "
🔵 Identificar e tratar dados discrepantes em variáveis ​​categóricas
✔			"codigo sql para substituir cacrateres espciais:

SELECT
  artist_s_name,
  REGEXP_REPLACE(artist_s_name, r'[^a-zA-Z0-9 ]', ' ')
  as sem_carcar_especial
FROM
  `proj002-lab-mariucha-ponte.Projeto02.spotify`

#codigo q significa caracteres especiias: r'[^a-zA-Z0-9 ]'"
🔵 Identificar e tratar dados discrepantes em variáveis ​​numéricas	  
✔			planiilha spotify tem um dado numerico sicrepante (pois esta como texto, tornando toda a coluna stribng) na linha 47 da coluna streams
🔵 Verificar e alterar os tipos de dados	  
✔			"SELECT
 safe_cast (streams as int64) as streams_limpo
FROM
  `proj002-lab-mariucha-ponte.Projeto02.spotify`


----

mas para tentar gerar o min max e avg junto com o safe cats usei essa 

SELECT 

  MIN(streams_limpo) AS minimo_streams_spotify,
  MAX(streams_limpo) AS maximo_streams_spotify,
  round(avg(streams_limpo), 2) as media_streams_spotify

FROM 
(  
  SELECT 
    SAFE_CAST(streams AS int64) as streams_limpo
       FROM 
       `proj002-lab-mariucha-ponte.Projeto02.spotify`
)"  

🔵 Unir (join) as tabelas de dados	  
✔			"juntei 4 views numa query so, e funcionou, cirando inclusive uma nova variavel , de soma de duas outras assim:

SELECT 


a.track_id , 

a.track_name ,

a.nome_artista_limpo,

a.artist_count ,

a.DATA_DE_LANCAMENTO ,

a.in_spotify_playlists ,

d.in_apple_playlists ,

d.in_deezer_playlists ,

c.soma_playlists_conc + in_spotify_playlists as soma_playlists , 

b.streams_limpo ,


FROM `proj002-lab-mariucha-ponte.Projeto02.spotify_limpo_sem_streams_com_obs_sobre_duplicados`

as a


join 

`proj002-lab-mariucha-ponte.Projeto02.spotify_streams_limpo` 

as b 


on 

a.track_id = b.track_id 


join 

`proj002-lab-mariucha-ponte.Projeto02.playlists_concorrentes_somadas` 

as c 


on 

a.track_id = c.track_id 


join 

`proj002-lab-mariucha-ponte.Projeto02.competition` 

as d 


on 

a.track_id = d.track_id

#adicionar as caracteristicas das musicas, dnceability, bpm, etc 

"
🔵 Criar novas variáveis ​​  
✔			"exemplo1:

SELECT  
  track_id,
  in_apple_playlists + in_deezer_playlists as playlists_concorrentes
FROM `proj002-lab-mariucha-ponte.Projeto02.copetition` 

---------------------------
exemplo2:

SELECT
  DATE(CONCAT(released_year, ""-"", released_month, ""-"", released_day)) AS DATA_DE_LANCAMENTO,
  SUM(in_spotify_charts) AS SOMA_CHARTS_SPOTFY,
  SUM(in_spotify_playlists) AS SOMA_PLAYLISTS_SPOTFY

  FROM
    `proj002-lab-mariucha-ponte.Projeto02.spotify` 
    GROUP BY DATA_DE_LANCAMENTO"  
    
🔵 Construir tabelas de dados auxiliares	  
✔			"with teste 
  as 
  (
  select 
  nome_artista_limpo,
  count(*) as nro_de_songs_do_artist

  from `proj002-lab-mariucha-ponte.Projeto02.spotify_limpo_sem_streams_com_obs_sobre_duplicados` 

  group by nome_artista_limpo
  )
#tabela temporaria

SELECT  
* except 
(
nome_alterado
)
FROM `proj002-lab-mariucha-ponte.Projeto02.spotify_limpo_sem_streams_com_obs_sobre_duplicados` as o
  

right join  teste
using (nome_artista_limpo)
#`USING` ao invés de `ON x=y`
"
🟣 Agrupar dados de acordo com variáveis ​​categóricas	  
✔			muito tranquilo, parece mesmo com tabelas dinamicas. é o icone de matriz, parece uma tabelinha, mas algumas celulas direitas e inferiores sao azuis
🟣 Visualizar variáveis ​​categóricas	  
✔			criei graficos de varios tipos para variaveis diferentes. achei bem tranquilo e quase intuitivo
🟣 Aplicar medidas de tendência central	  
✔			media e mediana, comparadas com desvio padrão. fiz com alguns dados e compreendi. o desvio padrão mais proxim oda media significa que há pouco desvio padrão, iu seja, que nao ha dados (muitos) dados distanmtes da média. se a diferenca entre desvi oapdroa e media for grande, significa que ha dados (muitos) longe da media. a mediana é o numero do meio (ou a media entre os dois numeros do meio, se a quantisdade de numeros for par. Se a media a e amédia foram muito diferentes isso significa que há uma diferenca grande entre os nuemros antes e depois da média, possivelemnte uma grande diferenca entre media e mediana ocorrerá tb an diferenca entre media e desvi opadrão?
🟣 Visualizar a distribuição dos dados	  
✔			"com dificuldades com meu computador pessoal, to usando um emprestado , entao nao quis inatalar o python e vou fazer no spreadsheets

usei indice corresp para juntar as planilhas da lab que tem as playlists dos cocnorrentes e do spotify, pelo trackID

fiz um histograma sobre a quantidade vezes que um artista aparece no spotify playlist e vi que a maioria so tem 1"
🟣 Aplicar medidas de dispersão	  
✔			"media e mediana, comparadas com desvio padrão. fiz com alguns dados e compreendi. o desvio padrão mais proxim oda media significa que há pouco desvio padrão, iu seja, que nao ha dados (muitos) dados distanmtes da média. se a diferenca entre desvi oapdroa e media for grande, significa que ha dados (muitos) longe da media. a mediana é o numero do meio (ou a media entre os dois numeros do meio, se a quantisdade de numeros for par. Se a media a e amédia foram muito diferentes isso significa que há uma diferenca grande entre os nuemros antes e depois da média, possivelemnte uma grande diferenca entre media e mediana ocorrerá tb an diferenca entre media e desvi opadrão?

desvio padrao baixo significa q há pouca distancia dos dados gerais pra media e mediana. (histograma sino)
desvio padrao alto significa que os daods em geral estao masi distantes da media (hitograma vale)"
🟣 Visualizar o comportamento dos dados ao longo do tempo	  
✔			fiz grafico de linha ao longo do tempo, vi como inserir filtro de "segmentacao de dados" (um tipo de grafico) flutuante na tela no dashboard
🟣 Calcular quartis, decis ou percentis	  
✔			"with q
as
(select 
streams_limpo,
ntile(4) over (order by streams_limpo) as quartis_streams
from `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` 
)

SELECT 
a.*,
q.quartis_streams
 FROM `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` as a
 
 left join q

 on a.streams_limpo=q.streams_limpo

-----------------------------------------

#categorizado em alto e baixo
WITH
  q AS (
  SELECT
    streams_limpo,
    NTILE(4) OVER (ORDER BY streams_limpo) AS quartis_streams
  FROM
    `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` )


SELECT
  a.*,
  q.quartis_streams,
IF
  (q.quartis_streams = 4, ""alto"", ""baixo"") as quartis_streams


FROM
  `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` AS a
LEFT JOIN
  q
ON
  a.streams_limpo=q.streams_limpo

WITH
  q AS (
  SELECT
    streams_limpo,
    NTILE(4) OVER (ORDER BY streams_limpo) AS quartis_streams
  FROM
    `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` )


SELECT
  a.*,
  q.quartis_streams,
IF
  (q.quartis_streams = 4, ""alto"", ""baixo"") as quartis_streams


FROM
  `proj002-lab-mariucha-ponte.Projeto02.hipoteses_completona_limpa` AS a
LEFT JOIN
  q
ON
  a.streams_limpo=q.streams_limpo

-------------------------------

quartis da categoria BPM

WITH
  q AS (
  SELECT
    track_id,
    bpm,
    NTILE(4) OVER (ORDER BY bpm) AS quartis_bpm
  FROM
    `proj002-lab-mariucha-ponte.Projeto02.tabela_completa_com_quartis` 
    )


SELECT
  a.*,
  q.quartis_bpm,
  #abaixo, estou colocando as categorias (nomes) em cada quartil
case
when q.quartis_bpm = 4 then ""alto""
when q.quartis_bpm = 3 then ""medio""
when q.quartis_bpm = 2 then ""baixo""
when q.quartis_bpm = 1 then ""irrisorio""
end
as quartil_bpm_nome

FROM
  `proj002-lab-mariucha-ponte.Projeto02.tabela_completa_com_quartis` AS a
 JOIN
  q
using (track_id)
"
🟣 Calcular correlação entre variáveis ​​	  
✔			"correlacao positiva entre quantidade de streams e quantidade de playlists onde a musica está
correlacao quase negativa (muito proxima de 0, indiferente) em caracteristicas da musica x streams
-----------------------------------------

SELECT

 round( 
      corr (streams_limpo,   in_spotify_playlists) 
  ,2)
  as streamsXplaylists_spotify,

round(
      corr (streams_limpo,   in_apple_playlists) 
  ,2)
  as streamsXplaylists_apple,

round(
      corr (streams_limpo,   in_deezer_playlists) 
  ,2)
  as streamsXplaylists_deezer,


round( 
      corr (bpm,  streams_limpo) 
  ,4)
  as bpmXstreams,


round( 
      corr (valence,  streams_limpo) 
  ,2)
  as valenceXstreams,


  round( 
      corr (danceability,  streams_limpo) 
  ,2)
  as danceXstreams,

 
  round( 
      corr (liveness,  streams_limpo) 
  ,2)
  as liveXstreams,


  round( 
      corr (speechness,  streams_limpo) 
  ,2)
  as speechXstreams,

  round( 
      corr (acousticness,  streams_limpo) 
  ,2)
  as acusticoXstreams,

  round( 
      corr (in_spotify_charts, in_deezer_charts) 
  ,2)
  as charts_spotifyXdeezer,

  round( 
      corr (in_spotify_charts, in_shazam_charts) 
  ,2)
  as charts_spotifyXshazam,

  round( 
      corr (in_spotify_charts, in_apple_charts) 
  ,2)
  as charts_spotifyXapple,


FROM
  `proj002-lab-mariucha-ponte.Projeto02.tabela_completa_com_quartis`

--------------------------------------
correlacao hipotese numero de musicas x numero de streams:

with base
as
(
  SELECT

nome_artista_limpo,
artist_count,
count(nome_artista_limpo) 
  as qts_musicas,
sum(streams_limpo) 
  as qts_streams

  FROM `proj002-lab-mariucha-ponte.Projeto02.tabela_completa_com_quartis` as tabela

group by 
nome_artista_limpo, 
artist_count

having artist_count = 1
and qts_streams is not null

)


SELECT

 round( 
      corr (base.qts_musicas, base.qts_streams) 
       ,2)
  as correspondencia,

  FROM base"
🔴 Aplicar segmentação	  
✔			fiz a segmentacao por quartis dos dtrems e comparei nao apena spor numeros ma stb por graficos barra com os numeros de cada uma das technical informations, e nao identifiquei relcao. concluo que nao tem correlcao
🔴 Validar hipótese	  
✔			"eu ja vinha validando as hipoteses e fiz uma aba neste arquivo com a conclusao sobre cada uma. porem ainda nao entendi bem como usar o isrograma, pois os 7 que fiz me deixam concluir que , novamnte, nao tem relacao.. vou ver o video novamente.

nao consegui absorver as coisas extra de estatistica do oraculo

link interessante: https://voitto.com.br/blog/artigo/teste-de-hipotese

coemcei a perguntar aochatgpt para entender  abase de estatistica"
🔴 Regressão linear	  
✔			nao encontrei esse conteudo na plataforma
🟠 Representar os dados por meio de tabela resumo ou scorecards	  
✔			muito teste e repetição e gráficos dieferentes. vi videos n youtube om outras solucoes também
🟠 Representar os dados através de gráficos simples	  
✔			esse foi mais fácil
🟠 Representar os dados por meio de gráficos ou recursos visuais avançados  	
✔			não foi mais dificil, como o nome sugeria
🟠 Representar os dados por meio de gráficos ou recursos visuais avançados		  
✔		marco adicional, nao cheguei nele
🟠 Aplicar opções de filtros para gerenciamento e interação	  
✔			"foi faciil tabém,. os videos ajudaram. 
E o chat gpt tambem me ajudou (com um filtro com o qual eu estava tendo dificuldade) a criar uma selecao por streams, com uma DAX e uma criacao de tabela adiciona:

Passo a passo (usando DAX)

Criar uma tabela auxiliar de plataformas (para o slicer):

No menu superior → Modelagem → Nova tabela.

Digite:

Plataformas = 
DATATABLE (
    ""Plataforma"", STRING,
    {
        {""Spotify""},
        {""Deezer""},
        {""Apple""}
    }
)


Agora você tem uma pequena tabela só com os 3 nomes das plataformas.

Criar uma medida que muda conforme a seleção:

Vá em Modelagem → Nova medida.

Digite algo assim (ajuste os nomes das colunas conforme estão na sua base):

Qtd_Playlists_Selecionada = 
SWITCH(
    SELECTEDVALUE(Plataformas[Plataforma]),
    ""Spotify"", SUM(Tabela[in_spotify_playlists]),
    ""Deezer"", SUM(Tabela[in_deezer_playlists]),
    ""Apple"", SUM(Tabela[in_apple_playlists]),
    BLANK()
)


O que acontece aqui:

SELECTEDVALUE pega a escolha do usuário no slicer (Spotify, Deezer ou Apple).

O SWITCH escolhe qual coluna somar com base nisso.

Montar o relatório:

Insira um slicer na tela e coloque nele Plataformas[Plataforma].

No gráfico de barras:

Eixo X → Track_id (ou nome da música).

Valores → a medida Qtd_Playlists_Selecionada."
🟢 Selecionar gráficos e informações relevantes	✔			
"me euni com inha dupla e discutimos quais graficos serviriam para cada caso e como fariamos funcionar. fiz u mrascunho no papel tambe, separando por cada hipotese,sendo que a hipotese 1 e a 5 foram juntas

contei com ajuda do coach vitor para inseriri um filtro visual que nao estava conseguindo. o principal problema é que eu estava puxando dados de tabelas diferenes e para o que eu queria, pra hipotese x, nao estava funcionado fáci lomo foi pra hipotese y.  os codigos usados para solucionar  foram:"
🟢 Selecionar gráficos e informações relevantes		
✔		marco2. voltarei a ele ao fim do periodo de entrega do proj 3
🟢 Criar uma apresentação   
✔			foi tranquilo. com a dupla definimos que a apresentaao seria simples, hipotese a hipotese, que eu faria o visual, ate pq meus graficos estavam organizados em dashboards ja, e por conta d eminha formacao em desing gráfico. peguei um modelo do canva apresentacoes para otimizar o tempo , fiz download como pptx e importei pro google apesentacoes. nele ajustei e inseri prints dos graficos do owerBI. Nao dediqui tempo a puxar os gráficos vinculados, então trabalhei ocm print mesmo, ate para dar tempo de fazer o projeto3 a tempo antes do fim do bootcamp.
🟢 Criar uma apresentação		
✔		marco2. voltarei a ele ao fim do periodo de entrega do proj 3
🟢 Apresentar resultados com conclusões e recomendações	
✔			onversei com a dupla, confirmamos que vamos usar a apsentacao com oficou depois de ajutarmos algumas coisas que discutimos. proximo passo: gravar o video e entegar as documentacoes.
🟢 Apresentar resultados com conclusões e recomendações		✔		marco2. voltarei a ele ao fim do periodo de entrega do proj 3

