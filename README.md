# Projeto Python: Ultimas avaliações de filmes do IMDB usando tecnicas de raspagem de dados

Para este projetos vamos retornar as ultimas 10 avaliações atualizadas de filmes em lançamento no IMDB

#### O foco deste projeto é avaliar o tempo de resposta usando Threads e Multiprocessos!

## Passo 1: Importar as bibliotecas 

    import requests #IMPORTA BLIBLIOTECA request PARA ACESSAR CONTEÚDO NA INTERNET
    import time #IMPORTA BIBLIOTECA time RESPONSAVEL POR GERIR O TEMPO DE EXECUÇÃO DE CADA CÓDIGO
    import csv #IMPORTA BIBLIOTECA csv RESPONSAVEL POR CONVERTER ARQUIVOS EM .CSV
    import random #IMPORTA BIBLIOTECA ramdom REPONSAVEL POR CRIAR ALEATÓRIEDADE ENTRE VALORES
    import concurrent.futures #IMPORTA BIBLIOTECA concurrent.futures RESPONSAVEL PELO GERENCIAMENTO DE THREADS
    from bs4 import BeautifulSoup #IMPORTA DA BIBLIOTECA bs4 O PACOTE BeatifulSoup RESPONSAVEL POR EXECUTAR UMA INTERFACE DE NAVEGAÇÃO NA INTERNET E MANIPULAR ARQUIVOS

## Passo 2: Definir o cabeçalho de navegação

    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Safari/537.36 Edge/12.246'} #ATRIBUI A VARIAVEL headers O CABEÇALHO DE NAVEGADDOR PARA QUE O GOOGLE ENTENDA QUE NÃO É O PYTHON QUE ESTÁ FAZENDO A REQUISIÇÃO E SIM UM NAVEGADOR

## Passo 3: Definir o maximo de Threads usadas para busca

    MAX_THREADS = 1 #ATRIBUI O VALOR 1 A VARIAVEL MAX_THREADS, OU SEJA O VALOR MAXIMO INICIAL DOS THREADS É 1

## Passo 4: Definir as funções para extração do filme e seus detalhes 

    def extract_movie_details(movie_link):#INICIO DE UMA FUNÇÃO RESPONSAVEL POR EXTRAIR DETALHES DOS FILMES DO imdb. USA COMO PARAMETRO O LINK DO FILME EXTRAIDO

        time.sleep(random.uniform(0, 0.2)) #DEFINE UM PEQUENO ESPAÇO DE TEMPO ANTES DO INICIO DA EXECUÇÃO DA FUNÇÃO COM BASE NO SISTEMA DE ALEATÓRIEDADE ENTRE 0 E 0.02

        response = requests.get(movie_link, headers=headers)#ATRIBUI A VARIAVEL RESPONSE INFORMAÇÕES DO SITE DO imdb DADO O LINK DO FILME E INFORMAÇÕES DO CABEÇALHO DE NAVEGADORES PARA QUE O GOOGLE ENTENDA SER UMA REQUISIÇÃO DE UM NAVEGADOR
        movie_soup = BeautifulSoup(response.content, 'html.parser') #ATRIBUI A VARIAVEL movie_soup AS INFORMAÇÕES OBTIDAS DO LINK EM HTML E RODA O LEITOR DE HTML

        if movie_soup is not None: #SE A VARIAVEL movie_soup NÃO POSSUI VALOR NONE FAÇA:
            title = None #ATRIBUA A VARIAVEL tittle O VALOR NONE
            date = None #ATRIBUA A VARIAVEL date O VALOR NONE
            
            # Encontrando a seção específica
            page_section = movie_soup.find('section', attrs={'class': 'ipc-page-section'}) #ATRIBUI A VARIAVEL page_section DADOS ENCONTRADOS NO HTML DO LINK DE INTERESSE
            
            if page_section is not None: #SE A VARIAVEL page_section NÃO POSSUI VALOR NONE FAÇA:
                # Encontrando todas as divs dentro da seção
                divs = page_section.find_all('div', recursive=False) #ATRIBUI A VARIAVEL divs TODAS AS INFORMAÇÕES EM HTML RELACIONADAS AS DIVS COM PARAMETRO RECURSIVE=FALSE
                
                if len(divs) > 1: #SE ENCONTRAR MAIS DE UMA DIV
                    target_div = divs[1] #ATRIBUA A VARIAVEL target_div APENAS A SEGUNDA DIV
                    
                    # Encontrando o título do filme
                    title_tag = target_div.find('h1') #ATRIBUA A VARIAVEL tittle_tag O VALOR h1 ENCONTRADO DENTRO DA target_div
                    
                    if title_tag: #SE title_tag FOR VERDADEIRO FAÇA:
                        title = title_tag.find('span').get_text() #ATRIBUA A VARIAVEL tittle O TEXTO ENCONTRADO DENTRO DO span                
                    # Encontrando a data de lançamento
                    date_tag = target_div.find('a', href=lambda href: href and 'releaseinfo' in href) #ATRIBUA A VARIAVEL date_tag OS VALORES ENCONTRADOS DENTRO DA target_div COM BASE NOS PARAMETROS DEFINIDOS DE BUSCA
                    
                    if date_tag: #SE date_tag FOR VERDADEIRO FAÇA:
                        date = date_tag.get_text().strip()#ATRIBUA A VARIAVEL date VALORES DE TEXTO ENCONTRADOS NA date_tag E REMOVA ESPAÇOS E TABULAÇÕES
                    
                    # Encontrando a classificação do filme
                    rating_tag = movie_soup.find('div', attrs={'data-testid': 'hero-rating-bar__aggregate-rating__score'})#ATRIBUA A VARIAVEL rating_tag INFORMAÇÕES ENCONTRADAS NA div DADO O PARAMETRO DE CLASSIFICAÇÃO DO FILME
                    rating = rating_tag.get_text() if rating_tag else None #ATRIBUA A VARIAVEL rating O TEXTO ENCONTRADO EM rating_tag SE rating_tag FOR VERDADEIRO, SENÃO ATRIBUA O VALOR NONE
                    
                    # Encontrando a sinopse do filme
                    plot_tag = movie_soup.find('span', attrs={'data-testid': 'plot-xs_to_m'})#ATRIBUA A VARIAVEL plot_tag INFORMAÇÕES ENCONTRADAS EM movie_soup SOBRE O span DADO O PARAMETRO DE BUSCA
                    plot_text = plot_tag.get_text().strip() if plot_tag else None #ATRIBUA A VARIAVEL plot_text O TEXTO ENCONTRADO SE A VARIAVEL plot_tag FOR VERDADEIRA, SENÃO  ATRIBUA O VALOR NONE
                    
                    with open('moviessingletherads.csv', mode='a', newline='', encoding='utf-8') as file: #ABRA O ARQUIVO movies.csv NO MODO DE ADICIONAR INFORMAÇÕES COM AS QUEBRAS DE LINHAS PRADONIZADAS NO FORMATO UTF-8 QUANDO SE TRATANDO DE CARACTERS ESPECIAIS
                        movie_writer = csv.writer(file, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)#ATRICBUA A VARIAVEL movie_writer UM ESCRITOR DE ARQUIVOS .CSV USANDO COMO PARAMETRO O ARQUIVO, VIRGULAS COMO DELIMITADOR, ASPAS PARA VALORES ESPECIAIS APENAS QUANDO NECESSARIO
                        if all([title, date, rating, plot_text]): #SE AS VARAVEIS tittle, date, rating, plot_text ACIMA FOREM VERDADEIRAS FAÇA:
                            print(title, date, rating, plot_text)#MOSTRE O QUE ESTA CONTIDO NAS VARIAVEIS title, date, rating e plot_text
                            movie_writer.writerow([title, date, rating, plot_text])#ESCREVA LINHA A LINHA O QUE ESTIVER CONTIDO NAS VARIAVEIS tittle, date, rating e plot_text NO FORMATO .CSV, NA VARIAVEL movie_writer

    def extract_movies(soup): #CRIE UMA FUNÇÃO extract_movies PARA A EXTRAÇÃO DOS FILMES USANDO soup COMO PARAMETRO 
        movies_table = soup.find('div', attrs={'data-testid': 'chart-layout-main-column'}).find('ul') #ATRIBUA A VARIAVEL movies_table VALORES ENCONTRADOS DENTRO DO soup COM BASE NOS PARAMETROS DEFINIDOS
        movies_table_rows = movies_table.find_all('li') #ATRIBUA A VARIAVEL movie_table_rows TODOS OS VALORES ENCONTRADOS DENTRO DA movie_table QUE CONTEM O PARAMETRO 'li'
        movie_links = ['https://imdb.com' + movie.find('a')['href'] for movie in movies_table_rows] #ATRIBUA A VARIAVEL movie_links UMA LISTA QUE CONTEM O LINK INICIAL DO IMDB MAIS TUDO ENCONTRADO NA VARIAVEL movie APÓS LOOP DA VARIAVEL movies_table_rows

        threads = min(MAX_THREADS, len(movie_links)) #ATRIBUA A VARIAVEL threads O VALOR MINIMO DE THREADS USADAS DADO A MAX_THREADS = 10 E A CONTAGEM DE LINKS RETORNADOS, COM O OBJETIVO DE EVITAR DESPERDICIO DE PODER DE PROCESSAMENTO E MAXIMIZAR A EFIENCIA DOS PROCESSOS
        with concurrent.futures.ThreadPoolExecutor(max_workers=threads) as executor:#CRIA UMA GERENCIADOR DE THREADS USANDO OS LIMITES DE USO DE THREADS COMO PARAMETRO E NOMEA-O COMO executor
            executor.map(extract_movie_details, movie_links)#MAPEA DENTRO DA FUNÇÃO OS VALORES CONTIDOS EM movie_links E ATRIBUI OS LINKS AO GERENCIADOR executor

## Passo 5: Definir a função principal para extração e alocação das informações obtidas em um arquivo .csv e retornar o tempo de resposta 

    def main():#CRIE A FUNÇÃO PRINCIPAL main()
        start_time = time.time() #ATRIBUA A VARIAVEL start_time O TEMPO ATUAL

        # IMDB Most Popular Movies - 100 movies
        popular_movies_url = 'https://www.imdb.com/chart/moviemeter/?ref_=nv_mv_mpm'#ATRIBUA A VARIAVEL popular_movies_url O LINK DEFINIDO
        response = requests.get(popular_movies_url, headers=headers) #ATRIBUA A VARIAVEL response AS INFORMAÇÕES DO LINK USANDO O PARAMETRO headers PARA QUE O GOOGLE ENTENDA QUE A PESQUISA É FEITA POR UM NAVEGADOR E NÃO PELO PYTHON
        soup = BeautifulSoup(response.content, 'html.parser') #ATRIBUA A VARIAVEL soup AS INFORMAÇÕES DO LINK EM HTML E RODA UM LEITOR HTML

        # Main function to extract the 100 movies from IMDB Most Popular Movies
        extract_movies(soup)#RODA A FUNÇÃO extract_movies 

        end_time = time.time()#ATRIBUA A VARIAVEL end_time O TEMPO ATUAL
        print('Total time taken: ', end_time - start_time) #MOSTRE A MENSAGEM DE TEMPO DE EXECUÇÃO

    if __name__ == '__main__': #SE O ARQUIVO ESTIVER SENDO EXECUTADO DIRETAMENTE NO PYTHON FAÇA:
        main()#RODE A FUNÇÃO


## Passo 6: Rode o código e avalie o tempo de resposta

Usando a IDE para escrita do código, rode-o e avalie o tempo de resposta. Fique a vontade em mudar os valores do MAX_THREADS e comparar os tempos de resposta obtidos.