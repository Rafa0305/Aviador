# Aviador

from selenium.webdriver.edge.options import Options as EdgeOptions
from selenium.webdriver.common.by import By 
from selenium import webdriver
from datetime import datetime, timedelta
from selenium.webdriver import ChromeOptions 
from time import sleep
import csv
import os

usuario = ''
senha = ''

# Duração da coleta (em minutos)
tempo_minutos = 60 #60  # Altere aqui conforme necessário
fim_coleta = datetime.now() + timedelta(minutes=tempo_minutos)
inicio_coleta = datetime.now()

# Nome do arquivo com timestamp
agora = inicio_coleta.strftime("%Y-%m-%d_%H-%M-%S")
nome_arquivo = f"resultados_{agora}.csv"
caminho_arquivo = os.path.join("C:\\coleta_aviador", nome_arquivo)


# Configurar o WebDriver
edge_options = EdgeOptions()
opts = ChromeOptions()
opts. add_experimental_option("detach", False)
driver = webdriver.Edge(options=edge_options)
driver = webdriver.Edge()

# Acessar o WhatsApp Web
url = 'https://sortenabet.bet.br/games/spribe/aviator'
driver.get(url)
driver.maximize_window()
sleep(5)

# clicar que é maior de 18 anos
try:
    driver.find_element(By.XPATH, '/html/body/div[5]/div[1]/div[2]/div[2]/button[2]').click()
    sleep(2)
except:
    print("Continuando... (nenhum botão de maior encontrado)")

# clcicar em entrar
driver.find_element(By.XPATH, '/html/body/div[1]/div[1]/div[1]/header/div[1]/div/div[2]/div/button[2]').click()
sleep(3)

# digitar usuario e senha
driver.find_element(By.XPATH, '/html/body/div[5]/div/div/div/div/div[1]/div/div[2]/div[1]/div[2]/form/div[1]/label/input').send_keys(usuario)
sleep(1)
driver.find_element(By.XPATH, '/html/body/div[5]/div/div/div/div/div[1]/div/div[2]/div[1]/div[2]/form/div[2]/div/label/input').send_keys(senha)
sleep(1)
driver.find_element(By.XPATH, '/html/body/div[5]/div/div/div/div/div[1]/div/div[2]/div[1]/div[2]/form/div[4]/button').click()
sleep(5)

# Acessa o iframe
iframe = driver.find_element(By.XPATH, '/html/body/div[1]/div[1]/div[3]/div[1]/section/section/iframe')
driver.switch_to.frame(iframe)

# Lista de resultados
valores_registrados = []

# Loop de coleta
while datetime.now() < fim_coleta:
    try:
        driver.find_element(By.XPATH, '/html/body/app-root/app-game/div/div[1]/div[2]/div/div[2]/div[1]/app-stats-widget/div/div[2]/div').click()
        sleep(1)
        resultado = driver.find_element(By.XPATH, '/html/body/app-root/app-game/div/div[1]/div[2]/div/div[2]/div[1]/app-stats-widget/div/app-stats-dropdown/div/div[2]').text
        resultados = resultado.split('\n')

        # Adiciona valores (permite repetir, mas nãSo em sequência)
        for val in resultados:
            if not (valores_registrados and valores_registrados[-1] == val):
                valores_registrados.append(val)

        print(f"[{datetime.now().strftime('%H:%M:%S')}] Capturados: {resultados}")
    except Exception as e:
        print(f"Erro ao capturar valores: {e}")

    sleep(15)

# Salva os dados no CSV
with open(caminho_arquivo, mode='w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerow(['inicio_execucao', inicio_coleta.strftime('%Y-%m-%d %H:%M:%S')])
    writer.writerow(['fim_execucao', datetime.now().strftime('%Y-%m-%d %H:%M:%S')])
    writer.writerow(['valores'])
    for val in valores_registrados:
        writer.writerow([val])

driver.quit()
print(f"Coleta finalizada. Arquivo salvo em: {caminho_arquivo}")

