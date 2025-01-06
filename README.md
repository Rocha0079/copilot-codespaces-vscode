from flask import Flask, request
from twilio.twiml.messaging_response import MessagingResponse

app = Flask(__name__)

# Dicionário para armazenar as informações
dados_diarios = {
    'cronograma': {},
    'financas': {
        'bruto': None,
        'despesas': None,
        'pessoais': None,
        'planos': None,
        'contas': {}
    }
}

@app.route('/sms', methods=['POST'])
def sms_reply():
    msg = request.form.get('Body').lower()
    resp = MessagingResponse()
    
    if 'olá' in msg:
        resposta = "Olá! Como posso ajudar você hoje? Use comandos como 'adicionar cronograma', 'adicionar finanças' ou 'ver cronograma'."
    elif 'adicionar cronograma' in msg:
        partes = msg.split()
        dia = partes[-1]  # Última palavra deve ser o dia
        atividade = ' '.join(partes[2:-1])  # Atividade está entre 'adicionar cronograma' e o dia
        dados_diarios['cronograma'][dia] = atividade
        resposta = f"Atividade para o dia {dia} adicionada: {atividade}"
    elif 'adicionar finanças' in msg:
        partes = msg.split()
        tipo = partes[2]
        valor = ' '.join(partes[3:])
        if tipo in ['bruto', 'despesas', 'pessoais', 'planos']:
            dados_diarios['financas'][tipo] = valor
            resposta = f"Finança '{tipo}' adicionada: {valor}"
        elif tipo in ['aluguel', 'agua', 'luz', 'internet']:
            dados_diarios['financas']['contas'][tipo] = valor
            resposta = f"Conta '{tipo}' adicionada: {valor}"
        else:
            resposta = "Tipo de finança desconhecido. Use 'bruto', 'despesas', 'pessoais', 'planos', 'aluguel', 'agua', 'luz' ou 'internet'."
    elif 'ver cronograma' in msg:
        cronograma = "\n".join([f"{dia}: {atividade}" for dia, atividade in dados_diarios['cronograma'].items()])
        
