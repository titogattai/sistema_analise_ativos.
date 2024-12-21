# ============================
# Importações Padrão
# ============================
import datetime
import math
import random
import csv
import os
import logging

# ============================
# Configuração de Log
# ============================
logging.basicConfig(filename="analise_ativos.log", level=logging.INFO,
                    format="%(asctime)s - %(levelname)s - %(message)s")

# ============================
# Funções Utilitárias
# ============================
def verificar_diretorio(diretorio):
    """Verifica e cria diretório se necessário."""
    if not os.path.exists(diretorio):
        os.makedirs(diretorio)
        logging.info(f"Diretório {diretorio} criado.")

def carregar_dados_arquivo(arquivo):
    """Carrega os dados a partir de um arquivo CSV."""
    dados = []
    with open(arquivo, 'r') as f:
        leitor = csv.reader(f)
        next(leitor)  # Pular cabeçalho
        for linha in leitor:
            data = linha[0]
            preco = float(linha[1])
            dados.append((data, preco))
    return dados

# ============================
# Análises Técnicas
# ============================
def calcular_rsi(dados, periodo=14):
    """Calcula RSI manualmente."""
    ganhos, perdas = 0, 0
    for i in range(1, periodo + 1):
        diferenca = dados[i][1] - dados[i - 1][1]
        if diferenca > 0:
            ganhos += diferenca
        else:
            perdas -= diferenca
    
    media_ganhos = ganhos / periodo
    media_perdas = perdas / periodo
    rs = media_ganhos / media_perdas if media_perdas != 0 else 0
    rsi = 100 - (100 / (1 + rs))
    return rsi

def calcular_media_movel(dados, periodo):
    """Calcula Média Móvel Simples."""
    soma = sum([dados[i][1] for i in range(-periodo, 0)])
    return soma / periodo

def calcular_macd(dados):
    """Calcula MACD manualmente."""
    ema_12 = calcular_media_movel(dados, 12)
    ema_26 = calcular_media_movel(dados, 26)
    macd = ema_12 - ema_26
    signal = calcular_media_movel(dados, 9)
    return macd, signal

def calcular_bollinger_bands(dados, periodo=20):
    """Calcula Bandas de Bollinger manualmente."""
    media = calcular_media_movel(dados, periodo)
    variancia = sum([(dados[i][1] - media) ** 2 for i in range(-periodo, 0)]) / periodo
    desvio_padrao = math.sqrt(variancia)
    upper_band = media + (2 * desvio_padrao)
    lower_band = media - (2 * desvio_padrao)
    return upper_band, lower_band

# ============================
# Funções Estatísticas
# ============================
def calcular_sharpe(dados, taxa_risco=0.02):
    """Calcula índice de Sharpe manualmente."""
    retornos = [(dados[i][1] / dados[i - 1][1]) - 1 for i in range(1, len(dados))]
    retorno_medio = sum(retornos) / len(retornos)
    volatilidade = math.sqrt(sum([(r - retorno_medio) ** 2 for r in retornos]) / len(retornos))
    sharpe = (retorno_medio - taxa_risco) / volatilidade if volatilidade != 0 else 0
    return sharpe

def calcular_drawdown(dados):
    """Calcula o drawdown máximo."""
    maximo = 0
    drawdown = 0
    for i in range(len(dados)):
        if dados[i][1] > maximo:
            maximo = dados[i][1]
        perda = (maximo - dados[i][1]) / maximo
        drawdown = max(drawdown, perda)
    return drawdown

# ============================
# Previsões Simples
# ============================
def prever_proximo_valor(dados):
    """Modelo básico para prever o próximo valor."""
    if len(dados) < 2:
        return dados[-1][1]
    tendencia = dados[-1][1] - dados[-2][1]
    return dados[-1][1] + tendencia

# ============================
# Visualizações Simples em Texto
# ============================
def exibir_graficos_texto(dados):
    """Exibe gráfico simples no terminal usando caracteres."""
    max_valor = max([d[1] for d in dados])
    min_valor = min([d[1] for d in dados])
    intervalo = max_valor - min_valor
    
    for data, preco in dados:
        barras = int(((preco - min_valor) / intervalo) * 50)
        print(f"{data}: {'#' * barras}")

# ============================
# Função Principal
# ============================
def sistema_analise(ticker, arquivo):
    """Executa análise completa para o ativo fornecido."""
    verificar_diretorio('resultados')
    dados = carregar_dados_arquivo(arquivo)
    if not dados:
        logging.error("Nenhum dado disponível.")
        return

    # Análises
    rsi = calcular_rsi(dados)
    macd, signal = calcular_macd(dados)
    upper_band, lower_band = calcular_bollinger_bands(dados)
    sharpe = calcular_sharpe(dados)
    drawdown = calcular_drawdown(dados)
    previsao = prever_proximo_valor(dados)

    # Resultados
    logging.info(f"{ticker} - RSI: {rsi}")
    logging.info(f"{ticker} - MACD: {macd}, Sinal: {signal}")
    logging.info(f"{ticker} - Bandas de Bollinger: ({upper_band}, {lower_band})")
    logging.info(f"{ticker} - Sharpe: {sharpe}, Drawdown: {drawdown}")
    logging.info(f"{ticker} - Previsão: {previsao}")

    print(f"Resultados para {ticker}:")
    print(f"RSI: {rsi}")
    print(f"MACD: {macd}, Signal: {signal}")
    print(f"Bandas de Bollinger: ({upper_band}, {lower_band})")
    print(f"Sharpe: {sharpe}")
    print(f"Drawdown: {drawdown}")
    print(f"Previsão: {previsao}")

    # Exibir gráficos
    exibir_graficos_texto(dados)

# ============================
# Execução
# ============================
if __name__ == "__main__":
    sistema_analise("AAPL", "dados_aapl.csv")
